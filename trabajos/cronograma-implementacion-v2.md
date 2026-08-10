# Cronograma de Implementación v2 — TravelHub Async Booking Engine
## Con Fechas Específicas y Ciclo de Releases

| Aspecto | Detalle |
|---|---|
| **Fecha de inicio** | 1 de junio de 2026 |
| **Fecha de finalización** | 26 de diciembre de 2026 |
| **Duración total** | 7 meses |
| **CAPEX** | 310.000 USD |
| **OPEX anual** | 170.000 USD |

---

## Calendario de Eventos Clave

| Fecha | Evento | Descripción |
|---|---|---|
| **01 Jun 2026** | 🚀 **Project Kickoff** | Inicio oficial del proyecto. Reunión de arranque con todos los stakeholders, presentación de alcance, equipo, plan y gobernanza. |
| **26 Jun 2026** | ✅ Arquitectura aprobada (H1) | Documento de arquitectura firmado. Gate de paso a desarrollo. |
| **14 Ago 2026** | 🟡 **Beta Release 1 — Core Engine** | Primera versión funcional del motor asíncrono en ambiente dev: ingesta de reservas, orquestador Step Functions, inventario y notificaciones. Flujo básico end-to-end operativo. |
| **25 Sep 2026** | 🟡 **Beta Release 2 — Full Feature** | Motor completo en staging: validación multi-país, APIs B2C/B2B, seguridad, observabilidad. Adaptador de coexistencia con monolito. Validado por usuarios internos (H3). |
| **23 Oct 2026** | 🟠 **Release Candidate 1 (RC1)** | Motor integrado con PMS externos, canales y proveedores de pago. Integraciones validadas con hoteles piloto (H4). Listo para pruebas de carga. |
| **20 Nov 2026** | 🟠 **Release Candidate 2 (RC2)** | Versión estabilizada post-pruebas de carga y seguridad. Defectos corregidos. Aprobación de go-live por stakeholders (H5). |
| **26 Dic 2026** | 🟢 **Final Release — Go-Live** | Motor en producción en los 6 países. Despliegue progresivo completado. Equipo y partners capacitados (H6). |

---

## Vista Mensual con Fechas

```
Jun 2026     Jul 2026     Ago 2026     Sep 2026     Oct 2026     Nov 2026     Dic 2026
|────────────|────────────|────────────|────────────|────────────|────────────|────────────|
|  FASE 1    |       FASE 2 — Desarrollo                        |            |            |
|  Diseño    |  Infra &   |  Negocio & |  Integr.   |            |            |            |
|  & Arq.    |  Core      |  Multi-país|  Interna   |  FASE 3    |  FASE 4    |  FASE 5    |
|            |            |            |  & Staging |  Integrac. |  Pruebas   |  Despliegue|
|────────────|────────────|────────────|────────────|────────────|────────────|────────────|
🚀Kickoff   H2           🟡Beta 1     🟡Beta 2    🟠RC1        🟠RC2        🟢Final
01-Jun       31-Jul       14-Ago       25-Sep       23-Oct       20-Nov       26-Dic
     H1
     26-Jun
```

---

## Mes 1 — Junio 2026: Diseño y Arquitectura

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| **01 Jun** | — | 🚀 **PROJECT KICKOFF** — Reunión de arranque con stakeholders técnicos y de negocio. Presentación de alcance, equipo, cronograma, riesgos y gobernanza del proyecto. | PM/PO, Arquitecto | Acta de kickoff, plan de proyecto aprobado |
| 01–05 Jun | S1 | Levantamiento de requisitos funcionales y no funcionales con negocio y operaciones | PM/PO, Arquitecto | Documento de requisitos validado |
| 01–05 Jun | S1 | Análisis de integraciones actuales con PMS externos y proveedores de pago | Ing. Integraciones | Mapa de integraciones y protocolos |
| 08–12 Jun | S2 | Diseño de la arquitectura event-driven: dominios, eventos, colas SQS, tópicos SNS, flujos Step Functions, microservicios EKS | Arquitecto | Documento de Arquitectura de Solución (SAD) |
| 08–12 Jun | S2 | Diseño del modelo de datos en DynamoDB y estrategia de caché con MemoryDB for Redis | Arquitecto, Dev Lead | Modelo de datos y estrategia de caché |
| 15–19 Jun | S3 | Definición de estándares de seguridad: IAM, KMS, Secrets Manager, WAF | Esp. Seguridad | Documento de estándares de seguridad |
| 15–19 Jun | S3 | Diseño de la estrategia de observabilidad: CloudWatch, X-Ray, alertas y dashboards | DevOps/SRE | Plan de observabilidad |
| 22–26 Jun | S4 | Diseño del pipeline CI/CD y estrategia de ambientes (dev, staging, producción) | DevOps/SRE | Documento de estrategia DevOps |
| **26 Jun** | S4 | ✅ **HITO H1: Arquitectura aprobada** — Revisión y firma por stakeholders | Arquitecto, PM/PO | SAD aprobado, gate de paso a Fase 2 |

