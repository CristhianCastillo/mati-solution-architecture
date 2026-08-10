# TravelHub: Contexto Actual

### Sobre la Empresa
TravelHub fue fundada en 2012 como una startup que buscaba revolucionar las reservas hoteleras en
Latinoamérica conectando hoteles pequeños y medianos (PYMES) con clientes directamente, sin
intermediarios tradicionales. En 2012 comenzó con 15 hoteles en Colombia. Para 2025, opera en 6
países (Colombia, Perú, Ecuador, México, Chile y Argentina) con 1,200 propiedades hoteleras
asociadas, 350 agencias de viaje partners, 180 operadores turísticos, y aproximadamente 450,000
viajeros activos. Los ingresos anuales proyectados son aproximadamente 51.6 millones USD, con
márgenes operacionales del 28-32%. Aunque estos números parecen sólidos, el crecimiento orgánico
no es suficiente en un mercado tan competitivo. TravelHub es una empresa mediana regional con
limitaciones estructurales que la impiden acceder a nuevos mercados, servicios, y oportunidades de
crecimiento que requiere el mercado de viajes para ser competitiva.

### Estructura Organizacional Actual
TravelHub tiene aproximadamente 120 empleados distribuidos en la región en múltiples áreas
funcionales: Ventas y Crecimiento (25 personas), Operaciones y Servicio al Cliente (35 personas),
Producto (8 personas), Tecnología (12 personas), Finanzas y Contabilidad (15 personas), y Recursos
Humanos y Cumplimiento (10 personas). Esta estructura funcional tiene un problema crítico: cada
departamento opera en silos con poca visibilidad cruzada. Los sistemas internos no comunican entre
sí, creando fricciones operacionales y financieras que crecen conforme la empresa escala.

### Procesos de Negocio Actuales
El flujo de negocio comienza cuando un hotel quiere asociarse. Un vendedor lo registra, el hotel carga
habitaciones y tarifas, y TravelHub sincroniza con su plataforma pública. Los viajeros buscan hoteles,
realizan la reserva mediante la plataforma y realizan el pago. En este punto TravelHub toma una
comisión (típicamente 15-20%) y transfiere el excedente al hotel. Es importante considerar que cada
país tiene regulaciones diferentes de impuestos, políticas de cancelación, y requisitos legales que se
deben cumplir. Algunos hoteles usan PMS propios y otros utilizan PMSs de mercado. Las integraciones
con terceros ocurren mediante procesos semiautomáticos. Si hay un problema en un país en una zona
horaria diferente, el tiempo de solución para el cliente se ve mucho más afectado.

## Situación actual tecnológica
### La Arquitectura Monolítica Original
TravelHub fue construida en 2012-2013 como una aplicación monolítica usando Java con una base de
datos PostgreSQL centralizada y un frontend desarrollado en JSP. La aplicación ha sido actualizada
constantemente sin mejorar la estructura fundamental. El resultado es código legado difícil de
entender, desfavoreciendo la cohesión semántica.

### Sistemas Auxiliares Desconectados
Además del PMS actual, TravelHub ha agregado sistemas satélite sin integración: Jira (2016-2017) para
tickets de operaciones que no se conecta con el monolito (cambios de reserva requieren creación
manual de tickets); Google Analytics y Redshift (2018) cuyo datos no están sincronizados (1,000
usuarios visitaron pero solo 800 transacciones procesadas); Salesforce (2019-2020) cuyo datos de
hoteles difieren del PMS (un teléfono actualizado en Salesforce no se refleja en monolito).

### Sistema de Contabilidad y Facturación
TravelHub usa SAP legacy (2015) para la contabilidad, la facturación, y la reconciliación con hoteles. El
proceso es manual: cada reserva debe ser contabilizada en SAP manualmente. Aproximadamente 2-
3% se contabilizan incorrectamente requiriendo 3 a 4 días de ajustes manuales cada mes. SAP está
configurado principalmente para Colombia; los otros 5 países requieren un procesamiento manual
que se toman 20 archivos de Excel donde se sincronizan datos, se aplica lógica de impuestos local, y
se generan reportes para autoridades fiscales. Si una tarifa cambia, no fluye automáticamente a SAP
y se descubre semanas después en auditorías.

