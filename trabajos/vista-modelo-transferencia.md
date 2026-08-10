# Vista de Modelo de Transferencia y Formato de Datos - TravelHub

## 1. Estándares de Mensajería

### 1.1 Comunicación Síncrona
| Componente | Estándar | Protocolo | Uso |
|------------|----------|-----------|-----|
| API Gateway → Microservicios | REST/HTTP | HTTPS/TLS 1.3 | Operaciones CRUD, consultas en tiempo real |
| Microservicios internos | gRPC | HTTP/2 | Comunicación de baja latencia entre servicios |
| Cliente Web/Mobile → API | REST/HTTP | HTTPS/TLS 1.3 | Búsquedas, reservas, consultas de usuario |
| Integraciones PMS externos | REST/SOAP | HTTPS | Sincronización de inventario y tarifas |

### 1.2 Comunicación Asíncrona
| Componente | Estándar | Tecnología | Uso |
|------------|----------|------------|-----|
| Event Bus | Event-Driven Architecture | Amazon EventBridge / Apache Kafka | Eventos de dominio (ReservaCreada, PagoConfirmado) |
| Colas de mensajes | Message Queue | Amazon SQS | Procesamiento de tareas pesadas, reintentos |
| Notificaciones | Pub/Sub | Amazon SNS | Alertas, notificaciones multi-canal |
| Streaming de datos | Event Streaming | Apache Kafka / Kinesis | Logs, métricas, analytics en tiempo real |

### 1.3 Patrones de Mensajería
| Patrón | Implementación | Casos de Uso |
|--------|----------------|--------------|
| Request-Response | REST API / gRPC | Búsqueda de hoteles, creación de reservas |
| Fire-and-Forget | SQS / EventBridge | Envío de emails, actualización de analytics |
| Publish-Subscribe | SNS / EventBridge | Notificaciones multi-sistema (CRM, contabilidad, comisiones) |
| Event Sourcing | EventBridge + DynamoDB Streams | Auditoría de transacciones, reconstrucción de estado |
| Saga Pattern | Step Functions + EventBridge | Reservas multi-servicio (hotel+vuelo+tour) |

---

## 2. Estándares de Intercambio de Datos

### 2.1 APIs Externas
| Sistema | Estándar | Versión | Descripción |
|---------|----------|---------|-------------|
| API Pública TravelHub | OpenAPI 3.1 | v2.x | Especificación REST para partners y clientes |
| Integraciones PMS | OTA (OpenTravel Alliance) | 2015A | Estándar hotelero para inventario y reservas |
| Pagos | PCI DSS / ISO 20022 | - | Seguridad en transacciones financieras |
| Contabilidad (SAP) | IDoc / OData | 4.0 | Integración con ERP legacy |
| CRM (Salesforce) | Salesforce REST API | v58.0 | Sincronización de clientes y oportunidades |

### 2.2 Contratos de Datos Internos
| Dominio | Esquema | Versionamiento | Validación |
|---------|---------|----------------|------------|
| Reservas | JSON Schema | Semantic Versioning | JSON Schema Validator |
| Inventario | Avro Schema | Schema Registry (Confluent) | Avro validation |
| Eventos | CloudEvents 1.0 | Event versioning en metadata | CloudEvents SDK |
| Analytics | Parquet / Delta Lake | Schema evolution | Great Expectations |

### 2.3 Gobierno de APIs
- **Versionamiento**: URL-based (`/api/v2/hotels`) + Header-based (`Accept: application/vnd.travelhub.v2+json`)
- **Deprecación**: Mínimo 12 meses de soporte para versiones anteriores
- **Rate Limiting**: Token bucket algorithm (100 req/min usuarios, 1000 req/min partners)
- **Autenticación**: OAuth 2.0 + JWT para APIs públicas, mTLS para servicios internos

---

## 3. Formatos de Datos

