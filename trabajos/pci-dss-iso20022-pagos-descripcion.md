# Estándares de Pagos: PCI DSS e ISO 20022

## Parte 1: PCI DSS (Payment Card Industry Data Security Standard)

### 1.1 ¿Qué es PCI DSS?

**PCI DSS** es un estándar de seguridad de la información establecido por el **PCI Security Standards Council** (fundado en 2006 por Visa, MasterCard, American Express, Discover y JCB) para proteger los datos de tarjetas de pago contra fraude y brechas de seguridad.

**Versión actual**: PCI DSS 4.0.1 (lanzada en marzo 2022, obligatoria desde marzo 2024)

**Objetivo**: Establecer requisitos mínimos técnicos y operacionales para proteger los datos de cuentas de pago y prevenir brechas de seguridad.

### 1.2 ¿Quién Debe Cumplir con PCI DSS?

**Cualquier organización que**:
- Procesa transacciones con tarjetas de pago
- Almacena datos de tarjetas
- Transmite información de tarjetas

**Aplica a TravelHub porque**:
- Procesa pagos de reservas hoteleras
- Transmite datos de tarjetas a procesadores de pago
- Almacena (temporalmente) información de transacciones

### 1.3 Niveles de Cumplimiento

Los comerciantes se clasifican según el volumen anual de transacciones:

| Nivel | Volumen Anual de Transacciones | Requisitos de Validación |
|-------|--------------------------------|--------------------------|
| **Nivel 1** | > 6 millones | Auditoría anual por QSA + escaneo trimestral |
| **Nivel 2** | 1-6 millones | Cuestionario anual (SAQ) + escaneo trimestral |
| **Nivel 3** | 20,000 - 1 millón (e-commerce) | SAQ anual + escaneo trimestral |
| **Nivel 4** | < 20,000 (e-commerce) o < 1M (otros) | SAQ anual + escaneo trimestral |

**TravelHub**: Con ~450,000 viajeros activos y múltiples transacciones por usuario, probablemente está en **Nivel 2 o 3**.

---

## 1.4 Los 12 Requisitos de PCI DSS 4.0

PCI DSS se organiza en **6 objetivos** y **12 requisitos**:

### Objetivo 1: Construir y Mantener una Red Segura

#### Requisito 1: Instalar y Mantener Controles de Seguridad de Red
**Qué significa**: Implementar firewalls, segmentación de red, y controles de acceso.

**Cambios en 4.0**:
- Término ampliado de "firewall" a "controles de seguridad de red" (incluye WAF, IPS, segmentación)
- Segmentación de red obligatoria para aislar el **Cardholder Data Environment (CDE)**

**Para TravelHub**:
- Segmentar servicios de pago del resto de la arquitectura
- Implementar AWS Security Groups restrictivos
- Usar AWS WAF para proteger APIs de pago
- Implementar Network ACLs para controlar tráfico entre subnets

**Ejemplo de arquitectura**:
```
┌─────────────────────────────────────────────────────┐
│  Public Subnet (Internet-facing)                    │
│  - API Gateway (solo endpoints públicos)            │
│  - WAF (filtrado de tráfico malicioso)              │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│  Application Subnet (Private)                       │
│  - Booking Service                                   │
│  - Inventory Service                                 │
│  - User Service                                      │
└──────────────────┬──────────────────────────────────┘
                   │
┌──────────────────▼──────────────────────────────────┐
│  Payment Subnet (Isolated CDE)                      │
│  - Payment Service (único con acceso a datos de     │
│    tarjetas)                                         │
│  - Tokenization Service                              │
│  - Security Groups ultra-restrictivos                │
│  - Logging exhaustivo                                │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
            Payment Gateway
         (Stripe, Adyen, etc.)
```

#### Requisito 2: Aplicar Configuraciones Seguras
**Qué significa**: Eliminar configuraciones por defecto, deshabilitar servicios innecesarios, aplicar hardening.

**Para TravelHub**:
- Cambiar contraseñas por defecto en bases de datos, servidores
- Deshabilitar puertos y servicios no utilizados
- Usar imágenes de contenedor endurecidas (hardened)
- Implementar Infrastructure as Code (Terraform) con configuraciones seguras

**Checklist**:
- [ ] Cambiar credenciales por defecto de RDS, ElastiCache, etc.
- [ ] Deshabilitar SSH directo (usar AWS Systems Manager Session Manager)
- [ ] Aplicar CIS Benchmarks para AMIs y contenedores
- [ ] Documentar todas las configuraciones de seguridad

---

### Objetivo 2: Proteger los Datos de Tarjetas

#### Requisito 3: Proteger los Datos de Cuentas Almacenados
**Qué significa**: Minimizar almacenamiento de datos de tarjetas, cifrar datos en reposo.

**Datos Prohibidos de Almacenar** (incluso cifrados):
- ❌ **CVV/CVC** (código de seguridad de 3-4 dígitos)
- ❌ **Datos de banda magnética completos**
- ❌ **PIN o PIN block**

**Datos Permitidos con Restricciones**:
- ✅ **PAN (Primary Account Number)**: Solo si es absolutamente necesario, cifrado con AES-256
- ✅ **Fecha de expiración**: Cifrada o tokenizada
- ✅ **Nombre del tarjetahabiente**: Cifrado

**Mejor Práctica: Tokenización**

En lugar de almacenar datos reales de tarjetas, usar **tokens**:

