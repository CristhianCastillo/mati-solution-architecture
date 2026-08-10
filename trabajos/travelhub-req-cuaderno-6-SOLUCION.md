# Cuaderno 6 - Alternativas de Solución para TravelHub

---

## Alternativa 1: Migración Completa a Microservicios con Infraestructura Multi-Región

| Item | Descripción |
|------|-------------|
| **Nombre de la Alternativa** | Migración Completa a Microservicios con Infraestructura Multi-Región |
| **Descripción** | Descomponer el monolito Java en microservicios independientes desplegados en contenedores orquestados con Kubernetes, distribuidos en múltiples regiones de AWS (São Paulo, México, Virginia). Incluye un API Gateway centralizado, event bus para sincronización en tiempo real entre servicios, y un data lake para consolidar datos de todos los sistemas satélite (Salesforce, SAP, BambooHR). Se reemplaza la sincronización semiautomática con PMS por integraciones event-driven en tiempo real. |
| **Principales Tecnologías Involucradas** | 1. AWS EKS (Kubernetes) + Multi-Region 2. Apache Kafka (Event Bus) 3. API Gateway (Kong/AWS API Gateway) 4. Terraform (IaC) + CI/CD (GitHub Actions) 5. AWS S3 + Glue + Redshift (Data Lake) |
| **Aspectos a Favor** | - Escalabilidad horizontal independiente por servicio, eliminando el cuello de botella del monolito. - Latencia reducida a <100ms con regiones en São Paulo y México. - Despliegues independientes por servicio sin ventanas de mantenimiento. - Sincronización en tiempo real elimina inconsistencias de datos. - Preparada para expansión a Brasil y multi-servicio. |
| **Aspectos Desfavorables** | - Alta complejidad técnica y riesgo de migración (13 años de código legacy). - Requiere reentrenamiento significativo del equipo actual de 12 personas en tecnología. - Período de transición largo con ambos sistemas coexistiendo. - Costos operativos de Kubernetes y multi-región significativamente mayores. - Riesgo de introducir problemas de consistencia distribuida. |
| **Presupuesto Requerido** | $2.8 - $3.5 millones USD (incluye infraestructura, licencias, consultoría externa y contingencia del 15%) |
| **Propiedad - Financiación** | Financiación interna con capital operativo de TravelHub. Posibilidad de financiamiento parcial mediante crédito tecnológico o inversión de riesgo. La propiedad intelectual es 100% de TravelHub. |
| **Recurso Humano Requerido** | - 4 arquitectos/ingenieros senior externos (consultoría, 18 meses). - Ampliación del equipo de DevOps de 2 a 5 personas. - Capacitación del equipo de tecnología actual (12 personas). - 1 Tech Lead interno dedicado a la migración. - Total estimado: 22 personas involucradas. |
| **Tiempo de Implementación** | 18 - 24 meses (fases: diseño 3 meses, migración core 9 meses, integración sistemas 4 meses, estabilización 2-4 meses) |

### Impacto de la Alternativa 1 en el Canvas

