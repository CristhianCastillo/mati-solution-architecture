# Cronograma de Implementación — TravelHub Async Booking Engine

## Resumen Ejecutivo

| Aspecto | Detalle |
|---|---|
| **Solución** | TravelHub Async Booking Engine — Motor asíncrono de reservas event-driven |
| **Duración total** | 7 meses (Mes 1 – Mes 7) |
| **CAPEX** | 310.000 USD |
| **OPEX anual** | 170.000 USD |
| **Equipo requerido** | 11–14 personas (arquitecto, 4-6 devs, integraciones, DevOps/SRE, seguridad, QA, PM/PO) |
| **Problema que resuelve** | Pérdida de reservas en temporada alta (35-40% de peticiones rechazadas, ~1M USD por período) por limitaciones del monolito síncrono |

---

## Vista General del Cronograma (Diagrama de Gantt Textual)

```
Mes         |  M1  |  M2  |  M3  |  M4  |  M5  |  M6  |  M7  |
------------|------|------|------|------|------|------|------|
Fase 1      |██████|      |      |      |      |      |      |  Diseño y Arquitectura
Fase 2      |      |██████|██████|██████|      |      |      |  Desarrollo
Fase 3      |      |      |      |      |██████|      |      |  Integración
Fase 4      |      |      |      |      |      |██████|      |  Pruebas y Estabilización
Fase 5      |      |      |      |      |      |      |██████|  Despliegue Progresivo
------------|------|------|------|------|------|------|------|
Hitos       | H1   |  H2  |      |  H3  |  H4  |  H5  |  H6  |
```

---

## Hitos Principales

| ID | Hito | Mes | Criterio de Aceptación |
|---|---|---|---|
| H1 | Arquitectura aprobada | Fin M1 | Documento de arquitectura revisado y firmado por stakeholders |
| H2 | Infraestructura base operativa | Fin M2 | Ambientes dev/staging desplegados, pipelines CI/CD funcionales |
| H3 | Motor asíncrono funcional en staging | Fin M4 | Flujo completo de reserva asíncrona ejecutándose end-to-end en staging |
| H4 | Integraciones con PMS y canales validadas | Fin M5 | Al menos 3 PMS externos integrados y canales B2C/B2B conectados |
| H5 | Pruebas de carga superadas | Fin M6 | Sistema soporta 2x el pico histórico de temporada alta sin degradación |
| H6 | Go-live completado | Fin M7 | Motor en producción en al menos 3 países con monitoreo activo |

---

## Fase 1 — Diseño y Arquitectura (Mes 1)

**Objetivo:** Definir la arquitectura detallada del motor asíncrono, establecer estándares técnicos y preparar la infraestructura base.

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Levantamiento de requisitos funcionales y no funcionales con negocio y operaciones | PM/PO, Arquitecto | Documento de requisitos validado | — |
| S1 | Análisis de integraciones actuales con PMS externos y proveedores de pago | Ing. Integraciones | Mapa de integraciones y protocolos actuales | — |
| S2 | Diseño de la arquitectura event-driven: definición de dominios, eventos, colas (SQS), tópicos (SNS), flujos (Step Functions) y microservicios (EKS) | Arquitecto | Documento de Arquitectura de Solución (SAD) | Requisitos validados |
| S2 | Diseño del modelo de datos en DynamoDB y estrategia de caché con MemoryDB for Redis | Arquitecto, Dev Lead | Modelo de datos y estrategia de caché | Requisitos validados |
| S3 | Definición de estándares de seguridad: políticas IAM, cifrado KMS, gestión de secretos (Secrets Manager), reglas WAF | Esp. Seguridad | Documento de estándares de seguridad | SAD aprobado |
| S3 | Diseño de la estrategia de observabilidad: métricas CloudWatch, trazas X-Ray, alertas y dashboards operacionales | DevOps/SRE | Plan de observabilidad | SAD aprobado |
| S4 | Diseño del pipeline CI/CD (CodePipeline, Docker, GitHub) y estrategia de ambientes (dev, staging, producción) | DevOps/SRE | Documento de estrategia DevOps | SAD aprobado |
| S4 | Revisión y aprobación de la arquitectura con stakeholders técnicos y de negocio | Arquitecto, PM/PO | **Hito H1: Arquitectura aprobada** | Todos los documentos de diseño |