```
Flujo sin tokenización (NO RECOMENDADO):
Usuario → TravelHub → Almacena PAN cifrado → Procesa pago

Flujo con tokenización (RECOMENDADO):
Usuario → TravelHub → Payment Gateway → Devuelve token
TravelHub almacena token (no PAN real)
Para cobros futuros: TravelHub → Envía token → Gateway procesa
```

**Para TravelHub**:
- **NO almacenar datos de tarjetas** directamente
- Usar tokenización de Stripe, Adyen, o procesador de pago
- Almacenar solo tokens de pago
- Si se requiere almacenar PAN (ej: suscripciones), usar **AWS KMS** con envelope encryption

**Ejemplo de datos almacenados**:
```json
{
  "bookingId": "bk_789xyz",
  "userId": "user_456",
  "paymentMethod": {
    "type": "credit_card",
    "token": "tok_1234567890abcdef",  // Token, NO PAN real
    "last4": "4242",                   // Últimos 4 dígitos (permitido)
    "brand": "visa",
    "expiryMonth": 12,
    "expiryYear": 2028
  },
  "amount": 450.00,
  "currency": "USD"
}
```

#### Requisito 4: Proteger los Datos de Tarjetas en Tránsito
**Qué significa**: Cifrar datos de tarjetas cuando se transmiten por redes públicas.

**Para TravelHub**:
- Usar **TLS 1.3** (mínimo TLS 1.2) para todas las comunicaciones
- Implementar **HSTS** (HTTP Strict Transport Security)
- Validar certificados SSL/TLS
- Usar **mTLS** (mutual TLS) para comunicación con payment gateways

**Configuración de TLS**:
```yaml
# API Gateway / Load Balancer
ssl_protocols: TLSv1.3 TLSv1.2
ssl_ciphers: 
  - TLS_AES_256_GCM_SHA384
  - TLS_AES_128_GCM_SHA256
  - TLS_CHACHA20_POLY1305_SHA256
ssl_prefer_server_ciphers: on
hsts_max_age: 31536000  # 1 año
hsts_include_subdomains: true
```

---

### Objetivo 3: Mantener un Programa de Gestión de Vulnerabilidades

#### Requisito 5: Proteger Todos los Sistemas contra Malware
**Qué significa**: Implementar antivirus, anti-malware, y detección de amenazas.

**Para TravelHub**:
- AWS GuardDuty para detección de amenazas
- Amazon Inspector para escaneo de vulnerabilidades
- Antivirus en instancias EC2 (si aplica)
- Escaneo de imágenes de contenedor (AWS ECR scanning)

#### Requisito 6: Desarrollar y Mantener Sistemas y Software Seguros
**Qué significa**: Aplicar parches de seguridad, realizar pruebas de seguridad, seguir prácticas de desarrollo seguro.

**Cambios en 4.0**:
- **Requisito 6.4.3 (nuevo)**: Scripts de pago deben ser gestionados y autorizados
- **Requisito 6.5 (ampliado)**: Prevención de vulnerabilidades comunes (OWASP Top 10)

**Para TravelHub**:
- Implementar **SAST** (Static Application Security Testing) en CI/CD
- Implementar **DAST** (Dynamic Application Security Testing)
- Escaneo de dependencias (Snyk, Dependabot)
- Revisión de código con foco en seguridad
- Aplicar parches de seguridad en < 30 días (críticos < 7 días)

**Pipeline de CI/CD con seguridad**:
```yaml
# .github/workflows/security.yml
name: Security Checks
on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: SAST - SonarQube
        uses: sonarsource/sonarqube-scan-action@master
      
      - name: Dependency scanning
        uses: snyk/actions/node@master
        with:
          command: test
      
      - name: Container scanning
        run: |
          docker build -t payment-service .
          trivy image payment-service
      
      - name: OWASP ZAP scan
        uses: zaproxy/action-baseline@v0.7.0
        with:
          target: 'https://staging-api.travelhub.com'
```

---

### Objetivo 4: Implementar Medidas de Control de Acceso Fuertes

#### Requisito 7: Restringir el Acceso a Datos de Tarjetas
**Qué significa**: Principio de mínimo privilegio, acceso basado en roles.

**Para TravelHub**:
- Implementar **RBAC** (Role-Based Access Control)
- Solo el Payment Service tiene acceso a datos de tarjetas
- Logs de auditoría de todos los accesos

**Ejemplo de roles**:
```yaml
roles:
  - name: payment_processor
    permissions:
      - payment:process
      - payment:read
      - payment:tokenize
    services:
      - payment-service
  
  - name: booking_manager
    permissions:
      - booking:create
      - booking:read
      - booking:update
    services:
      - booking-service
    # NO tiene acceso a payment-service
  
  - name: support_agent
    permissions:
      - booking:read
      - payment:read_masked  # Solo últimos 4 dígitos
    services:
      - booking-service
```

#### Requisito 8: Identificar Usuarios y Autenticar el Acceso
**Qué significa**: Autenticación fuerte, MFA obligatorio, gestión de contraseñas.

**Cambios en 4.0**:
- **MFA obligatorio** para todo acceso al CDE
- Contraseñas de mínimo 12 caracteres (antes 8)
- Autenticación sin contraseña permitida (biometría, FIDO2)