| Canvas del Negocio | Eliminar | Crecer | Reducir | Crear |
|---------------------|----------|--------|---------|-------|
| **Socios Clave** | | Proveedores de vuelos y multi-servicios gracias a APIs abiertas | | Alianzas con proveedores cloud multi-región |
| **Actividades Clave** | Soporte y resolución manual de inconsistencias de datos | Gestión de reservas y pagos (capacidad 100% en picos) | Cálculo manual de comisiones (automatizado) | Integración de sistemas y orquestación event-driven |
| **Recursos Clave** | Plataforma monolítica actual | Base de datos de clientes y reservas (distribuida y escalable) | | Arquitectura de microservicios, Data Lake, pipelines CI/CD |
| **Estructura de Costos** | | | Costos por ineficiencias operativas (reducción del 80% en tickets) | Inversión en infraestructura multi-región y Kubernetes |
| **Propuesta de Valor** | | Plataforma integral multi-servicio; Mayor alcance regional | | Sincronización en tiempo real de tarifas y disponibilidad |
| **Relación con Clientes** | Asistencia personalizada por incidentes (reducida drásticamente) | Reserva en la plataforma (sin rechazos en picos) | Resolución de disputas (menos inconsistencias) | Co-creación y Analytics con datos en tiempo real |
| **Canales** | | Plataforma WEB (menor latencia, mejor experiencia) | | APIs de integración B2B |
| **Fuentes de Ingreso** | | Comisiones por reservas (recuperación del $1M en temporadas altas) | Costos por oportunidades perdidas | Comisiones por multi-servicios; Integraciones B2B |
| **Segmentos de Clientes** | | Viajeros (mejor experiencia); Hoteles PYMES (sincronización real) | | Segmento B2B; Socios de negocio internacionales (Brasil) |

---

## Alternativa 2: Modernización Incremental con Strangler Fig Pattern y Plataforma de Integración

| Item | Descripción |
|------|-------------|
| **Nombre de la Alternativa** | Modernización Incremental con Strangler Fig Pattern y Plataforma de Integración |
| **Descripción** | Mantener el monolito como núcleo operativo mientras se extraen progresivamente los módulos más críticos (reservas, inventario, pagos) como servicios independientes usando el patrón Strangler Fig. Se implementa una plataforma de integración (iPaaS) para conectar los sistemas satélite (Salesforce, SAP, BambooHR) sin reescribirlos. Se agrega un CDN y caching distribuido para resolver la latencia inmediata. El monolito se va "estrangulando" gradualmente conforme los nuevos servicios lo reemplazan. |
| **Principales Tecnologías Involucradas** | 1. AWS ECS Fargate (contenedores serverless) 2. MuleSoft / AWS EventBridge (iPaaS/integración) 3. Amazon CloudFront + ElastiCache (CDN + caching) 4. AWS CodePipeline (CI/CD) 5. PostgreSQL Aurora (BD escalable compatible) |
| **Aspectos a Favor** | - Menor riesgo: el monolito sigue operando mientras se migra gradualmente. - Resultados tempranos: CDN y caching reducen latencia en semanas. - iPaaS resuelve la desconexión entre sistemas satélite sin reescribirlos. - Curva de aprendizaje menor para el equipo actual. - Inversión distribuida en el tiempo, no requiere capital inicial grande. |
| **Aspectos Desfavorables** | - Coexistencia prolongada de monolito y servicios nuevos genera complejidad operativa. - La integración vía iPaaS puede crear dependencia de un vendor (MuleSoft). - No resuelve completamente la escalabilidad hasta fases avanzadas. - Temporadas altas intermedias pueden seguir con rechazos parciales. - El monolito sigue acumulando deuda técnica mientras coexiste. |
| **Presupuesto Requerido** | $1.5 - $2.0 millones USD (distribuidos en fases anuales: Fase 1 $600K, Fase 2 $500K, Fase 3 $400-900K) |
| **Propiedad - Financiación** | Financiación interna con flujo de caja operativo. El costo distribuido permite financiar cada fase con los ahorros generados por la fase anterior. Propiedad 100% TravelHub, con posible licenciamiento iPaaS. |
| **Recurso Humano Requerido** | - 2 consultores externos especializados en integración (12 meses). - Ampliación del equipo de DevOps de 2 a 4 personas. - 1 ingeniero de integración iPaaS dedicado. - Capacitación gradual del equipo actual. - Total estimado: 17 personas involucradas. |
| **Tiempo de Implementación** | 12 - 18 meses (Fase 1: CDN + caching + CI/CD en 3 meses; Fase 2: iPaaS + integración sistemas en 5 meses; Fase 3: extracción de servicios críticos en 6-10 meses) |

### Impacto de la Alternativa 2 en el Canvas

