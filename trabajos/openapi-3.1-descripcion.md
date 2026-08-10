# OpenAPI 3.1 - Descripción del Estándar

## ¿Qué es OpenAPI?

OpenAPI Specification (OAS) es un estándar independiente del lenguaje de programación para describir APIs HTTP de manera estructurada y legible tanto para humanos como para máquinas. Permite documentar, diseñar y consumir APIs sin necesidad de acceder al código fuente o inspeccionar el tráfico de red.

**Versión actual**: OpenAPI 3.1.1 (última actualización estable)  
**Organización**: OpenAPI Initiative (parte de Linux Foundation)  
**Formato**: YAML o JSON

---

## Principales Mejoras en OpenAPI 3.1

### 1. Compatibilidad Total con JSON Schema 2020-12
OpenAPI 3.0 estaba basado en JSON Schema Draft 05. La versión 3.1 se alinea completamente con **JSON Schema Draft 2020-12**, lo que significa:

- Cualquier documento JSON Schema válido es ahora un esquema OpenAPI válido
- Eliminación de inconsistencias entre OpenAPI y JSON Schema
- Soporte para vocabularios extendidos de JSON Schema
- Mejor interoperabilidad con herramientas de validación estándar

**Impacto**: Reutilización directa de esquemas JSON existentes sin modificaciones.

### 2. Soporte para Webhooks
OpenAPI 3.1 introduce soporte nativo para **webhooks** (callbacks asíncronos), permitiendo documentar APIs que envían notificaciones a URLs del cliente.

```yaml
webhooks:
  bookingConfirmed:
    post:
      summary: Notificación cuando una reserva es confirmada
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BookingEvent'
      responses:
        '200':
          description: Webhook recibido correctamente
```

**Uso en TravelHub**: Documentar notificaciones a hoteles cuando se crea/cancela una reserva.

### 3. Keyword `examples` (Plural)
Ahora se soporta el keyword `examples` (array de ejemplos) dentro de los esquemas, además del singular `example`.

```yaml
schema:
  type: object
  properties:
    status:
      type: string
      examples:
        - "confirmed"
        - "pending"
        - "cancelled"
```

**Ventaja**: Múltiples ejemplos para diferentes escenarios (éxito, error, casos edge).

### 4. Mejoras en el Objeto `info`
El objeto `info` ahora soporta campos adicionales:

- `summary`: Resumen breve de la API (complementa `description`)
- `license.identifier`: Identificador SPDX para licencias estándar

```yaml
info:
  title: TravelHub API
  version: 2.1.0
  summary: API para gestión de reservas hoteleras en Latinoamérica
  description: |
    API completa para búsqueda, reserva y gestión de hoteles.
  license:
    name: Apache 2.0
    identifier: Apache-2.0
```

### 5. Null como Tipo Válido
En OpenAPI 3.0, representar valores nulos requería `nullable: true`. En 3.1, se usa directamente el tipo `null` de JSON Schema:

**OpenAPI 3.0**:
```yaml
type: string
nullable: true
```

**OpenAPI 3.1**:
```yaml
type: [string, "null"]
# o
type: string
type: "null"
```

### 6. Exclusión Mutua con `oneOf`, `anyOf`, `allOf`
Soporte completo para composición de esquemas usando JSON Schema:

```yaml
oneOf:
  - $ref: '#/components/schemas/CreditCardPayment'
  - $ref: '#/components/schemas/BankTransferPayment'
  - $ref: '#/components/schemas/CashPayment'
discriminator:
  propertyName: paymentType
```

**Uso en TravelHub**: Modelar diferentes métodos de pago por país.

---

## Estructura de un Documento OpenAPI 3.1

### Componentes Principales