**Para TravelHub**:
- Implementar **MFA** para todos los empleados (AWS IAM, Okta, Auth0)
- Usar **SSO** (Single Sign-On) con SAML/OIDC
- Contraseñas de mínimo 12 caracteres con complejidad
- Rotación de credenciales cada 90 días
- Bloqueo de cuenta después de 6 intentos fallidos

**Política de contraseñas**:
```json
{
  "passwordPolicy": {
    "minimumLength": 12,
    "requireUppercase": true,
    "requireLowercase": true,
    "requireNumbers": true,
    "requireSymbols": true,
    "maxAge": 90,
    "preventReuse": 12,
    "lockoutThreshold": 6,
    "lockoutDuration": 30
  }
}
```

#### Requisito 9: Restringir el Acceso Físico a Datos de Tarjetas
**Qué significa**: Controlar acceso físico a servidores, centros de datos.

**Para TravelHub** (infraestructura en AWS):
- AWS es responsable de seguridad física (modelo de responsabilidad compartida)
- TravelHub debe controlar acceso a oficinas donde se accede al CDE
- Implementar controles de acceso físico (badges, cámaras, logs)

---

### Objetivo 5: Monitorear y Probar Redes Regularmente

#### Requisito 10: Registrar y Monitorear Todo el Acceso a Recursos de Red y Datos de Tarjetas
**Qué significa**: Logging exhaustivo, retención de logs, monitoreo en tiempo real.

**Cambios en 4.0**:
- **Requisito 10.4.1.1 (nuevo)**: Detección automatizada de anomalías
- Logs deben incluir contexto adicional (user agent, geolocalización)

**Para TravelHub**:
- Centralizar logs en **CloudWatch Logs** o **ELK Stack**
- Retener logs por mínimo **12 meses** (3 meses online, 9 meses archivados)
- Implementar alertas en tiempo real para eventos sospechosos
- Logs inmutables (WORM - Write Once Read Many)

**Eventos que deben registrarse**:
- ✅ Todos los accesos a datos de tarjetas
- ✅ Acciones de usuarios con privilegios
- ✅ Acceso a logs de auditoría
- ✅ Intentos de autenticación (exitosos y fallidos)
- ✅ Creación/eliminación de cuentas
- ✅ Cambios en configuraciones de seguridad
- ✅ Inicialización/detención de logs

**Formato de log estructurado**:
```json
{
  "timestamp": "2026-02-25T22:00:00.123Z",
  "eventType": "payment.processed",
  "userId": "user_456",
  "ipAddress": "192.168.1.100",
  "userAgent": "Mozilla/5.0...",
  "geolocation": {"country": "CO", "city": "Bogotá"},
  "action": "process_payment",
  "resource": "payment-service",
  "result": "success",
  "paymentToken": "tok_1234567890abcdef",
  "amount": 450.00,
  "currency": "USD",
  "traceId": "trace_abc123"
}
```

#### Requisito 11: Probar la Seguridad de Sistemas y Redes Regularmente
**Qué significa**: Escaneos de vulnerabilidades, pruebas de penetración, monitoreo de integridad de archivos.

**Para TravelHub**:
- **Escaneos de vulnerabilidades trimestrales** por ASV (Approved Scanning Vendor)
- **Pruebas de penetración anuales** (internas y externas)
- **Monitoreo de integridad de archivos** (FIM) en sistemas críticos
- **Escaneos de seguridad de aplicaciones** (SAST/DAST) en cada release

**Herramientas**:
- AWS Inspector (escaneo de vulnerabilidades)
- Qualys, Tenable (ASV aprobados)
- Burp Suite, OWASP ZAP (pruebas de penetración)
- AIDE, Tripwire (FIM)

---

### Objetivo 6: Mantener una Política de Seguridad de la Información

#### Requisito 12: Soportar la Seguridad de la Información con Políticas y Programas Organizacionales
**Qué significa**: Documentar políticas de seguridad, capacitación de empleados, gestión de riesgos.

**Cambios en 4.0**:
- **Requisito 12.3.1 (nuevo)**: Análisis de riesgos dirigido (targeted risk analysis)
- **Requisito 12.8 (ampliado)**: Gestión de riesgo de proveedores terceros
- **Requisito 12.10 (nuevo)**: Plan de respuesta a incidentes documentado y probado

**Para TravelHub**:
- Documentar **Política de Seguridad de la Información**
- Capacitación anual de seguridad para todos los empleados
- Capacitación específica de PCI DSS para equipo de desarrollo y operaciones
- Análisis de riesgos anual
- Plan de respuesta a incidentes (probado semestralmente)
- Gestión de proveedores (validar cumplimiento PCI DSS de payment gateways)

**Documentos requeridos**:
- [ ] Política de Seguridad de la Información
- [ ] Política de Uso Aceptable
- [ ] Política de Control de Acceso
- [ ] Política de Gestión de Contraseñas
- [ ] Política de Retención de Datos
- [ ] Plan de Respuesta a Incidentes
- [ ] Plan de Continuidad de Negocio
- [ ] Registro de Análisis de Riesgos

---

## 1.5 Estrategias de Cumplimiento para TravelHub

### Estrategia 1: Reducir el Alcance del CDE (Recomendada)

**Objetivo**: Minimizar los sistemas que manejan datos de tarjetas.

**Implementación**:
1. **NO almacenar datos de tarjetas** en TravelHub
2. Usar **iframes de pago** del payment gateway (Stripe Elements, Adyen Drop-in)
3. Datos de tarjeta van directamente del navegador al gateway
4. TravelHub solo recibe tokens de pago