| Canvas del Negocio | Eliminar | Crecer | Reducir | Crear |
|---------------------|----------|--------|---------|-------|
| **Socios Clave** | | Sistemas de gestión hotelera PMS (mejor integración) | | Proveedor iPaaS como socio tecnológico |
| **Actividades Clave** | | Gestión de reservas y pagos (mejora progresiva de capacidad) | Soporte y resolución manual de inconsistencias (iPaaS sincroniza datos) | Integración de sistemas mediante plataforma iPaaS |
| **Recursos Clave** | | Base de datos de clientes y reservas (migración a Aurora escalable) | Dependencia de la plataforma monolítica (se reduce gradualmente) | Plataforma de integración centralizada; CDN distribuido |
| **Estructura de Costos** | | | Costos por ineficiencias operativas (reducción gradual) | Licenciamiento iPaaS; Costos de CDN y caching |
| **Propuesta de Valor** | | Conexión directa hoteles-clientes (más confiable) | | Sincronización mejorada entre sistemas |
| **Relación con Clientes** | | Reserva en la plataforma (menos rechazos progresivamente) | Asistencia personalizada por incidentes (menos incidentes) | |
| **Canales** | | Plataforma WEB (latencia reducida con CDN) | | |
| **Fuentes de Ingreso** | | Comisiones por reservas (recuperación parcial en temporadas altas) | Costos por oportunidades perdidas (reducción gradual) | |
| **Segmentos de Clientes** | | Viajeros (mejor experiencia de respuesta) | | |

---

## Alternativa 3: Replatforming a Arquitectura Serverless Event-Driven en la Nube

| Item | Descripción |
|------|-------------|
| **Nombre de la Alternativa** | Replatforming a Arquitectura Serverless Event-Driven en la Nube |
| **Descripción** | Reconstruir los flujos críticos del negocio (reservas, pagos, inventario, comisiones) como funciones serverless orquestadas por eventos, eliminando la gestión de servidores. Se usa una arquitectura completamente event-driven donde cada cambio de tarifa, disponibilidad o reserva genera eventos que se propagan en tiempo real a todos los sistemas consumidores. Se implementa multi-región nativa de AWS con replicación automática. Los sistemas satélite se integran mediante event bridges y adaptadores serverless. |
| **Principales Tecnologías Involucradas** | 1. AWS Lambda + Step Functions (serverless) 2. Amazon EventBridge + SQS/SNS (event-driven) 3. Amazon DynamoDB Global Tables (BD multi-región) 4. AWS CDK (IaC) + CI/CD nativo 5. Amazon API Gateway + CloudFront (APIs + CDN) |
| **Aspectos a Favor** | - Escalabilidad automática e ilimitada: Lambda escala a demanda sin configuración. - Costo por uso: no se paga por infraestructura ociosa fuera de temporadas altas. - Multi-región nativa con DynamoDB Global Tables y replicación automática. - Eliminación total de la gestión de servidores (DevOps de 2 personas es suficiente). - Time-to-market rápido para nuevas funcionalidades (funciones independientes). |
| **Aspectos Desfavorables** | - Lock-in fuerte con AWS: migrar a otro proveedor sería muy costoso. - Límites de ejecución de Lambda (15 min) pueden afectar procesos batch complejos. - Cambio de paradigma radical para el equipo (de Java monolítico a serverless event-driven). - DynamoDB requiere rediseño completo del modelo de datos relacional actual. - Debugging y observabilidad en arquitecturas serverless distribuidas es más complejo. |
| **Presupuesto Requerido** | $1.8 - $2.5 millones USD (infraestructura serverless tiene menor costo operativo mensual pero mayor inversión inicial en rediseño y capacitación) |
| **Propiedad - Financiación** | Financiación interna. Posibilidad de acceder a créditos AWS para startups/scaleups en LATAM. Propiedad 100% TravelHub, pero con dependencia operativa de AWS. |
| **Recurso Humano Requerido** | - 3 arquitectos serverless externos (consultoría, 12 meses). - Reentrenamiento intensivo del equipo de tecnología (12 personas) en serverless. - 1 especialista en DynamoDB y modelado NoSQL. - Equipo de DevOps actual (2 personas) es suficiente con serverless. - Total estimado: 18 personas involucradas. |
| **Tiempo de Implementación** | 14 - 20 meses (Fase 1: diseño + capacitación 3 meses; Fase 2: reconstrucción flujos críticos 7 meses; Fase 3: integración sistemas satélite 3 meses; Fase 4: migración de datos y cutover 1-3 meses; estabilización 2-4 meses) |

