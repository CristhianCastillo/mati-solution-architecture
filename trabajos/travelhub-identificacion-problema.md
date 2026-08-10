# Problema a Resolver
## Lista de Chequeo incial

### Value Proposition Canvas
#### Generadoes de Alegrias
Implementación de una arquitectura escalable para procesar el 100 % de las solicitudes en picos de demanda Sincronización en tiempo real de tarifas y disponibilidad entre el hotel y la plataforma.
#### Productos y Servicios
Plataforma digital de reservas hoteleras para PYMES en Latinoamérica.
#### Mitigadores de dolores​
Eliminación de la latencia de red mediante infraestructura multi-región Automatización del flujo de datos para evitar la creación manual de 70-80 tickets diarios por inconsistencias​.
#### Alegrías
Confirmación exitosa de reservas en temporadas de alta demanda como Semana Santa y Navidad Tiempos de respuesta menores a 400ms para usuarios en toda la región.
#### Trabajos
Los viajeros buscan hoteles y realizan pagos de forma directa Los hoteles cargan sus habitaciones y tarifas en la plataforma.
#### Frustraciones​
Rechazo masivo de peticiones (35-40 %) en temporadas críticas. Pérdida de 3 millones de dólares anuales por incapacidad técnica de ofrecer nuevos servicios corporativos. Inconsistencia de datos donde el hotel no reconoce reservas realizadas por el cliente.

### Cuestionario
#### Qué genera la necesidad de una solución?
La pérdida económica directa de aproximadamente un millón de dólares ($1,000,000 USD) durante periodos críticos como Semana Santa y Navidad, debido a la incapacidad técnica de procesar reservas.  Además, la falta de escalabilidad ha provocado la pérdida de contratos corporativos de hasta 3 millones de dólares anuales. Adicional a la Inconsistencia de datos donde el hotel no reconoce reservas realizadas por el cliente.

#### Cuál es la pregunta o problema clave a ser resuelto?
¿Cómo puede TravelHub modernizar su arquitectura monolítica legada y su infraestructura regionalizada para eliminar los cuellos de botella técnico que causan el rechazo masivo de peticiones impidiendo procesar el 40% de la demanda y sincronizar datos en tiempo real para las temporadas altas?

#### Esta organización u otra similar, han resuelto previamente este problema? Cómo?
En la actualidad se puede identificar que la competencia (ej. Booking) ya ofrece los multiservicios, reportes y paquetes en tiempo real que TravelHub no puede proveer rápidamente, provocando esto que los clientes sean captados por la competencia.​

Internamente, se reconoce la necesidad de migrar hacia una arquitectura moderna y flexible (microservicios/nube distribuida) para soportar la expansión.

#### Quién es el stakeholder principal? Existen otros stakeholders involucrados?
La stakeholder principal ​
 - María González (Directora General)​
Otros involucrados clave son ​
 - Laura Fernández (Producto/Tecnología)​
 - Equipo de DevOps (solo 2 personas) ​
 - Equipo de Operaciones  ​
 - Hoteles asociados que sufren por las inconsistencias

#### Redefina el problema sin perder el significado original
Incapacidad estructural de la plataforma tecnológica para escalar operativamente y mantener la integridad de los datos bajo alta demanda transaccional, resultando en una degradación crítica del servicio y pérdida de ingresos.

#### Pregunte por qué y para qué es necesaria una solución a dicho problema
La solución es necesaria para detener la fuga de capital actual (millones de USD)  y para convertir a TravelHub en un competidor tecnológico global capaz de expandirse e innovar, habilitando la visión estratégica de expansión a Brasil y nuevos servicios.

#### Aumente el foco del problema - Conecte el problema con otros elementos
El problema no es solo técnico, Adicional también se conecta con silos organizacionales donde los sistemas (Salesforce, SAP, PMS) no se comunican, generando 70-80 tickets diarios de error y procesos de onboarding de hoteles que tardan hasta dos semanas.

#### Disminuya el foco del problema - Divida el problema
Este problema puede ser dividido en: ​
- Latencia geográfica (AWS us-east-1) ​
- Deuda técnica (Monolito de 2012)​
- Fallas de integración (Datos inconsistentes entre hoteles y plataforma)​
- Rigidez operativa (DevOps manual)​
- Saturación del servidor PostgreSQL centralizado​
- Desfase en la sincronización semiautomática con los PMS de los hoteles.

#### Redireccione el foco - fuerzas externas que causan el problema
Crecimiento acelerado del mercado geográfico a 6 países sin adaptar la infraestructura, presión competitiva de plataformas globales y la estacionalidad extrema del mercado turístico latinoamericano.