**Arquitectura**:
```
┌─────────────┐
│   Usuario   │
└──────┬──────┘
       │ (1) Ingresa datos de tarjeta en iframe
       ▼
┌─────────────────────┐
│  Payment Gateway    │
│  (Stripe/Adyen)     │◄──────────────┐
└──────┬──────────────┘               │
       │ (2) Devuelve token            │ (4) Procesa pago
       ▼                               │     con token
┌─────────────────────┐               │
│  TravelHub          │───────────────┘
│  (solo maneja       │ (3) Envía token
│   tokens)           │
└─────────────────────┘
```

**Ventajas**:
- ✅ TravelHub **NO está en alcance** de la mayoría de requisitos PCI DSS
- ✅ Solo necesita cumplir **SAQ A** (el más simple)
- ✅ Reducción del 90% en costo de cumplimiento
- ✅ Menor riesgo de brechas de seguridad

### Estrategia 2: Usar Payment Gateway con PCI DSS Nivel 1

**Proveedores recomendados**:
- **Stripe**: PCI DSS Level 1, tokenización incluida, soporte LATAM
- **Adyen**: PCI DSS Level 1, multi-moneda, soporte local
- **PayU**: Popular en LATAM, PCI DSS Level 1
- **Mercado Pago**: Amplia adopción en LATAM

**Responsabilidades**:
- **Payment Gateway**: Almacenamiento seguro, procesamiento, cumplimiento PCI DSS
- **TravelHub**: Transmisión segura (TLS), validación de tokens, logging

### Estrategia 3: Segmentación de Red Estricta

Si TravelHub debe manejar datos de tarjetas:

```
┌────────────────────────────────────────────────────┐
│  Fuera del CDE (No PCI DSS)                        │
│  - Frontend                                         │
│  - Booking Service                                  │
│  - Inventory Service                                │
│  - User Service                                     │
└────────────────────────────────────────────────────┘

┌────────────────────────────────────────────────────┐
│  Dentro del CDE (PCI DSS completo)                 │
│  - Payment Service (único con acceso a PAN)        │
│  - Tokenization Service                             │
│  - Payment Database (cifrada)                       │
│  - Logs de auditoría                                │
└────────────────────────────────────────────────────┘
```

---

## 1.6 Checklist de Cumplimiento PCI DSS para TravelHub

### Fase 1: Fundamentos (Meses 1-2)
- [ ] Definir alcance del CDE
- [ ] Implementar tokenización con payment gateway
- [ ] Configurar TLS 1.3 en todas las APIs
- [ ] Implementar segmentación de red básica
- [ ] Configurar logging centralizado

### Fase 2: Controles de Acceso (Meses 3-4)
- [ ] Implementar MFA para todos los empleados
- [ ] Configurar RBAC en servicios
- [ ] Aplicar política de contraseñas (12+ caracteres)
- [ ] Documentar matriz de acceso

### Fase 3: Monitoreo y Testing (Meses 5-6)
- [ ] Implementar alertas de seguridad en tiempo real
- [ ] Configurar FIM en sistemas críticos
- [ ] Realizar primer escaneo de vulnerabilidades (ASV)
- [ ] Implementar SAST/DAST en CI/CD

### Fase 4: Documentación y Políticas (Meses 7-8)
- [ ] Documentar todas las políticas de seguridad
- [ ] Crear plan de respuesta a incidentes
- [ ] Capacitar a empleados en seguridad
- [ ] Realizar análisis de riesgos

### Fase 5: Validación (Meses 9-10)
- [ ] Completar SAQ (Self-Assessment Questionnaire)
- [ ] Realizar prueba de penetración
- [ ] Corregir hallazgos
- [ ] Obtener Attestation of Compliance (AOC)

---


## Parte 2: ISO 20022 (Mensajería de Pagos Financieros)

### 2.1 ¿Qué es ISO 20022?

**ISO 20022** es un estándar global para mensajería financiera electrónica desarrollado por la **International Organization for Standardization (ISO)**. Proporciona un lenguaje común y una plataforma para el desarrollo de mensajes financieros en toda la industria de servicios financieros.

**Lanzamiento**: 2004 (adopción masiva desde 2020+)  
**Adopción esperada**: Para 2025, se espera que soporte más del 80% de transacciones globales de alto valor

**Diferencia clave con formatos legacy**:
- **Formatos legacy** (MT, SWIFT FIN): Mensajes de texto plano con campos de longitud fija
- **ISO 20022**: Mensajes XML/JSON estructurados con datos ricos y descriptivos

### 2.2 ¿Por Qué ISO 20022?

#### Problemas con Formatos Legacy

**Ejemplo de mensaje SWIFT MT103 (legacy)**:
```
:20:REFERENCE123
:32A:260225USD450,00
:50K:/1234567890
JUAN PEREZ
BOGOTA
:59:/9876543210
HOTEL PARADISE
CARTAGENA
:70:PAYMENT FOR BOOKING
```

**Limitaciones**:
- ❌ Campos cortos y ambiguos
- ❌ Información limitada (140 caracteres en campo de referencia)
- ❌ Difícil de automatizar
- ❌ No soporta datos estructurados (direcciones completas, facturas)
- ❌ Requiere interpretación manual

#### Ventajas de ISO 20022