### Sistema de Comisiones de Ventas
El cálculo de comisiones es completamente manual. Los vendedores tienen esquemas diferentes por
país, tipo de hotel, y segmentos de clientes. Al final de cada mes, RH extrae datos del PMS, verifica
asignaciones, calcula comisiones y aplica los bonos pactados. Este proceso toma de 5 a 7 días. Los
errores son frecuentes: un vendedor solicita una comisión por una reserva que otro vendedor realizó,
cambios de políticas inconsistentes retroactivamente, cálculos de comisiones incorrectos. Es común
ver disputas entre vendedores que requirieron arbitraje del VP de Ventas. La falta de automatización
erosiona la confianza en la organización.

#### Sistema de Recursos Humanos
TravelHub usa BambooHR en la nube para el manejo de la nómina y el manejo de beneficios. Pero la
integración es complicada. Cuando alguien es contratado, RH entra los datos en BambooHR, luego un
administrador crea manualmente la cuenta de la persona en el PMS, el usuario en Salesforce, la
entrada en Active Directory y agrega al usuario en Slack. Este Onboarding toma entre 5 y 10 días. Si
alguien es despedido, el acceso debe revocarse manualmente. BambooHR no se comunica con
Salesforce. Cuando un vendedor es promovido, sus hoteles deben transferirse manualmente en el
monolito o las comisiones se pueden perder.

### Sistema de Gestión de Documentos y Cumplimiento
TravelHub maneja documentación sensible (contratos, acuerdos, políticas GDPR/LGPD) la cual está
distribuida en varias cuentas de Dropbox personales y Google Drive. No hay versionamiento
centralizado, auditoría de acceso, o retención. Cuando se necesita auditar qué documentos fueron
acordados con un cliente, por lo que no hay forma rápida de responder. Esta situación crea un riesgo
legal y de cumplimiento.

### Infraestructura y Operaciones
TravelHub opera toda su infraestructura en AWS en una sola región: us-east-1. Esto fue eficiente en
2015 cuando la mayoría de usuarios estaban en Colombia. Pero hoy con usuarios en Argentina, Chile,
y Perú, la latencia es problemática: un viajero en Buenos Aires espera 300-400ms solo en latencia de
red antes de que el servidor procese. Operacionalmente, el equipo de DevOps es muy pequeño (2
personas). No hay pipeline de CI/CD moderno; los despliegues son manuales tomando entre 30 y 45
minutos. Cuando hay un bug, hay que esperar al próximo deployment window (2x/semana) o hacer
despliegue de emergencia (requiere aprobación del VP ). No hay estrategia de recuperación de
desastres moderna. Si AWS us-east-1 falla, TravelHub está offline. La recuperación toma usualmente
entre 4 y 6 horas manualmente.

## Problemas Actuales de Negocio: La Desalineación Tecnología-Estrategia

### Oportunidades Perdidas y Pérdidas de Ingresos
María González, Directora General, recibe solicitudes de hoteles para servicios que TravelHub no
puede ofrecer. Hace 6 meses, una cadena de 50 propiedades en 3 países, solicitó reportes
consolidados en tiempo real a nivel corporativo para concretar un negocio con la empresa. Sin
embargo, TravelHub tuvo que responder que implementar esa funcionalidad requeriría 8 meses. Este
tiempo era insostenible para el cliente por lo que decidieron irse para Booking. TravelHub perdió
cerca de 3 millones de dólares anuales en comisiones. Otras veces, los operadores turísticos quieren
tener acceso a paquetes dinámicos (hotel+vuelo+actividades). TravelHub solo ofrece hoteles en este
momento. Agregar las otras funcionalidades puede llevar 6 meses dada la arquitectura actual. Por
todo lo anterior, TravelHub sigue solo manejando hoteles, mientras la competencia se mueve a multi-
servicios. Por ejemplo, durante la Semana Santa y Navidad, entre un 35 a 40% de las peticiones son
rechazadas. Algunas estimaciones conservadoras sugieren pérdidas cercanas al millón de dólares en
esos períodos.