---

## Mes 2 — Julio 2026: Infraestructura y Servicios Core

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 29 Jun–03 Jul | S1 | Provisión de infraestructura AWS: VPC, subnets, EKS cluster, SQS, SNS, EventBridge, DynamoDB, MemoryDB (IaC) | DevOps/SRE | Infraestructura base desplegada |
| 29 Jun–03 Jul | S1 | Configuración de pipelines CI/CD y ambientes dev/staging | DevOps/SRE | Pipelines operativos |
| 06–10 Jul | S2 | Desarrollo del microservicio de recepción de reservas (API Gateway + Lambda) | Equipo Backend | Microservicio de ingesta en dev |
| 06–10 Jul | S2 | Implementación de autenticación y autorización (API Gateway + Cognito/JWT) | Esp. Seguridad, Backend | Capa de seguridad en API Gateway |
| 13–17 Jul | S3 | Desarrollo del orquestador de reservas con Step Functions | Equipo Backend | State machine de orquestación en dev |
| 13–17 Jul | S3 | Desarrollo del microservicio de inventario y disponibilidad (EKS + DynamoDB + MemoryDB) | Equipo Backend | Servicio de inventario en dev |
| 20–24 Jul | S4 | Desarrollo del microservicio de notificaciones (SNS + SES + Pinpoint) | Equipo Backend | Servicio de notificaciones en dev |
| 20–24 Jul | S4 | Implementación de reintentos automáticos y dead-letter queues (DLQ) | Equipo Backend | Política de reintentos y DLQ configuradas |
| **31 Jul** | — | ✅ **HITO H2: Infraestructura base operativa** | DevOps/SRE | Ambientes dev/staging funcionales |

---

## Mes 3 — Agosto 2026: Servicios de Negocio y Validación Multi-País

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 03–07 Ago | S1 | Desarrollo del microservicio de validación multi-país (impuestos, cancelaciones, requisitos legales) | Equipo Backend | Servicio de validación multi-país en dev |
| 03–07 Ago | S1 | Desarrollo del microservicio de seguimiento de estado de reserva (histórico y buzón) | Equipo Backend | Servicio de tracking en dev |
| 10–14 Ago | S2 | Desarrollo de la capa de API para canales B2C (viajeros directos) | Equipo Backend | API B2C en staging |
| 10–14 Ago | S2 | Desarrollo de la capa de API para canales B2B (agencias y operadores turísticos) | Equipo Backend | API B2B en staging |
| **14 Ago** | S2 | 🟡 **BETA RELEASE 1 — Core Engine** | Equipo Backend, QA | Motor asíncrono con flujo básico end-to-end funcional en dev. Incluye: ingesta, orquestador, inventario, notificaciones, APIs B2C/B2B. |
| 17–21 Ago | S3 | Implementación de cifrado en tránsito y en reposo (KMS) | Esp. Seguridad | Cifrado aplicado en todos los servicios |
| 17–21 Ago | S3 | Configuración de reglas WAF para APIs públicas | Esp. Seguridad | Reglas WAF activas en staging |
| 24–28 Ago | S4 | Pruebas unitarias y de integración de todos los microservicios | QA, Equipo Backend | Cobertura de pruebas >80% |
| 24–28 Ago | S4 | Configuración de observabilidad: dashboards CloudWatch, trazas X-Ray, alertas de SLA | DevOps/SRE | Dashboards y alertas operativas |

---