**Ejemplo de mensaje ISO 20022 (pain.001 - Payment Initiation)**:
```xml
<CstmrCdtTrfInitn>
  <PmtInf>
    <PmtInfId>PMT-2026-02-25-001</PmtInfId>
    <PmtMtd>TRF</PmtMtd>
    <ReqdExctnDt>2026-02-25</ReqdExctnDt>
    <Dbtr>
      <Nm>TravelHub Colombia SAS</Nm>
      <PstlAdr>
        <StrtNm>Calle 123</StrtNm>
        <BldgNb>45-67</BldgNb>
        <PstCd>110111</PstCd>
        <TwnNm>Bogotá</TwnNm>
        <Ctry>CO</Ctry>
      </PstlAdr>
      <Id>
        <OrgId>
          <Othr>
            <Id>NIT900123456</Id>
          </Othr>
        </OrgId>
      </Id>
    </Dbtr>
    <CdtTrfTxInf>
      <PmtId>
        <InstrId>INSTR-001</InstrId>
        <EndToEndId>BK-789XYZ</EndToEndId>
      </PmtId>
      <Amt>
        <InstdAmt Ccy="USD">450.00</InstdAmt>
      </Amt>
      <Cdtr>
        <Nm>Hotel Paradise</Nm>
        <PstlAdr>
          <StrtNm>Avenida San Martín</StrtNm>
          <BldgNb>100</BldgNb>
          <PstCd>130001</PstCd>
          <TwnNm>Cartagena</TwnNm>
          <Ctry>CO</Ctry>
        </PstlAdr>
      </Cdtr>
      <RmtInf>
        <Ustrd>Payment for booking BK-789XYZ - Hotel Paradise - Check-in: 2026-03-15</Ustrd>
        <Strd>
          <RfrdDocInf>
            <Tp>
              <CdOrPrtry>
                <Cd>CINV</Cd>
              </CdOrPrtry>
            </Tp>
            <Nb>INV-2026-001</Nb>
          </RfrdDocInf>
        </Strd>
      </RmtInf>
    </CdtTrfTxInf>
  </PmtInf>
</CstmrCdtTrfInitn>
```

**Ventajas**:
- ✅ **Datos ricos**: Direcciones completas, referencias de facturas, propósito del pago
- ✅ **Estructurado**: Fácil de parsear y automatizar
- ✅ **Extensible**: Campos adicionales sin romper compatibilidad
- ✅ **Interoperable**: Funciona entre diferentes sistemas y países
- ✅ **Trazabilidad**: End-to-end tracking de pagos
- ✅ **Cumplimiento**: Facilita AML (Anti-Money Laundering) y KYC (Know Your Customer)

### 2.3 Arquitectura de ISO 20022

#### Componentes Principales

1. **Business Model**: Modelo conceptual de procesos de negocio
2. **Message Definitions**: Definiciones de mensajes (XML Schema)
3. **Data Dictionary**: Diccionario de elementos de datos
4. **Message Transport**: Protocolos de transporte (SWIFT, HTTP, MQ)

#### Estructura de Mensajes

Todos los mensajes ISO 20022 siguen una estructura consistente:

```
[business_area].[message_functionality].[variant].[version]

Ejemplos:
- pain.001.001.09: Payment Initiation (Customer Credit Transfer)
- pacs.008.001.08: Financial Institution Credit Transfer
- camt.053.001.08: Bank to Customer Statement
- acmt.001.001.05: Account Opening Instruction
```

**Áreas de negocio principales**:
- `pain`: Payments Initiation (iniciación de pagos)
- `pacs`: Payments Clearing and Settlement (compensación y liquidación)
- `camt`: Cash Management (gestión de efectivo)
- `acmt`: Account Management (gestión de cuentas)
- `reda`: Reference Data (datos de referencia)

### 2.4 Mensajes ISO 20022 Relevantes para TravelHub

#### A. pain.001 - Customer Credit Transfer Initiation

**Propósito**: Iniciar transferencia de crédito de TravelHub a hotel.

**Flujo**: TravelHub → Banco TravelHub → Banco Hotel → Hotel

**Uso en TravelHub**: Pagar comisiones a hoteles al final del mes.

**Estructura simplificada**:
```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:pain.001.001.09">
  <CstmrCdtTrfInitn>
    <GrpHdr>
      <MsgId>MSG-2026-02-001</MsgId>
      <CreDtTm>2026-02-25T22:00:00Z</CreDtTm>
      <NbOfTxs>1</NbOfTxs>
      <CtrlSum>450.00</CtrlSum>
      <InitgPty>
        <Nm>TravelHub Colombia SAS</Nm>
      </InitgPty>
    </GrpHdr>
    <PmtInf>
      <PmtInfId>PMT-2026-02-001</PmtInfId>
      <PmtMtd>TRF</PmtMtd>
      <ReqdExctnDt>2026-02-26</ReqdExctnDt>
      <Dbtr>
        <Nm>TravelHub Colombia SAS</Nm>
      </Dbtr>
      <DbtrAcct>
        <Id>
          <IBAN>CO1234567890123456789012</IBAN>
        </Id>
      </DbtrAcct>
      <DbtrAgt>
        <FinInstnId>
          <BIC>BANCCOLOBB</BIC>
        </FinInstnId>
      </DbtrAgt>
      <CdtTrfTxInf>
        <PmtId>
          <EndToEndId>HOTEL-PARADISE-FEB2026</EndToEndId>
        </PmtId>
        <Amt>
          <InstdAmt Ccy="COP">1800000.00</InstdAmt>
        </Amt>
        <CdtrAgt>
          <FinInstnId>
            <BIC>DAVICOLOBB</BIC>
          </FinInstnId>
        </CdtrAgt>
        <Cdtr>
          <Nm>Hotel Paradise SAS</Nm>
        </Cdtr>
        <CdtrAcct>
          <Id>
            <IBAN>CO9876543210987654321098</IBAN>
          </Id>
        </CdtrAcct>
        <RmtInf>
          <Ustrd>Commission payment for February 2026 - 15 bookings</Ustrd>
        </RmtInf>
      </CdtTrfTxInf>
    </PmtInf>
  </CstmrCdtTrfInitn>
</Document>
```