```yaml
openapi: 3.1.0
info:
  title: TravelHub API
  version: 2.1.0
  description: API de reservas hoteleras

servers:
  - url: https://api.travelhub.com/v2
    description: Producción
  - url: https://api-staging.travelhub.com/v2
    description: Staging

paths:
  /hotels:
    get:
      summary: Buscar hoteles
      operationId: searchHotels
      parameters:
        - name: city
          in: query
          required: true
          schema:
            type: string
      responses:
        '200':
          description: Lista de hoteles
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/HotelList'

components:
  schemas:
    HotelList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/Hotel'
    
    Hotel:
      type: object
      required:
        - id
        - name
      properties:
        id:
          type: string
          examples: ["hotel_123"]
        name:
          type: string
        rating:
          type: number
          minimum: 0
          maximum: 5

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

---

## Elementos Clave del Estándar

### 1. **Info Object**
Metadatos de la API: título, versión, descripción, contacto, licencia.

### 2. **Servers Object**
URLs base de los servidores (producción, staging, desarrollo).

### 3. **Paths Object**
Endpoints disponibles y operaciones HTTP (GET, POST, PUT, DELETE, PATCH).

### 4. **Components Object**
Elementos reutilizables:
- `schemas`: Modelos de datos
- `responses`: Respuestas comunes
- `parameters`: Parámetros reutilizables
- `securitySchemes`: Esquemas de autenticación
- `examples`: Ejemplos reutilizables
- `requestBodies`: Cuerpos de petición
- `headers`: Encabezados HTTP
- `callbacks`: Callbacks asíncronos

### 5. **Security Object**
Mecanismos de autenticación y autorización:
- API Keys
- HTTP Basic/Bearer
- OAuth 2.0
- OpenID Connect

### 6. **Tags Object**
Agrupación lógica de operaciones para mejor organización.

---

## Beneficios de OpenAPI 3.1 para TravelHub

### 1. **Generación Automática de Código**
Herramientas como OpenAPI Generator pueden crear:
- SDKs para clientes (Java, Python, JavaScript, PHP)
- Stubs de servidor (Spring Boot, Express, FastAPI)
- Modelos de datos

**Impacto**: Reducción del 60-70% del tiempo de integración para partners.

### 2. **Documentación Interactiva**
Herramientas como Swagger UI, Redoc, Stoplight generan documentación navegable con:
- Pruebas en vivo ("Try it out")
- Ejemplos de peticiones/respuestas
- Validación automática

**Impacto**: Onboarding de partners de 2 semanas a 2-3 días.

### 3. **Validación Automática**
Validación de peticiones/respuestas contra el contrato:
- En desarrollo: detección temprana de errores
- En producción: validación de entrada (seguridad)
- En testing: contract testing automatizado

**Impacto**: Reducción del 40% de bugs relacionados con contratos de API.

### 4. **Versionamiento y Evolución**
Documentación clara de cambios entre versiones:
- Deprecación de endpoints
- Nuevos campos opcionales
- Breaking changes

**Impacto**: Gestión ordenada de múltiples versiones de API (v1, v2, v3).

### 5. **Gobierno de APIs**
Centralización de estándares:
- Naming conventions
- Estructuras de error consistentes
- Patrones de paginación
- Rate limiting

**Impacto**: Consistencia entre 15+ microservicios de TravelHub.

---

## Herramientas del Ecosistema OpenAPI

| Herramienta | Propósito | Uso en TravelHub |
|-------------|-----------|------------------|
| **Swagger UI** | Documentación interactiva | Portal de desarrolladores público |
| **Redoc** | Documentación estática elegante | Documentación interna |
| **OpenAPI Generator** | Generación de código | SDKs para partners (Java, Python, JS) |
| **Spectral** | Linting y validación | CI/CD para validar specs |
| **Stoplight Studio** | Editor visual de OpenAPI | Diseño de APIs por equipo de Producto |
| **Postman** | Testing de APIs | QA y testing manual |
| **Prism** | Mock server | Desarrollo frontend antes de backend |
| **Swagger Codegen** | Generación de código (legacy) | Migración de código existente |

---

## Ejemplo Completo: Endpoint de Reservas

```yaml
openapi: 3.1.0
info:
  title: TravelHub Booking API
  version: 2.1.0
  summary: API para gestión de reservas hoteleras
  contact:
    name: TravelHub API Support
    email: api-support@travelhub.com
    url: https://developers.travelhub.com

servers:
  - url: https://api.travelhub.com/v2
    description: Producción
  - url: https://api-staging.travelhub.com/v2
    description: Staging