## Mes 4 — Septiembre 2026: Integración Interna y Estabilización en Staging

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 01–04 Sep | S1 | Integración end-to-end del flujo completo de reserva asíncrona en staging | Equipo Backend, QA | Flujo completo funcional en staging |
| 01–04 Sep | S1 | Desarrollo del adaptador de compatibilidad con el monolito actual | Equipo Backend | Adaptador monolito ↔ motor asíncrono |
| 07–11 Sep | S2 | Pruebas de integración del flujo completo: reserva → validación → confirmación → notificación | QA | Reporte de pruebas de integración |
| 07–11 Sep | S2 | Pruebas de resiliencia: simulación de fallos, caídas de servicios, timeouts | QA, DevOps/SRE | Reporte de resiliencia |
| 14–18 Sep | S3 | Corrección de defectos y optimización de rendimiento | Equipo Backend | Defectos críticos resueltos |
| 14–18 Sep | S3 | Documentación técnica de APIs, flujos de eventos y runbooks operacionales | Arquitecto, Backend | Documentación técnica completa |
| 21–25 Sep | S4 | Validación del flujo completo con usuarios internos (operaciones y ventas) | PM/PO, QA | Validación de usuarios internos |
| 21–25 Sep | S4 | Revisión de seguridad y auditoría de configuración AWS | Esp. Seguridad | Reporte de auditoría de seguridad |
| **25 Sep** | S4 | 🟡 **BETA RELEASE 2 — Full Feature** | Todo el equipo | Motor completo en staging: todos los microservicios, seguridad, observabilidad, adaptador de coexistencia. Validado por usuarios internos. |
| **25 Sep** | S4 | ✅ **HITO H3: Motor asíncrono funcional en staging** | Arquitecto, PM/PO | Gate de paso a integración externa |

---

## Mes 5 — Octubre 2026: Integración con Canales y PMS Externos

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 28 Sep–02 Oct | S1 | Desarrollo de adaptadores para los 3 PMS externos prioritarios | Ing. Integraciones, Backend | Adaptadores PMS en staging |
| 28 Sep–02 Oct | S1 | Configuración de sincronización bidireccional de inventario con PMS externos | Ing. Integraciones | Sincronización de inventario validada |
| 05–09 Oct | S2 | Integración con proveedores de pago a través del flujo asíncrono | Ing. Integraciones | Flujo de pago integrado |
| 05–09 Oct | S2 | Integración del canal B2C con el nuevo motor | Equipo Backend | Canal B2C conectado |
| 12–16 Oct | S3 | Integración del canal B2B (APIs para agencias y operadores) | Equipo Backend | Canal B2B conectado |
| 12–16 Oct | S3 | Pruebas de integración con PMS reales en staging | QA, Ing. Integraciones | Reporte de pruebas PMS |
| 19–23 Oct | S4 | Pruebas end-to-end con datos reales (reservas de prueba con hoteles piloto) | QA, PM/PO | Reporte de pruebas con hoteles piloto |
| 19–23 Oct | S4 | Validación de experiencia del cliente: acuse de recibo, tiempos de confirmación, notificaciones | PM/PO, QA | Validación UX |
| **23 Oct** | S4 | 🟠 **RELEASE CANDIDATE 1 (RC1)** | Todo el equipo | Motor integrado con PMS externos, canales B2C/B2B y proveedores de pago. Validado con hoteles piloto y datos reales. Listo para pruebas de carga. |
| **23 Oct** | S4 | ✅ **HITO H4: Integraciones validadas** | Arquitecto, PM/PO | Gate de paso a pruebas de carga |

---

## Mes 6 — Noviembre 2026: Pruebas de Carga y Estabilización

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 02–06 Nov | S1 | Diseño de escenarios de pruebas de carga: simulación de Semana Santa y Navidad (2x pico histórico) | QA, Arquitecto | Plan de pruebas de carga |
| 02–06 Nov | S1 | Ejecución de pruebas de carga progresivas: 1x, 1.5x, 2x del pico histórico | QA, DevOps/SRE | Resultados de pruebas de carga |
| 09–13 Nov | S2 | Ajuste de autoescalado: EKS, Lambda concurrency, throughput SQS/DynamoDB | DevOps/SRE | Políticas de autoescalado optimizadas |
| 09–13 Nov | S2 | Pruebas de estrés y punto de quiebre: identificar límites del sistema | QA, DevOps/SRE | Reporte de límites y recomendaciones |
| 16–20 Nov | S3 | Pruebas de recuperación ante desastres: failover, recuperación | DevOps/SRE | Reporte de DR y tiempos de recuperación |
| 16–20 Nov | S3 | Pruebas de seguridad: penetration testing, cifrado, permisos IAM | Esp. Seguridad | Reporte de seguridad final |
| **20 Nov** | S3 | 🟠 **RELEASE CANDIDATE 2 (RC2)** | Todo el equipo | Versión estabilizada post-pruebas de carga y seguridad. Todos los defectos críticos y de rendimiento corregidos. |
| 23–27 Nov | S4 | Corrección de defectos residuales de rendimiento y seguridad | Equipo Backend, DevOps | Defectos resueltos |
| 27 Nov | S4 | Aprobación para go-live: revisión final con stakeholders técnicos y de negocio | Arquitecto, PM/PO | Acta de aprobación go-live |
| **27 Nov** | S4 | ✅ **HITO H5: Pruebas de carga superadas, go-live aprobado** | Arquitecto, PM/PO | Gate de paso a despliegue |