**Elementos clave**:
- `MsgId`: ID único del mensaje
- `NbOfTxs`: Número de transacciones en el mensaje
- `CtrlSum`: Suma de control (total de montos)
- `PmtMtd`: Método de pago (TRF = Transfer)
- `ReqdExctnDt`: Fecha de ejecución solicitada
- `Dbtr`: Deudor (TravelHub)
- `Cdtr`: Acreedor (Hotel)
- `EndToEndId`: ID end-to-end para trazabilidad
- `RmtInf`: Información de remesa (propósito del pago)

#### B. pacs.008 - Financial Institution Credit Transfer

**Propósito**: Transferencia entre instituciones financieras.

**Flujo**: Banco TravelHub → Banco Hotel

**Uso**: Procesamiento interbancario (generalmente transparente para TravelHub).

#### C. camt.053 - Bank to Customer Statement

**Propósito**: Estado de cuenta del banco al cliente.

**Flujo**: Banco TravelHub → TravelHub

**Uso en TravelHub**: Reconciliación automática de pagos recibidos y enviados.

**Estructura simplificada**:
```xml
<Document xmlns="urn:iso:std:iso:20022:tech:xsd:camt.053.001.08">
  <BkToCstmrStmt>
    <GrpHdr>
      <MsgId>STMT-2026-02-25</MsgId>
      <CreDtTm>2026-02-25T23:00:00Z</CreDtTm>
    </GrpHdr>
    <Stmt>
      <Id>STMT-001</Id>
      <CreDtTm>2026-02-25T23:00:00Z</CreDtTm>
      <Acct>
        <Id>
          <IBAN>CO1234567890123456789012</IBAN>
        </Id>
        <Ccy>COP</Ccy>
      </Acct>
      <Bal>
        <Tp>
          <CdOrPrtry>
            <Cd>OPBD</Cd> <!-- Opening Balance -->
          </CdOrPrtry>
        </Tp>
        <Amt Ccy="COP">50000000.00</Amt>
        <CdtDbtInd>CRDT</CdtDbtInd>
        <Dt>2026-02-25</Dt>
      </Bal>
      <Bal>
        <Tp>
          <CdOrPrtry>
            <Cd>CLBD</Cd> <!-- Closing Balance -->
          </CdOrPrtry>
        </Tp>
        <Amt Ccy="COP">48200000.00</Amt>
        <CdtDbtInd>CRDT</CdtDbtInd>
        <Dt>2026-02-25</Dt>
      </Bal>
      <Ntry>
        <Amt Ccy="COP">1800000.00</Amt>
        <CdtDbtInd>DBIT</CdtDbtInd>
        <Sts>BOOK</Sts>
        <BookgDt>2026-02-25</BookgDt>
        <ValDt>2026-02-25</ValDt>
        <BkTxCd>
          <Domn>
            <Cd>PMNT</Cd>
            <Fmly>
              <Cd>ICDT</Cd>
              <SubFmlyCd>ESCT</SubFmlyCd>
            </Fmly>
          </Domn>
        </BkTxCd>
        <NtryDtls>
          <TxDtls>
            <Refs>
              <EndToEndId>HOTEL-PARADISE-FEB2026</EndToEndId>
            </Refs>
            <RltdPties>
              <Cdtr>
                <Nm>Hotel Paradise SAS</Nm>
              </Cdtr>
            </RltdPties>
            <RmtInf>
              <Ustrd>Commission payment for February 2026</Ustrd>
            </RmtInf>
          </TxDtls>
        </NtryDtls>
      </Ntry>
    </Stmt>
  </BkToCstmrStmt>
</Document>
```

**Uso para reconciliación**:
1. TravelHub recibe camt.053 diariamente del banco
2. Parser extrae transacciones con `EndToEndId`
3. Sistema de reconciliación matchea con pagos enviados
4. Actualiza estado de pagos a hoteles automáticamente

#### D. camt.054 - Bank to Customer Debit/Credit Notification

**Propósito**: Notificación en tiempo real de débitos/créditos.

**Flujo**: Banco → TravelHub (webhook o polling)

**Uso en TravelHub**: Notificación inmediata cuando un cliente paga una reserva.

### 2.5 Beneficios de ISO 20022 para TravelHub

#### 1. Reconciliación Automática

**Problema actual**:
- Reconciliación manual entre SAP y sistema de pagos
- 2-3% de errores requieren 3-4 días de ajustes
- 20 archivos Excel para 6 países

**Solución con ISO 20022**:
```
Flujo automatizado:
1. TravelHub envía pain.001 con EndToEndId = bookingId
2. Banco procesa pago
3. Banco envía camt.053 con mismo EndToEndId
4. Sistema de reconciliación matchea automáticamente
5. Actualiza estado en base de datos
6. Genera reporte para contabilidad
```