paths:
  /bookings:
    post:
      summary: Crear una nueva reserva
      operationId: createBooking
      tags:
        - Bookings
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateBookingRequest'
            examples:
              basicBooking:
                summary: Reserva básica
                value:
                  hotelId: "hotel_123"
                  checkIn: "2026-03-15"
                  checkOut: "2026-03-18"
                  guests: 2
                  roomType: "double"
              corporateBooking:
                summary: Reserva corporativa
                value:
                  hotelId: "hotel_456"
                  checkIn: "2026-04-01"
                  checkOut: "2026-04-05"
                  guests: 1
                  roomType: "suite"
                  corporateCode: "CORP2026"
      responses:
        '201':
          description: Reserva creada exitosamente
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BookingResponse'
              examples:
                success:
                  value:
                    meta:
                      version: "2.1.0"
                      timestamp: "2026-02-25T22:00:00Z"
                      requestId: "req_abc123"
                    data:
                      bookingId: "bk_789xyz"
                      status: "confirmed"
                      confirmationCode: "TH-ABC123"
                      totalAmount: 450.00
                      currency: "USD"
        '400':
          description: Petición inválida
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                invalidDates:
                  value:
                    type: "https://api.travelhub.com/errors/invalid-dates"
                    title: "Invalid Date Range"
                    status: 400
                    detail: "Check-out date must be after check-in date"
                    instance: "/api/v2/bookings"
        '402':
          description: Error de pago
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
        '409':
          description: Habitación no disponible
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

  /bookings/{bookingId}:
    get:
      summary: Obtener detalles de una reserva
      operationId: getBooking
      tags:
        - Bookings
      security:
        - bearerAuth: []
      parameters:
        - name: bookingId
          in: path
          required: true
          schema:
            type: string
            pattern: '^bk_[a-zA-Z0-9]+$'
          examples:
            - "bk_789xyz"
      responses:
        '200':
          description: Detalles de la reserva
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/BookingResponse'
        '404':
          description: Reserva no encontrada
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'

webhooks:
  bookingStatusChanged:
    post:
      summary: Notificación de cambio de estado de reserva
      description: |
        TravelHub enviará este webhook cuando el estado de una reserva cambie
        (confirmada, cancelada, modificada).
      requestBody:
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/BookingWebhookEvent'
      responses:
        '200':
          description: Webhook procesado correctamente

components:
  schemas:
    CreateBookingRequest:
      type: object
      required:
        - hotelId
        - checkIn
        - checkOut
        - guests
        - roomType
      properties:
        hotelId:
          type: string
          description: ID del hotel
          examples: ["hotel_123"]
        checkIn:
          type: string
          format: date
          description: Fecha de check-in (ISO 8601)
          examples: ["2026-03-15"]
        checkOut:
          type: string
          format: date
          description: Fecha de check-out (ISO 8601)
          examples: ["2026-03-18"]
        guests:
          type: integer
          minimum: 1
          maximum: 10
          description: Número de huéspedes
          examples: [2]
        roomType:
          type: string
          enum: [single, double, suite, family]
          description: Tipo de habitación
        corporateCode:
          type: [string, "null"]
          description: Código corporativo (opcional)
          examples: ["CORP2026", null]

    BookingResponse:
      type: object
      required:
        - meta
        - data
      properties:
        meta:
          $ref: '#/components/schemas/ResponseMeta'
        data:
          type: object
          required:
            - bookingId
            - status
            - confirmationCode
          properties:
            bookingId:
              type: string
              examples: ["bk_789xyz"]
            status:
              type: string
              enum: [pending, confirmed, cancelled]
            confirmationCode:
              type: string
              examples: ["TH-ABC123"]
            totalAmount:
              type: number
              format: double
              examples: [450.00]
            currency:
              type: string
              pattern: '^[A-Z]{3}$'
              examples: ["USD", "COP", "BRL"]

    ResponseMeta:
      type: object
      required:
        - version
        - timestamp
        - requestId
      properties:
        version:
          type: string
          examples: ["2.1.0"]
        timestamp:
          type: string
          format: date-time
          examples: ["2026-02-25T22:00:00Z"]
        requestId:
          type: string
          examples: ["req_abc123"]

    ErrorResponse:
      type: object
      required:
        - type
        - title
        - status
      properties:
        type:
          type: string
          format: uri
          description: URI que identifica el tipo de error
          examples: ["https://api.travelhub.com/errors/payment-failed"]
        title:
          type: string
          description: Título legible del error
          examples: ["Payment Processing Failed"]
        status:
          type: integer
          description: Código de estado HTTP
          examples: [400, 402, 404, 409, 500]
        detail:
          type: string
          description: Explicación detallada del error
          examples: ["Insufficient funds in the provided payment method"]
        instance:
          type: string
          format: uri
          description: URI de la petición que causó el error
          examples: ["/api/v2/bookings"]
        traceId:
          type: string
          description: ID de traza para debugging
          examples: ["trace_abc123"]
        errors:
          type: array
          description: Lista de errores específicos de campos
          items:
            type: object
            properties:
              field:
                type: string
                examples: ["checkIn"]
              code:
                type: string
                examples: ["INVALID_DATE"]
              message:
                type: string
                examples: ["Check-in date must be in the future"]

    BookingWebhookEvent:
      type: object
      required:
        - eventId
        - eventType
        - timestamp
        - data
      properties:
        eventId:
          type: string
          examples: ["evt_abc123"]
        eventType:
          type: string
          enum: [booking.created, booking.confirmed, booking.cancelled, booking.modified]
        timestamp:
          type: string
          format: date-time
        data:
          type: object
          properties:
            bookingId:
              type: string
            previousStatus:
              type: string
            newStatus:
              type: string

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
      description: |
        Token JWT obtenido del endpoint /auth/token.
        Incluir en el header: Authorization: Bearer <token>

  parameters:
    PageParam:
      name: page
      in: query
      schema:
        type: integer
        minimum: 1
        default: 1
      description: Número de página para paginación

    PageSizeParam:
      name: pageSize
      in: query
      schema:
        type: integer
        minimum: 1
        maximum: 100
        default: 20
      description: Cantidad de resultados por página