**Riesgos de la fase:**
- Requisitos incompletos por falta de disponibilidad de stakeholders de negocio.
- Subestimación de la complejidad de integración con PMS heterogéneos.

---

## Fase 2 — Desarrollo del Motor Asíncrono y Microservicios (Meses 2, 3 y 4)

**Objetivo:** Construir el motor de reservas asíncrono, los microservicios de soporte y la infraestructura cloud.

### Mes 2 — Infraestructura y Servicios Core

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Provisión de infraestructura AWS: VPC, subnets, EKS cluster, colas SQS, tópicos SNS, EventBridge, DynamoDB, MemoryDB | DevOps/SRE | Infraestructura base desplegada (IaC con CloudFormation/Terraform) | H1 |
| S1 | Configuración de pipelines CI/CD y ambientes dev/staging | DevOps/SRE | **Hito H2: Pipelines y ambientes operativos** | Infraestructura base |
| S2 | Desarrollo del microservicio de recepción de reservas (API Gateway + Lambda): validación de entrada, generación de acuse de recibo, publicación en cola SQS | Equipo Backend | Microservicio de ingesta desplegado en dev | H1, Infraestructura |
| S2 | Implementación de autenticación y autorización (API Gateway + IAM + Cognito/JWT) | Esp. Seguridad, Backend | Capa de seguridad en API Gateway | Infraestructura base |
| S3 | Desarrollo del orquestador de reservas con AWS Step Functions: flujo de validación de disponibilidad, bloqueo de inventario, confirmación y notificación | Equipo Backend | State machine de orquestación en dev | Microservicio de ingesta |
| S3 | Desarrollo del microservicio de inventario y disponibilidad (EKS + DynamoDB + MemoryDB) | Equipo Backend | Servicio de inventario en dev | Infraestructura base |
| S4 | Desarrollo del microservicio de notificaciones (SNS + SES + Pinpoint): confirmación, cambios de estado, errores | Equipo Backend | Servicio de notificaciones en dev | Orquestador |
| S4 | Implementación de la lógica de reintentos automáticos y dead-letter queues (DLQ) para manejo de fallos | Equipo Backend | Política de reintentos y DLQ configuradas | Orquestador, colas SQS |

### Mes 3 — Servicios de Negocio y Validación Multi-País

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Desarrollo del microservicio de validación multi-país: reglas de impuestos, políticas de cancelación y requisitos legales por país | Equipo Backend | Servicio de validación multi-país en dev | Orquestador |
| S1 | Desarrollo del microservicio de seguimiento de estado de reserva (histórico y buzón del cliente) | Equipo Backend | Servicio de tracking en dev | Servicio de inventario |
| S2 | Desarrollo de la capa de API para canales B2C (viajeros directos) | Equipo Backend | API B2C en staging | Todos los servicios core |
| S2 | Desarrollo de la capa de API para canales B2B (agencias y operadores turísticos) | Equipo Backend | API B2B en staging | Todos los servicios core |
| S3 | Implementación de cifrado en tránsito y en reposo (KMS) para mensajes y datos de reserva | Esp. Seguridad | Cifrado aplicado en todos los servicios | Servicios desplegados |
| S3 | Configuración de reglas WAF para protección de APIs públicas | Esp. Seguridad | Reglas WAF activas en staging | APIs desplegadas |
| S4 | Pruebas unitarias y de integración de todos los microservicios | QA, Equipo Backend | Reporte de cobertura de pruebas (>80%) | Todos los microservicios |
| S4 | Configuración de observabilidad: dashboards CloudWatch, trazas X-Ray, alertas de SLA | DevOps/SRE | Dashboards y alertas operativas en staging | Servicios desplegados |