### 3.1 Formato por Caso de Uso
| Caso de Uso | Formato Principal | Formato Alternativo | Justificación |
|-------------|-------------------|---------------------|---------------|
| **APIs REST** | JSON | XML (legacy) | Ligero, amplio soporte, legible |
| **Comunicación gRPC** | Protocol Buffers | - | Binario compacto, alto rendimiento |
| **Event Streaming** | Avro | JSON | Schema evolution, compresión eficiente |
| **Data Lake** | Parquet | Delta Lake | Columnar, compresión, analytics optimizado |
| **Configuración** | YAML | JSON | Legible, comentarios, infraestructura como código |
| **Documentos legales** | PDF/A | - | Archivado a largo plazo, cumplimiento |
| **Reportes ejecutivos** | CSV / Excel | PDF | Interoperabilidad con herramientas de negocio |
| **Logs estructurados** | JSON | - | Parseo automático, indexación en ELK/CloudWatch |

### 3.2 Especificaciones de Formato JSON

#### Estructura Estándar de Respuesta API
```json
{
  "meta": {
    "version": "2.1.0",
    "timestamp": "2026-02-25T22:00:00Z",
    "requestId": "req_abc123xyz"
  },
  "data": {
    // Payload principal
  },
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalPages": 15,
    "totalItems": 300
  },
  "links": {
    "self": "/api/v2/hotels?page=1",
    "next": "/api/v2/hotels?page=2",
    "prev": null
  }
}
```

#### Estructura de Error Estándar (RFC 7807 - Problem Details)
```json
{
  "type": "https://api.travelhub.com/errors/payment-failed",
  "title": "Payment Processing Failed",
  "status": 402,
  "detail": "Insufficient funds in the provided payment method",
  "instance": "/api/v2/bookings/bk_789xyz",
  "traceId": "trace_abc123",
  "errors": [
    {
      "field": "paymentMethod.cardNumber",
      "code": "INSUFFICIENT_FUNDS",
      "message": "The card has insufficient funds"
    }
  ]
}
```

### 3.3 Especificaciones de Formato Avro (Eventos)

```json
{
  "type": "record",
  "namespace": "com.travelhub.events",
  "name": "BookingCreated",
  "version": "2.0.0",
  "fields": [
    {"name": "eventId", "type": "string"},
    {"name": "eventType", "type": "string", "default": "booking.created"},
    {"name": "timestamp", "type": "long", "logicalType": "timestamp-millis"},
    {"name": "bookingId", "type": "string"},
    {"name": "hotelId", "type": "string"},
    {"name": "userId", "type": "string"},
    {"name": "checkIn", "type": "string"},
    {"name": "checkOut", "type": "string"},
    {"name": "totalAmount", "type": "double"},
    {"name": "currency", "type": "string"},
    {"name": "country", "type": "string"}
  ]
}
```

### 3.4 Formato Protocol Buffers (gRPC)

```protobuf
syntax = "proto3";

package travelhub.inventory.v1;

message GetAvailabilityRequest {
  string hotel_id = 1;
  string check_in = 2;  // ISO 8601
  string check_out = 3;
  int32 guests = 4;
}

message GetAvailabilityResponse {
  repeated Room available_rooms = 1;
  
  message Room {
    string room_id = 1;
    string room_type = 2;
    double price = 3;
    string currency = 4;
    int32 available_units = 5;
  }
}
```

---

## 4. Transformación y Mapeo de Datos

### 4.1 Capas de Transformación
| Capa | Responsabilidad | Herramienta |
|------|-----------------|-------------|
| **API Gateway** | Validación de entrada, rate limiting | AWS API Gateway / Kong |
| **Service Mesh** | Enrutamiento, retry, circuit breaker | Istio / AWS App Mesh |
| **ETL/ELT** | Transformación batch para analytics | AWS Glue / Apache Spark |
| **Stream Processing** | Transformación en tiempo real | Apache Flink / Kinesis Analytics |
| **Data Mapper** | Conversión entre formatos (JSON↔XML↔Avro) | Apache Camel / Spring Integration |

### 4.2 Mapeo de Sistemas Legacy
| Sistema Origen | Formato Origen | Sistema Destino | Formato Destino | Frecuencia |
|----------------|----------------|-----------------|-----------------|------------|
| Monolito Java | JDBC/SQL | Event Bus | Avro | Tiempo real (CDC) |
| SAP | IDoc | Data Lake | Parquet | Batch nocturno |
| Salesforce | REST/JSON | CRM Service | JSON | Webhook en tiempo real |
| PMS Externos | OTA XML | Inventory Service | JSON | Polling cada 5 min |