#### Vuelta 180 grados - El problema puede ser algo totalmente opuesto?
¿El problema es el exceso de demanda o la falta de una estrategia de "despacho" que priorice las peticiones más rentables cuando el sistema está saturado?

#### Liste la información relevante para entender el problema en el orden en el que ha ocurrido
Considerando la creación, los cambios y evoluciones que se han presentado en los años se puede identificar lo siguiente:​
- 2012: Fundación con 15 hoteles y arquitectura monolítica.​
- 2015: Implementación de SAP legacy manual.​
- 2019-2020: Adopción de Salesforce sin integración con el core.​
- 2025: Alcance de 1,200 hoteles y detección de 40 % de rechazos en temporadas altas

#### Cuál es la distancia temporal entre eventos clave?
13 años entre la creación del monolito (2012) y la crisis de escalabilidad actual (2025)

#### Determine los momentos en los que la información se conoció
- Registro inicial de hoteles en 2012. ​
- Creación del monolito Java en 2013. 
- Adopción de SAP en 2015. ​
- Apertura en 6 países. ​
- Implementación de Jira en 2016. ​
- Detección de latencia en Argentina y Chile. ​
- Falla de sincronización con Google Analytics en 2018. ​
- Adopción de Salesforce en 2019. ​
- Identificación de inconsistencia de teléfonos entre sistemas. ​
- Reporte de 2-3 % de errores contables en SAP. ​
- Tiempo de 4 días para ajustes manuales en SAP. ​
- Demora de 5-7 días en cálculo de comisiones. ​
- Disputas de vendedores por comisiones manuales. ​
- Onboarding de empleados de 10 días. ​
- Pérdida de comisiones por cambios de hotel manuales. ​
- Identificación de riesgo legal por archivos en Dropbox/Google Drive. ​
- Reporte de latencia de 400ms en Buenos Aires. ​
- Constatación de falta de CI/CD y despliegues manuales de 45 min. ​
- Recuperación de desastres de 6 horas. ​
- Solicitud rechazada a cadena de 50 propiedades. ​
- Fuga de cliente hacia Booking.com. ​
- Cuantificación de pérdida de 3 millones USD anuales. ​
- Identificación de rechazo de 40 % de peticiones en Navidad. ​
- Reporte de 80 tickets diarios por inconsistencias. ​
- Roadmap bloqueado: solo 5 cambios de 12 posibles al año

#### Defina una línea de Tiempo
- 2012 a 2014: Nacimiento y aquitectura base
- 2015 a 2018: Crecimiento regional y adicion de sistemas satelites desconectados (SAP, Jira)
- 2019 a 2024: Escalamiento masivo la deuda tecnica del monolito comienza a causar perdidas visibles
- 2025: Crisis de saturacion (40% rechazos) y desalineacion estrategica total.

### Documentación formal del problema
#### Solicitante
María González - <mgonzalez@travelhub.com>, Directora General de TravelHub 
#### Unidad Organizacional
Dirección General / Operaciones y Tecnología
#### Situacion actual
La plataforma tecnológica es un monolito Java de 2012 que opera en una sola región de AWS. Esto provoca que, durante los picos de demanda estacionales, el sistema no pueda escalar, rechazando entre el 35 % y el 40 % de las solicitudes de reserva. Operativamente, existen silos de información donde Salesforce, SAP y el Core no se comunican, generando 80 tickets diarios de soporte y procesos manuales que toman hasta 7 días.
#### Situacion Objetivo
Lograr una plataforma elástica y distribuida que procese el 100 % de las peticiones en cualquier temporada. Integración total de los sistemas internos (CRM, ERP, RRHH) para eliminar procesos manuales y reducir los tickets de soporte en un 80 %.
#### Objetivos propuestos
Resolver el cuello de botella de escalabilidad del monolito para recuperar el millón de dólares perdido en temporadas altas, de forma que implemente la  Modernización de la arquitectura core, implementación de pipelines CI/CD modernos y despliegue multi-región para reducir la latencia regional.
#### Criterios de aceptacion exitosa de la solución
- Tasa de rechazo de peticiones inferior al 5 % durante la próxima temporada alta (Semana Santa/Navidad).​
- Reducción del 80 % en tickets diarios por inconsistencia de datos de disponibilidad.​
- Latencia de red máxima de 100ms para usuarios en Argentina, Chile y Perú.​
- Automatización del 100 % del cálculo de comisiones y reconciliación con SAP