**Resultado**:
- ✅ Reducción del 95% en errores de reconciliación
- ✅ Tiempo de reconciliación: 3-4 días → 1 hora
- ✅ Eliminación de 20 archivos Excel

#### 2. Pagos Transfronterizos Eficientes

**Problema actual**:
- Pagos a hoteles en 6 países con diferentes monedas
- Información limitada en transferencias
- Difícil rastrear pagos internacionales

**Solución con ISO 20022**:
- Datos ricos: dirección completa, propósito del pago, referencia de factura
- Trazabilidad end-to-end con `EndToEndId`
- Soporte multi-moneda nativo
- Cumplimiento automático con regulaciones locales

#### 3. Cumplimiento Regulatorio

**Regulaciones que requieren datos ricos**:
- **AML (Anti-Money Laundering)**: Identificación completa de partes
- **KYC (Know Your Customer)**: Datos de clientes y beneficiarios
- **FATCA/CRS**: Reporte de transacciones internacionales
- **GDPR/LGPD**: Trazabilidad de datos personales

**ISO 20022 facilita cumplimiento**:
- Campos estructurados para identificación de partes
- Propósito del pago claramente definido
- Trazabilidad completa de transacciones
- Auditoría simplificada

#### 4. Integración con Bancos Modernos

**Bancos que soportan ISO 20022**:
- **Colombia**: Bancolombia, Davivienda, BBVA Colombia
- **Brasil**: Itaú, Bradesco, Banco do Brasil
- **México**: BBVA México, Santander México
- **Argentina**: Banco Galicia, Banco Macro
- **Internacional**: SWIFT (migración completa para 2025)

### 2.6 Implementación de ISO 20022 en TravelHub

#### Arquitectura de Integración

```
┌─────────────────────────────────────────────────────┐
│  TravelHub Payment Service                          │
│  - Genera mensajes pain.001                         │
│  - Procesa mensajes camt.053/054                    │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  ISO 20022 Adapter                                  │
│  - Validación de mensajes contra XSD                │
│  - Transformación JSON ↔ XML                        │
│  - Enriquecimiento de datos                         │
│  - Manejo de errores                                │
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────┐
│  Bank Integration Layer                             │
│  - APIs bancarias (REST/SOAP)                       │
│  - SWIFT Alliance Lite2                             │
│  - EBICS (Electronic Banking Internet Communication)│
└──────────────────┬──────────────────────────────────┘
                   │
                   ▼
              Bancos (CO, BR, MX, AR, PE, CL)
```

#### Componentes de Implementación

**1. ISO 20022 Message Generator**

```python
# Ejemplo simplificado en Python
from iso20022 import Pain001Generator

def create_payment_to_hotel(booking, hotel, amount):
    generator = Pain001Generator()
    
    message = generator.create_message(
        message_id=f"PMT-{booking.id}",
        creation_datetime=datetime.now(),
        number_of_transactions=1,
        control_sum=amount
    )
    
    payment_info = generator.add_payment_info(
        payment_info_id=f"PMTINF-{booking.id}",
        payment_method="TRF",
        requested_execution_date=booking.checkout_date + timedelta(days=7),
        debtor_name="TravelHub Colombia SAS",
        debtor_iban="CO1234567890123456789012",
        debtor_bic="BANCCOLOBB"
    )
    
    generator.add_credit_transfer(
        payment_info=payment_info,
        end_to_end_id=booking.id,
        amount=amount,
        currency=hotel.currency,
        creditor_name=hotel.legal_name,
        creditor_iban=hotel.iban,
        creditor_bic=hotel.bic,
        remittance_info=f"Commission for booking {booking.id} - {hotel.name}"
    )
    
    return generator.to_xml()
```

**2. ISO 20022 Message Parser**

```python
# Parser de camt.053 (Bank Statement)
from iso20022 import Camt053Parser

def process_bank_statement(xml_content):
    parser = Camt053Parser(xml_content)
    
    statement = parser.parse()
    
    for entry in statement.entries:
        if entry.credit_debit_indicator == "DBIT":
            # Pago saliente
            end_to_end_id = entry.end_to_end_id
            
            # Buscar pago pendiente
            payment = Payment.objects.get(
                booking_id=end_to_end_id,
                status="pending"
            )
            
            # Actualizar estado
            payment.status = "completed"
            payment.bank_reference = entry.entry_reference
            payment.value_date = entry.value_date
            payment.save()
            
            # Notificar al hotel
            notify_hotel_payment_completed(payment)
```

**3. Validación de Mensajes**

```python
from lxml import etree

def validate_iso20022_message(xml_content, message_type):
    # Cargar esquema XSD
    schema_path = f"schemas/{message_type}.xsd"
    schema = etree.XMLSchema(file=schema_path)
    
    # Parsear XML
    doc = etree.fromstring(xml_content.encode())
    
    # Validar
    if not schema.validate(doc):
        errors = schema.error_log
        raise ValidationError(f"Invalid ISO 20022 message: {errors}")
    
    return True
```

#### Flujo de Pago Completo

**Escenario**: TravelHub paga comisión a hotel después de checkout