---

## 5. Seguridad y Cumplimiento

### 5.1 Encriptación de Datos
| Escenario | Método | Estándar |
|-----------|--------|----------|
| Datos en tránsito | TLS 1.3 | HTTPS, mTLS para servicios internos |
| Datos en reposo | AES-256 | AWS KMS, envelope encryption |
| Datos sensibles (PII) | Field-level encryption | AWS KMS + application-level |
| Tokens de pago | Tokenización | PCI DSS compliant vault |

### 5.2 Cumplimiento por País
| País | Regulación | Impacto en Formato de Datos |
|------|------------|------------------------------|
| Brasil | LGPD | Consentimiento explícito en JSON, data residency |
| Argentina | PDPA | Logs de acceso a datos personales |
| México | LFPDPPP | Aviso de privacidad en metadata |
| UE (viajeros) | GDPR | Right to erasure, data portability (JSON export) |
| Todos | PCI DSS | No almacenar CVV, enmascarar PAN |

---

## 6. Monitoreo y Observabilidad

### 6.1 Formato de Logs Estructurados
```json
{
  "timestamp": "2026-02-25T22:00:00.123Z",
  "level": "INFO",
  "service": "booking-service",
  "traceId": "trace_abc123",
  "spanId": "span_xyz789",
  "userId": "user_456",
  "action": "create_booking",
  "duration_ms": 234,
  "status": "success",
  "metadata": {
    "hotelId": "hotel_123",
    "country": "CO"
  }
}
```

### 6.2 Métricas y Trazas
| Tipo | Formato | Herramienta |
|------|---------|-------------|
| Métricas | Prometheus format | CloudWatch / Prometheus |
| Trazas distribuidas | OpenTelemetry | AWS X-Ray / Jaeger |
| Logs | JSON estructurado | CloudWatch Logs / ELK |

---

## 7. Decisiones Arquitectónicas Clave

### ADR-001: JSON como formato principal para APIs REST
**Decisión**: Usar JSON como formato estándar para todas las APIs REST públicas y la mayoría de internas.

**Razones**:
- Amplio soporte en todos los lenguajes y frameworks
- Legible por humanos, facilita debugging
- Menor curva de aprendizaje para partners
- Ecosistema maduro de herramientas de validación

**Alternativas consideradas**: XML (muy verboso), Protocol Buffers (no legible)

### ADR-002: Avro para event streaming
**Decisión**: Usar Apache Avro para eventos en Kafka/EventBridge.

**Razones**:
- Schema evolution sin romper compatibilidad
- Compresión superior a JSON (30-40% menor tamaño)
- Schema Registry para gobierno centralizado
- Integración nativa con ecosistema Kafka

**Alternativas consideradas**: JSON (sin schema evolution), Protobuf (menos soporte en Kafka)

### ADR-003: gRPC para comunicación interna de alta frecuencia
**Decisión**: Usar gRPC para comunicación entre microservicios críticos (inventario, pricing).

**Razones**:
- 7-10x más rápido que REST en benchmarks internos
- Contratos fuertemente tipados (Protocol Buffers)
- Streaming bidireccional nativo
- Menor latencia en comunicación inter-servicio

**Alternativas consideradas**: REST (más lento), GraphQL (overhead innecesario)

---

## 8. Roadmap de Implementación

| Fase | Componente | Formato/Estándar | Prioridad |
|------|------------|------------------|-----------|
| **Fase 1** | API Gateway + REST APIs | JSON + OpenAPI 3.1 | Alta |
| **Fase 1** | Event Bus básico | JSON (transición a Avro) | Alta |
| **Fase 2** | gRPC entre microservicios core | Protocol Buffers | Media |
| **Fase 2** | Schema Registry | Avro | Media |
| **Fase 3** | Data Lake | Parquet + Delta Lake | Media |
| **Fase 3** | Integración OTA para PMS | OTA XML → JSON | Baja |
| **Fase 4** | Stream processing | Avro + Flink | Baja |