### Mes 4 — Integración Interna y Estabilización en Staging

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Integración end-to-end del flujo completo de reserva asíncrona en staging | Equipo Backend, QA | Flujo completo funcional en staging | Todos los microservicios |
| S1 | Desarrollo del adaptador de compatibilidad con el monolito actual (coexistencia durante migración) | Equipo Backend | Adaptador monolito ↔ motor asíncrono | Flujo completo |
| S2 | Pruebas de integración del flujo completo: reserva → validación → confirmación → notificación | QA | Reporte de pruebas de integración | Flujo completo en staging |
| S2 | Pruebas de resiliencia: simulación de fallos en proveedores, caídas de servicios, timeouts | QA, DevOps/SRE | Reporte de resiliencia | Flujo completo en staging |
| S3 | Corrección de defectos y optimización de rendimiento | Equipo Backend | Defectos críticos resueltos | Reportes de pruebas |
| S3 | Documentación técnica de APIs, flujos de eventos y runbooks operacionales | Arquitecto, Backend | Documentación técnica completa | Servicios estabilizados |
| S4 | Validación del flujo completo con usuarios internos (operaciones y ventas) | PM/PO, QA | **Hito H3: Motor asíncrono funcional en staging** | Defectos resueltos |
| S4 | Revisión de seguridad y auditoría de configuración AWS | Esp. Seguridad | Reporte de auditoría de seguridad | Servicios estabilizados |

**Riesgos de la fase:**
- Curva de aprendizaje del equipo en patrones event-driven y servicios gestionados de AWS.
- Complejidad en la consistencia eventual entre microservicios (inventario, reservas, notificaciones).
- Sobrecostos en servicios cloud si el dimensionamiento no se ajusta correctamente en staging.

---

## Fase 3 — Integración con Canales y PMS Externos (Mes 5)

**Objetivo:** Conectar el motor asíncrono con los PMS hoteleros externos, proveedores de pago y canales de distribución.

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Desarrollo de adaptadores para los 3 PMS externos prioritarios (según volumen de reservas) | Ing. Integraciones, Backend | Adaptadores PMS desplegados en staging | H3, Mapa de integraciones |
| S1 | Configuración de sincronización bidireccional de inventario con PMS externos | Ing. Integraciones | Sincronización de inventario validada | Adaptadores PMS |
| S2 | Integración con proveedores de pago existentes a través del flujo asíncrono | Ing. Integraciones | Flujo de pago integrado en staging | H3 |
| S2 | Integración del canal B2C (plataforma web/móvil de viajeros) con el nuevo motor | Equipo Backend | Canal B2C conectado al motor asíncrono | H3 |
| S3 | Integración del canal B2B (APIs para agencias y operadores turísticos) | Equipo Backend | Canal B2B conectado al motor asíncrono | H3 |
| S3 | Pruebas de integración con PMS reales en ambiente de staging | QA, Ing. Integraciones | Reporte de pruebas de integración PMS | Adaptadores y sincronización |
| S4 | Pruebas end-to-end con datos reales (reservas de prueba con hoteles piloto) | QA, PM/PO | Reporte de pruebas con hoteles piloto | Todas las integraciones |
| S4 | Validación de la experiencia del cliente: acuse de recibo, tiempos de confirmación, notificaciones | PM/PO, QA | **Hito H4: Integraciones validadas** | Pruebas end-to-end |

**Riesgos de la fase:**
- Incompatibilidad o limitaciones de algunos PMS hoteleros con el modelo asíncrono.
- Resistencia de agencias B2B que dependen de respuestas síncronas en sus integraciones actuales.
- Inconsistencias de inventario durante la sincronización bidireccional con PMS externos.