```
1. Usuario hace checkout (2026-03-18)
2. Sistema calcula comisión del hotel (85% de $450 = $382.50)
3. Payment Service genera pain.001:
   - EndToEndId: "BK-789XYZ"
   - Amount: $382.50 USD
   - Creditor: Hotel Paradise
   - Remittance: "Commission for booking BK-789XYZ"
4. ISO 20022 Adapter valida mensaje contra XSD
5. Adapter envía a banco vía API
6. Banco procesa pago (1-2 días hábiles)
7. Banco envía camt.054 (notificación de débito)
8. TravelHub recibe notificación
9. Sistema actualiza estado: "pending" → "processing"
10. Banco envía camt.053 (estado de cuenta diario)
11. Sistema matchea EndToEndId y actualiza: "processing" → "completed"
12. Sistema envía email a hotel confirmando pago
```

### 2.7 Migración a ISO 20022

#### Fase 1: Preparación (Meses 1-2)
- [ ] Estudiar especificaciones ISO 20022 (pain, pacs, camt)
- [ ] Evaluar soporte de bancos actuales
- [ ] Seleccionar librería/framework (iso20022-py, iso20022-js)
- [ ] Descargar esquemas XSD oficiales

#### Fase 2: Implementación Básica (Meses 3-4)
- [ ] Implementar generador de pain.001
- [ ] Implementar parser de camt.053
- [ ] Integrar con 1 banco piloto (Colombia)
- [ ] Probar pagos en ambiente sandbox

#### Fase 3: Reconciliación Automática (Meses 5-6)
- [ ] Implementar sistema de matching por EndToEndId
- [ ] Migrar reconciliación manual a automática
- [ ] Eliminar archivos Excel
- [ ] Integrar con SAP vía API

#### Fase 4: Expansión Multi-País (Meses 7-9)
- [ ] Integrar bancos en Brasil, México, Argentina
- [ ] Implementar conversión de monedas
- [ ] Soportar regulaciones locales por país
- [ ] Escalar a todos los hoteles

#### Fase 5: Optimización (Meses 10-12)
- [ ] Implementar camt.054 para notificaciones en tiempo real
- [ ] Optimizar performance (procesamiento batch)
- [ ] Implementar dashboard de pagos en tiempo real
- [ ] Capacitar equipo de finanzas

---

## 3. Comparación PCI DSS vs ISO 20022

| Aspecto | PCI DSS | ISO 20022 |
|---------|---------|-----------|
| **Propósito** | Seguridad de datos de tarjetas | Mensajería de pagos financieros |
| **Alcance** | Tarjetas de crédito/débito | Transferencias bancarias, pagos B2B |
| **Tipo** | Estándar de seguridad | Estándar de mensajería |
| **Obligatorio** | Sí (para procesar tarjetas) | No (pero creciente adopción) |
| **Formato** | N/A (políticas y controles) | XML/JSON |
| **Auditoría** | Anual (QSA o SAQ) | No requiere auditoría |
| **Costo** | Alto (cumplimiento completo) | Medio (implementación técnica) |
| **Beneficio** | Reducción de riesgo de brechas | Automatización, eficiencia, cumplimiento |

**Relación entre ambos**:
- **PCI DSS**: Protege datos de tarjetas cuando clientes pagan reservas
- **ISO 20022**: Facilita pagos de TravelHub a hoteles (transferencias bancarias)

**Uso conjunto en TravelHub**:
1. Cliente paga reserva con tarjeta → **PCI DSS** (tokenización, TLS, logging)
2. TravelHub paga comisión a hotel → **ISO 20022** (pain.001, reconciliación automática)

---

## 4. Conclusión y Recomendaciones

### Para PCI DSS

**Estrategia recomendada**: **Reducir alcance del CDE**
- Usar tokenización con Stripe/Adyen
- NO almacenar datos de tarjetas en TravelHub
- Cumplir con SAQ A (el más simple)
- Inversión estimada: $50,000 - $100,000 USD/año

**ROI**:
- ✅ Reducción del 90% en costo de cumplimiento vs. PCI DSS completo
- ✅ Menor riesgo de brechas (datos de tarjetas nunca en TravelHub)
- ✅ Más rápido time-to-market (no esperar auditorías)

### Para ISO 20022

**Estrategia recomendada**: **Adopción gradual**
- Comenzar con pain.001 y camt.053
- Integrar con bancos en Colombia primero
- Expandir a otros países progresivamente
- Inversión estimada: $80,000 - $150,000 USD (implementación inicial)

**ROI**:
- ✅ Reducción del 95% en errores de reconciliación
- ✅ Tiempo de reconciliación: 3-4 días → 1 hora
- ✅ Eliminación de 20 archivos Excel
- ✅ Habilitación de expansión a Brasil y otros mercados

### Priorización

**Fase 1 (Meses 1-6)**: PCI DSS
- Crítico para operar legalmente
- Riesgo alto si no se cumple (multas, pérdida de capacidad de procesar pagos)

**Fase 2 (Meses 7-12)**: ISO 20022
- Mejora operacional significativa
- Habilita escalabilidad internacional
- Reduce costos operacionales

---

Content was rephrased for compliance with licensing restrictions.

**Referencias:**
[1] PCI DSS 4.0 Compliance Guide - https://www.pcicompliance.com/pci-dss-4-0/
[2] ISO 20022 Standard Explained - https://banq.ai/standards/iso-20022
[3] ISO 20022 Payment Messaging - https://prioritycommerce.com/resource-center/what-is-iso-20022/
[4] PCI Security Standards Council - https://www.pcisecuritystandards.org/