security:
  - bearerAuth: []

tags:
  - name: Bookings
    description: Operaciones relacionadas con reservas
  - name: Hotels
    description: Operaciones relacionadas con hoteles
  - name: Payments
    description: Operaciones relacionadas con pagos
```

---

## Mejores Prácticas para TravelHub

### 1. **Versionamiento en URL**
```
https://api.travelhub.com/v2/bookings
```
No en headers (más explícito para partners).

### 2. **Usar `operationId` Único**
Facilita generación de código con nombres de métodos consistentes.

### 3. **Documentar Todos los Códigos de Estado**
Incluir 200, 201, 400, 401, 403, 404, 409, 422, 429, 500, 503.

### 4. **Ejemplos Realistas**
Usar datos de ejemplo que reflejen casos reales de uso.

### 5. **Estructuras de Error Consistentes**
Seguir RFC 7807 (Problem Details) para todos los errores.

### 6. **Validación en CI/CD**
Usar Spectral para validar specs antes de merge:
```bash
spectral lint openapi.yaml
```

### 7. **Separar Specs por Dominio**
- `booking-api.yaml`
- `inventory-api.yaml`
- `payment-api.yaml`

Luego combinar con `$ref` externas.

---

## Migración de OpenAPI 3.0 a 3.1

### Cambios Principales

| OpenAPI 3.0 | OpenAPI 3.1 |
|-------------|-------------|
| `nullable: true` | `type: ["string", "null"]` |
| `example: "value"` | `examples: ["value1", "value2"]` |
| No soporta webhooks | `webhooks:` nativo |
| JSON Schema Draft 05 | JSON Schema 2020-12 |
| `exclusiveMinimum: true` | `exclusiveMinimum: 5` (valor directo) |

### Herramientas de Migración
- **openapi-format**: Convierte automáticamente de 3.0 a 3.1
- **Spectral**: Detecta incompatibilidades

---

## Conclusión

OpenAPI 3.1 es el estándar moderno para documentar APIs HTTP. Para TravelHub, adoptar este estándar significa:

✅ **Reducción de tiempo de integración** para partners (70%)  
✅ **Documentación siempre actualizada** (generada del código)  
✅ **Validación automática** de contratos (menos bugs)  
✅ **Generación de SDKs** en múltiples lenguajes  
✅ **Gobierno centralizado** de APIs entre microservicios  
✅ **Mejor experiencia de desarrollador** (DX)

**Recomendación**: Implementar OpenAPI 3.1 como parte de la Fase 1 de la transformación arquitectónica, comenzando con las APIs más críticas (Bookings, Inventory, Payments).

---

## Referencias

- [OpenAPI Specification 3.1.1](https://spec.openapis.org/oas/v3.1.1)
- [JSON Schema 2020-12](https://json-schema.org/draft/2020-12/release-notes.html)
- [OpenAPI Initiative](https://www.openapis.org/)
- [Swagger Tools](https://swagger.io/tools/)