---

## Fase 4 — Pruebas de Carga y Estabilización (Mes 6)

**Objetivo:** Validar que el sistema soporta los picos de demanda de temporada alta y estabilizar para producción.

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Diseño de escenarios de pruebas de carga: simulación de Semana Santa y Navidad (2x pico histórico) | QA, Arquitecto | Plan de pruebas de carga | H4 |
| S1 | Ejecución de pruebas de carga progresivas: 1x, 1.5x, 2x del pico histórico | QA, DevOps/SRE | Resultados de pruebas de carga | Plan de pruebas |
| S2 | Ajuste de autoescalado: configuración de políticas de escalado en EKS, Lambda concurrency, throughput de SQS/DynamoDB | DevOps/SRE | Políticas de autoescalado optimizadas | Resultados de carga |
| S2 | Pruebas de estrés y punto de quiebre: identificar límites del sistema | QA, DevOps/SRE | Reporte de límites y recomendaciones | Resultados de carga |
| S3 | Pruebas de recuperación ante desastres: simulación de caída de servicios, failover, recuperación | DevOps/SRE | Reporte de DR y tiempos de recuperación | Autoescalado configurado |
| S3 | Pruebas de seguridad: penetration testing, validación de cifrado, revisión de permisos IAM | Esp. Seguridad | Reporte de seguridad final | Servicios estabilizados |
| S4 | Corrección de defectos de rendimiento y seguridad identificados | Equipo Backend, DevOps | Defectos resueltos | Reportes de pruebas |
| S4 | Aprobación para go-live: revisión final con stakeholders técnicos y de negocio | Arquitecto, PM/PO | **Hito H5: Pruebas de carga superadas, aprobación go-live** | Todos los reportes |

**Riesgos de la fase:**
- Sobrecosto por necesidad de pruebas de carga adicionales si los resultados iniciales no son satisfactorios.
- Descubrimiento tardío de cuellos de botella en DynamoDB o MemoryDB bajo carga extrema.
- Configuración inadecuada de autoescalado que genere sobrecostos en producción.

---

## Fase 5 — Despliegue Progresivo y Acompañamiento Operativo (Mes 7)

**Objetivo:** Desplegar el motor asíncrono en producción de forma progresiva por país, con acompañamiento y monitoreo continuo.

| Sem. | Actividad | Responsable | Entregable | Dependencia |
|---|---|---|---|---|
| S1 | Despliegue en producción — País piloto (Colombia): canary deployment con 10% del tráfico | DevOps/SRE, Backend | Motor en producción (Colombia, 10% tráfico) | H5 |
| S1 | Monitoreo intensivo 24/7: dashboards, alertas, war room operativo | DevOps/SRE, Operaciones | Reporte de monitoreo día 1-3 | Despliegue piloto |
| S2 | Incremento progresivo de tráfico en Colombia: 10% → 50% → 100% | DevOps/SRE | Colombia al 100% en motor asíncrono | Monitoreo satisfactorio |
| S2 | Despliegue en producción — Segundo grupo de países (Perú, Ecuador) | DevOps/SRE, Backend | Motor en producción (3 países) | Colombia estable |
| S3 | Despliegue en producción — Tercer grupo de países (México, Chile, Argentina) | DevOps/SRE, Backend | Motor en producción (6 países) | Segundo grupo estable |
| S3 | Capacitación al equipo de operaciones y soporte en el nuevo flujo de reservas | PM/PO, Arquitecto | Equipo de operaciones capacitado | Despliegue activo |
| S4 | Capacitación a agencias B2B y operadores turísticos en el nuevo modelo asíncrono | PM/PO, Ing. Integraciones | Partners capacitados | Despliegue activo |
| S4 | Cierre del proyecto: lecciones aprendidas, documentación final, transición a operaciones | PM/PO, Arquitecto | **Hito H6: Go-live completado** | Todos los países en producción |