### Limitaciones Operacionales e Ineficiencias
El equipo de Operaciones maneja entre 70 y 80 tickets diarios sobre inconsistencias de datos. El
problema más frecuente suele ser que un cliente reserva pero el hotel le responde que la habitación
no está disponible o que la reserva nunca se realizó. Otro problema muy frecuente es el cambio de
tarifa por parte del hotel que se refleja hasta horas después cuando ya el cliente reservó con un valor
menor. Estos tickets podrían reducirse en un 80% con un rediseño del proceso, en su lugar, hoy
requieren una solución manual que toma de 2 a 3 días cada uno en ser solucionado. La falta de
integración entre Salesforce y el monolito hace que Ventas gaste mucho tiempo copiando datos entre
estos dos sistemas. El Onboarding de un nuevo hotel toma entre una y dos semanas. Laura Fernández
tiene un roadmap de 12 meses de cambios solicitados pero solo puede implementar entre 4 y 5 al año
porque Tecnología está sobrecargada manteniendo la plataforma actual. Internamente, el cálculo de
comisiones consume entre 5 y 7 días por mes lo que genera inconformidades en la fuerza de ventas.
El onboarding de empleados toma en promedio 7 días. Finalmente se manejan múltiples hojas excel
para el control de los impuestos lo que usualmente genera muchos errores difíciles de identificar.

## El Futuro: Hacia Dónde Debe Ir TravelHub
### Visión Estratégica y Planes de Crecimiento
El equipo ejecutivo tiene visión clara pero reconoce que la tecnología actual es cuello de botella.
Primero, el crecimiento geográfico: la entrada a Brasil requeriría cumplimiento LGPD, procesamiento
de la moneda real brasileño, así como los métodos de pago locales. La infraestructura actual no
soporta esto. Segundo, la expansión de servicios: ser plataforma de viajes completa (hoteles, vuelos,
tours, transporte, restaurantes) va a requerir integraciones con múltiples proveedores, APIs
versionamiento y orquestación de transacciones complejas. El sistema actual no fue diseñado para
eso. Tercero, innovación en el modelo de negocio: se desea crear el programa de suscripción
TravelHub Plus con descuentos progresivos, puntos de lealtad, acceso temprano a ofertas. Esto
requiere un sistema de gestión de suscripciones, facturación recurrente y la integración con un motor
de búsqueda para descuentos dinámicos.

### Inteligencia Artificial y Automatización
El equipo de Producto ve oportunidades significativas en inteligencia artificial. Para servicio al cliente,
implementar chatbots multi-idioma que resuelvan 80% de consultas sobre cambios de reserva,
cancelaciones y consultas sobre planes actuales. Esta solución requiere de modelos entrenados en
múltiples idiomas y bases de conocimiento integradas. Otro objetivo es poder utilizar la IA para
generar recomendaciones personalizadas, entrenar modelos que sugieran hoteles basados en
búsquedas anteriores, similitud con viajeros de un mismo perfil, o patrones socio-económicos. Estas
propuestas requieren de una infraestructura de datos limpia y una excelente calidad de datos. Para
llevar a cabo la detección de fraude en los pagos, se quiere el uso de modelos que identifiquen
transacciones anómalas en tiempo real. Para operaciones internas, se desea automatizar el cálculo
de comisiones mediante reglas claras, validación automática de documentos de cumplimiento,
reconciliación automática entre el sistema actual y SAP usando machine learning para resolver
discrepancias pequeñas. Todo lo anterior, requiere una infraestructura de datos limpia, pipelines de
ML versionados, y monitoreo constante de los modelos.

### Nueva Línea de Negocio: Analytics como Servicio
TravelHub está evaluando una nueva línea de negocio: vender analytics a sus socios de negocio
(hoteles y operadores turísticos). En lugar de que un hotel vea solo sus reservas en un reporte
nocturno, podría acceder a un tablero de control ejecutivo mostrando ocupación en tiempo real,
comparativa con competidores en la misma ciudad, análisis de poder predictivo (qué fechas
probablemente tengan demanda), recomendaciones sobre qué amenidades agregar basado en
análisis de búsquedas de los viajeros. Esto requeriría una plataforma B2B2B separada que ingiera
datos de TravelHub y datos públicos, aplique los modelos de ML, y expondrá a través de APIs y
dashboards la información. Sería una nueva fuente de ingresos: suscripción por hotel o por análisis
específico. Pero va a requerir un data lake, pipelines de transformación, y gobernanza de datos.

### Posibles Adquisiciones y Consolidación
TravelHub ha comenzado conversaciones de adquisición con dos startups pequeñas pero innovadoras.
Basecamp (40 empleados) es una plataforma de tours y actividades con 8,000 proveedores. NightStay
(20 empleados) ofrece hospedajes alternativos (glamping, zonas de camping, carros casa, cabañas, y
casas vacacionales). Si TravelHub adquiere estas empresas, tendría que integrar 2 nuevas bases de
datos, 2 PMSs nuevos y 2 sistemas de pago en una plataforma coherente. Esto va a requerir una
arquitectura moderna y flexible, no el monolito actual.