### Impacto de la Alternativa 3 en el Canvas

| Canvas del Negocio | Eliminar | Crecer | Reducir | Crear |
|---------------------|----------|--------|---------|-------|
| **Socios Clave** | | Proveedores de vuelos y multi-servicios (APIs serverless facilitan integración) | | Dependencia estratégica con AWS como socio cloud |
| **Actividades Clave** | Soporte y resolución manual de inconsistencias (eventos en tiempo real eliminan desfases); Cálculo manual de comisiones (automatizado con Step Functions) | Gestión de reservas y pagos (escalabilidad ilimitada) | Onboarding y gestión de inventario (automatizado por eventos) | Gobernanza de datos event-driven; Automatización de operaciones serverless |
| **Recursos Clave** | Plataforma monolítica actual; Infraestructura AWS us-east-1 (reemplazada por multi-región) | | | Arquitectura serverless event-driven multi-región; Funciones Lambda como unidad de despliegue |
| **Estructura de Costos** | Costos por ineficiencias operativas (eliminados con automatización) | | Infraestructura y tecnología (modelo pay-per-use reduce costos fuera de picos) | Inversión en reentrenamiento y rediseño de modelo de datos |
| **Propuesta de Valor** | | Plataforma integral multi-servicio (serverless facilita agregar servicios); Mayor alcance y presencia regional | | Escalabilidad automática garantizada; Sincronización en tiempo real por eventos |
| **Relación con Clientes** | Asistencia personalizada por incidentes (drásticamente reducida) | Reserva en la plataforma (0% rechazos en picos) | Resolución de disputas | Relación automatizada mediante eventos y notificaciones en tiempo real |
| **Canales** | | Plataforma WEB (latencia mínima multi-región); APIs de integración B2B (serverless nativas) | | Canales B2B2B habilitados por APIs escalables |
| **Fuentes de Ingreso** | Costos por oportunidades perdidas (eliminados) | Comisiones por reservas (recuperación total del $1M+ en temporadas); Comisiones por multi-servicios | | Analytics como Servicio (habilitado por event streaming); Integraciones B2B |
| **Segmentos de Clientes** | | Viajeros (experiencia sin fricciones); Hoteles PYMES (datos en tiempo real); Segmento B2B | | Socios de negocio internacionales (Brasil habilitado por multi-región) |

---

## Resumen Comparativo

| Criterio | Alternativa 1: Microservicios | Alternativa 2: Strangler Fig + iPaaS | Alternativa 3: Serverless Event-Driven |
|----------|-------------------------------|--------------------------------------|----------------------------------------|
| **Presupuesto** | $2.8 - $3.5M | $1.5 - $2.0M | $1.8 - $2.5M |
| **Tiempo** | 18 - 24 meses | 12 - 18 meses | 14 - 20 meses |
| **Riesgo** | Alto | Bajo-Medio | Medio-Alto |
| **Escalabilidad resultante** | Alta | Media (progresiva) | Muy Alta (automática) |
| **Vendor Lock-in** | Bajo | Medio (iPaaS) | Alto (AWS) |
| **Complejidad para el equipo** | Alta | Baja-Media | Alta |
| **Resultados tempranos** | Lentos (6+ meses) | Rápidos (3 meses) | Medios (4-5 meses) |