**Riesgos de la fase:**
- Resistencia al cambio del cliente final ante la confirmación diferida (no inmediata).
- Inconsistencias de inventario durante la coexistencia del monolito y el motor asíncrono.
- Incidentes en producción durante el despliegue progresivo que requieran rollback.

---

## Matriz de Responsabilidades (RACI)

| Actividad | Arquitecto | Backend (4-6) | Integraciones | DevOps/SRE | Seguridad | QA | PM/PO |
|---|---|---|---|---|---|---|---|
| Diseño de arquitectura | **R** | C | C | C | C | I | **A** |
| Provisión de infraestructura | C | I | I | **R** | C | I | **A** |
| Desarrollo de microservicios | C | **R** | C | I | C | I | **A** |
| Integración con PMS externos | C | C | **R** | I | I | C | **A** |
| Configuración de seguridad | C | I | I | C | **R** | I | **A** |
| Pruebas de carga | C | C | I | **R** | I | **R** | **A** |
| Pruebas de integración | I | C | C | I | I | **R** | **A** |
| Despliegue progresivo | C | C | C | **R** | C | C | **A** |
| Capacitación y gestión del cambio | C | I | C | I | I | I | **R/A** |

*R = Responsable, A = Aprobador, C = Consultado, I = Informado*

---

## Distribución Estimada del Presupuesto (CAPEX: 310.000 USD)

| Fase | % del CAPEX | Monto Estimado (USD) |
|---|---|---|
| Fase 1 — Diseño y Arquitectura | 10% | 31.000 |
| Fase 2 — Desarrollo | 45% | 139.500 |
| Fase 3 — Integración | 18% | 55.800 |
| Fase 4 — Pruebas y Estabilización | 15% | 46.500 |
| Fase 5 — Despliegue y Acompañamiento | 12% | 37.200 |
| **Total** | **100%** | **310.000** |

---

## Dependencias Críticas

```
Fase 1 (Diseño) ──► Fase 2 (Desarrollo) ──► Fase 3 (Integración) ──► Fase 4 (Pruebas) ──► Fase 5 (Despliegue)
     │                      │                        │
     ▼                      ▼                        ▼
 Aprobación de         Infraestructura          PMS externos
 stakeholders          AWS operativa            disponibles para
 (H1)                  (H2)                     integración (H4)
```

- **Fase 2 no puede iniciar** sin la aprobación de la arquitectura (H1).
- **Fase 3 depende** de que el motor asíncrono esté funcional en staging (H3).
- **Fase 4 depende** de que las integraciones con PMS y canales estén validadas (H4).
- **Fase 5 depende** de la aprobación de go-live tras superar pruebas de carga (H5).
- **Dependencia externa:** Disponibilidad de los equipos técnicos de los PMS hoteleros para pruebas de integración (Fase 3).

---

## Plan de Mitigación de Riesgos Principales

| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|
| Retrasos por complejidad de integración con PMS | Alta | Alto | Priorizar los 3 PMS de mayor volumen; usar adaptadores genéricos para el resto |
| Sobrecostos en servicios cloud | Media | Medio | Configurar alertas de billing, usar reserved capacity para servicios predecibles |
| Resistencia al cambio del cliente final | Media | Alto | Comunicación proactiva, UX clara de seguimiento de estado, tiempos de confirmación < 2 min |
| Inconsistencias de inventario en migración | Media | Alto | Período de coexistencia con validación dual (monolito + motor asíncrono) |
| Curva de aprendizaje del equipo | Media | Medio | Capacitación en Mes 1, pair programming, arquitecto como coach técnico |
| Vendor lock-in con AWS | Baja | Alto | Diseño con abstracciones sobre servicios cloud, documentar alternativas |
| Vulnerabilidades de seguridad | Baja | Alto | Auditoría de seguridad en Fase 2 y Fase 4, penetration testing antes de go-live |