---

## Mes 7 — Diciembre 2026: Despliegue Progresivo y Go-Live

| Fecha | Semana | Actividad | Responsable | Entregable |
|---|---|---|---|---|
| 01–05 Dic | S1 | Despliegue en producción — Colombia (piloto): canary deployment 10% del tráfico | DevOps/SRE, Backend | Motor en producción (Colombia, 10%) |
| 01–05 Dic | S1 | Monitoreo intensivo 24/7: dashboards, alertas, war room operativo | DevOps/SRE, Operaciones | Reporte de monitoreo día 1-5 |
| 07–11 Dic | S2 | Incremento progresivo de tráfico en Colombia: 10% → 50% → 100% | DevOps/SRE | Colombia al 100% |
| 07–11 Dic | S2 | Despliegue — Segundo grupo: Perú, Ecuador | DevOps/SRE, Backend | Motor en producción (3 países) |
| 14–18 Dic | S3 | Despliegue — Tercer grupo: México, Chile, Argentina | DevOps/SRE, Backend | Motor en producción (6 países) |
| 14–18 Dic | S3 | Capacitación al equipo de operaciones y soporte | PM/PO, Arquitecto | Equipo capacitado |
| 21–24 Dic | S4 | Capacitación a agencias B2B y operadores turísticos | PM/PO, Ing. Integraciones | Partners capacitados |
| 21–24 Dic | S4 | Cierre del proyecto: lecciones aprendidas, documentación final, transición a operaciones | PM/PO, Arquitecto | Acta de cierre |
| **26 Dic** | — | 🟢 **FINAL RELEASE — GO-LIVE COMPLETADO** | Todo el equipo | Motor asíncrono en producción en los 6 países, monitoreo activo, equipo y partners capacitados. |
| **26 Dic** | — | ✅ **HITO H6: Go-live completado** | PM/PO, Arquitecto | Proyecto cerrado, transición a operaciones |

---

## Resumen del Ciclo de Releases

```
  Jun          Jul          Ago          Sep          Oct          Nov          Dic
   │            │            │            │            │            │            │
   🚀           │            🟡           🟡           🟠           🟠           🟢
   Kickoff      │            Beta 1       Beta 2       RC1          RC2          Final
   01-Jun       │            14-Ago       25-Sep       23-Oct       20-Nov       26-Dic
   │            │            │            │            │            │            │
   ├── Diseño ──┤            │            │            │            │            │
   │            ├─── Desarrollo ──────────┤            │            │            │
   │            │            │            ├─ Integr. ──┤            │            │
   │            │            │            │            ├── Pruebas ─┤            │
   │            │            │            │            │            ├─ Deploy ───┤
```

| Release | Fecha | Alcance | Ambiente | Criterio de Salida |
|---|---|---|---|---|
| **Beta 1** | 14 Ago 2026 | Motor core: ingesta, orquestador, inventario, notificaciones, APIs B2C/B2B | Dev | Flujo básico end-to-end funcional, pruebas unitarias >80% |
| **Beta 2** | 25 Sep 2026 | Feature-complete: validación multi-país, seguridad, observabilidad, adaptador monolito | Staging | Validación por usuarios internos, auditoría de seguridad aprobada |
| **RC1** | 23 Oct 2026 | Integraciones externas: PMS, pagos, canales B2C/B2B con datos reales | Staging | Pruebas con hoteles piloto exitosas, integraciones validadas |
| **RC2** | 20 Nov 2026 | Estabilización: pruebas de carga 2x pico, DR, penetration testing | Pre-producción | Pruebas de carga superadas, defectos críticos resueltos, go-live aprobado |
| **Final** | 26 Dic 2026 | Go-live: despliegue progresivo en 6 países, capacitación, cierre | Producción | 6 países en producción, monitoreo activo, partners capacitados |
