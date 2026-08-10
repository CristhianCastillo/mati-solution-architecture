# OTA (OpenTravel Alliance) - Descripción del Estándar

## 1. ¿Qué es OpenTravel Alliance?

**OpenTravel Alliance (OTA)** es una organización sin fines de lucro fundada en 1999 que desarrolla y mantiene especificaciones abiertas basadas en XML y JSON para el intercambio electrónico de información de negocio en toda la industria de viajes y hospitalidad.

**Misión**: Crear, expandir y promover la adopción de especificaciones abiertas para el intercambio de datos entre todos los sectores de la industria de viajes.

### Características Principales
- **Estándar abierto**: Especificaciones gratuitas y de libre uso
- **Independiente de plataforma**: Funciona en cualquier sistema operativo o lenguaje
- **Basado en XML/JSON**: Formatos estructurados y ampliamente soportados
- **Orientado a la industria**: Diseñado específicamente para viajes y hospitalidad
- **Respaldado por la industria**: Más de 200 miembros incluyendo hoteles, aerolíneas, OTAs, PMSs, GDS

### Miembros Destacados
- **Hoteles**: Marriott, Hilton, Hyatt, IHG, Accor
- **OTAs**: Booking.com, Expedia, Priceline
- **Tecnología**: Oracle Hospitality, Amadeus, Sabre, Travelport
- **Asociaciones**: HTNG (Hotel Technology Next Generation), HEDNA

---

## 2. Versiones del Estándar OTA

### 2.1 Línea de Tiempo de Versiones

| Versión | Año | Características Principales |
|---------|-----|----------------------------|
| **OTA 1.0** | 1999-2015 | Especificaciones XML originales, DTDs |
| **2013B** | 2013 | Última versión ampliamente adoptada de 1.0 |
| **2015A** | 2015 | Versión más utilizada actualmente en producción |
| **2016A** | 2016 | Mejoras en contenido hotelero |
| **OTA 2.0** | 2016+ | Nueva arquitectura orientada a objetos |
| **2018A** | 2018 | Soporte JSON + XML, Swagger, escalabilidad |
| **2019A** | 2019 | Mejoras en contenido hotelero (push/pull) |
| **2020+** | 2020+ | Actualizaciones continuas, sostenibilidad, accesibilidad |

### 2.2 OTA 1.0 vs OTA 2.0

| Aspecto | OTA 1.0 | OTA 2.0 |
|---------|---------|---------|
| **Formato** | Solo XML | XML + JSON |
| **Arquitectura** | Mensajes independientes | Modelo de objetos reutilizables |
| **Documentación** | XSD Schemas | XSD + Swagger/OpenAPI |
| **Personalización** | Limitada | Alta (extensiones modulares) |
| **Adopción** | Universal (legacy) | Creciente (sistemas modernos) |
| **Complejidad** | Media | Alta (más flexible) |

**Recomendación para TravelHub**: Usar **OTA 2015A** para integraciones con PMSs legacy y migrar gradualmente a **OTA 2.0 (2019A+)** para nuevas integraciones.

---

## 3. Arquitectura del Estándar OTA

### 3.1 Estructura de Mensajes

Todos los mensajes OTA siguen una estructura consistente:

```
OTA_[Dominio][Función][Tipo]
```

**Componentes del nombre**:
- `OTA_`: Prefijo estándar
- `[Dominio]`: Hotel, Air, Vehicle, Rail, Cruise, etc.
- `[Función]`: Res (Reservation), Avail (Availability), Rate, Inv (Inventory), etc.
- `[Tipo]`: RQ (Request) o RS (Response)

**Ejemplos**:
- `OTA_HotelResRQ` / `OTA_HotelResRS`: Crear/modificar reserva de hotel
- `OTA_HotelAvailRQ` / `OTA_HotelAvailRS`: Consultar disponibilidad de hotel
- `OTA_HotelRateAmountNotifRQ` / `OTA_HotelRateAmountNotifRS`: Notificar cambios de tarifas
- `OTA_HotelInvCountNotifRQ` / `OTA_HotelInvCountNotifRS`: Notificar cambios de inventario

### 3.2 Patrón Request-Response

OTA utiliza comunicación **síncrona** basada en el patrón request-response:

1. **Sistema A** envía un mensaje `RQ` (Request) vía HTTP POST
2. **Sistema B** procesa la petición
3. **Sistema B** responde inmediatamente con un mensaje `RS` (Response)
4. La respuesta indica éxito o error con códigos estándar

```xml
<!-- Request -->
POST /api/ota HTTP/1.1
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<OTA_HotelResRQ xmlns="http://www.opentravel.org/OTA/2003/05" 
                Version="2015A" 
                TimeStamp="2026-02-25T22:00:00Z">
  <!-- Contenido de la petición -->
</OTA_HotelResRQ>

<!-- Response -->
HTTP/1.1 200 OK
Content-Type: application/xml

<?xml version="1.0" encoding="UTF-8"?>
<OTA_HotelResRS xmlns="http://www.opentravel.org/OTA/2003/05" 
                Version="2015A" 
                TimeStamp="2026-02-25T22:00:01Z">
  <Success/>
  <!-- Contenido de la respuesta -->
</OTA_HotelResRS>
```

### 3.3 Elementos Comunes

Todos los mensajes OTA comparten elementos comunes:

| Elemento | Descripción | Obligatorio |
|----------|-------------|-------------|
| `xmlns` | Namespace XML de OTA | Sí |
| `Version` | Versión del esquema (ej: "2015A") | Sí |
| `TimeStamp` | Fecha/hora del mensaje (ISO 8601) | Sí |
| `Target` | Sistema destino (producción/test) | No |
| `PrimaryLangID` | Idioma principal (ISO 639) | No |
| `EchoToken` | Token para correlacionar request/response | Recomendado |

**Respuestas exitosas**:
```xml
<Success/>
```

**Respuestas con errores**:
```xml
<Errors>
  <Error Type="3" Code="322">
    Invalid check-in date
  </Error>
</Errors>
```

**Tipos de error**:
- `Type="1"`: Unknown (error desconocido)
- `Type="3"`: Business rule (regla de negocio)
- `Type="4"`: Authentication (autenticación)
- `Type="6"`: Authorization (autorización)
- `Type="10"`: Required field missing (campo requerido faltante)
- `Type="13"`: Application error (error de aplicación)

---

## 4. Dominios Principales de OTA

### 4.1 Hotel (Hospitalidad)

El dominio más relevante para TravelHub. Incluye mensajes para:

#### A. Gestión de Inventario y Disponibilidad (ARI - Availability, Rates, Inventory)

| Mensaje | Propósito | Flujo |
|---------|-----------|-------|
| `OTA_HotelAvailNotifRQ/RS` | Actualizar disponibilidad y restricciones | PMS → Channel Manager |
| `OTA_HotelInvCountNotifRQ/RS` | Actualizar conteo de inventario | PMS → Channel Manager |
| `OTA_HotelRateAmountNotifRQ/RS` | Actualizar tarifas | PMS → Channel Manager |
| `OTA_HotelAvailRQ/RS` | Consultar disponibilidad | OTA → PMS |

#### B. Reservas

| Mensaje | Propósito | Flujo |
|---------|-----------|-------|
| `OTA_HotelResRQ/RS` | Crear/modificar reserva | OTA → PMS |
| `OTA_HotelResNotifRQ/RS` | Notificar nueva reserva | Channel Manager → PMS |
| `OTA_CancelRQ/RS` | Cancelar reserva | OTA → PMS |
| `OTA_ReadRQ/RS` | Leer detalles de reserva | OTA → PMS |

#### C. Contenido Descriptivo

| Mensaje | Propósito | Flujo |
|---------|-----------|-------|
| `OTA_HotelDescriptiveContentNotifRQ/RS` | Enviar contenido del hotel | PMS → OTA |
| `OTA_HotelDescriptiveInfoRQ/RS` | Solicitar contenido del hotel | OTA → PMS |

#### D. Búsqueda y Cotización

| Mensaje | Propósito | Flujo |
|---------|-----------|-------|
| `OTA_HotelSearchRQ/RS` | Buscar hoteles por criterios | OTA → Agregador |
| `OTA_HotelRatePlanRQ/RS` | Obtener planes de tarifas | OTA → PMS |

### 4.2 Otros Dominios (Expansión Futura de TravelHub)

| Dominio | Prefijo | Ejemplos de Mensajes |
|---------|---------|----------------------|
| **Aéreo** | `OTA_Air*` | `OTA_AirAvailRQ`, `OTA_AirBookRQ` |
| **Vehículos** | `OTA_Veh*` | `OTA_VehAvailRateRQ`, `OTA_VehResRQ` |
| **Tours/Actividades** | `OTA_Tour*` | `OTA_TourActivityAvailRQ`, `OTA_TourActivityBookRQ` |
| **Cruceros** | `OTA_Cruise*` | `OTA_CruiseCabinAvailRQ` |
| **Seguros** | `OTA_Insurance*` | `OTA_InsurancePlanSearchRQ` |
| **Paquetes** | `OTA_Pkg*` | `OTA_PkgBookRQ` (hotel+vuelo+actividades) |

**Relevancia para TravelHub**: Cuando se expanda a vuelos, tours y transporte, OTA proporciona mensajes estándar para integrar estos servicios.

---

## 5. Mensajes OTA Críticos para TravelHub

### 5.1 OTA_HotelAvailNotifRQ/RS (Actualización de Disponibilidad)

**Propósito**: El PMS del hotel notifica cambios en disponibilidad, restricciones y políticas.

**Flujo**: PMS Hotel → TravelHub

**Frecuencia**: Tiempo real o cada 5-15 minutos

**Estructura simplificada**:
```xml
<OTA_HotelAvailNotifRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z">
  <AvailStatusMessages HotelCode="HOTEL123">
    <AvailStatusMessage>
      <StatusApplicationControl 
        Start="2026-03-15" 
        End="2026-03-20" 
        InvTypeCode="DELUXE"
        RatePlanCode="BAR"/>
      <RestrictionStatus 
        Status="Open"
        MinLOS="2"
        MaxLOS="7"
        Restriction="Arrival"/>
      <LengthsOfStay>
        <LengthOfStay MinMaxMessageType="SetMinLOS" Time="2"/>
      </LengthsOfStay>
    </AvailStatusMessage>
  </AvailStatusMessages>
</OTA_HotelAvailNotifRQ>
```

**Elementos clave**:
- `HotelCode`: Identificador único del hotel
- `Start/End`: Rango de fechas
- `InvTypeCode`: Tipo de habitación
- `RatePlanCode`: Plan tarifario
- `Status`: Open (abierto) / Close (cerrado)
- `MinLOS/MaxLOS`: Mínima/máxima estadía
- `Restriction`: Arrival (llegada), Departure (salida), Stay (estadía)

### 5.2 OTA_HotelInvCountNotifRQ/RS (Actualización de Inventario)

**Propósito**: Actualizar el número de habitaciones disponibles.

**Flujo**: PMS Hotel → TravelHub

**Estructura simplificada**:
```xml
<OTA_HotelInvCountNotifRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z">
  <Inventories HotelCode="HOTEL123">
    <Inventory>
      <StatusApplicationControl 
        Start="2026-03-15" 
        End="2026-03-20" 
        InvTypeCode="DELUXE"/>
      <InvCounts>
        <InvCount Count="10" CountType="2"/>
        <!-- CountType="2" = Rooms available for sale -->
      </InvCounts>
    </Inventory>
  </Inventories>
</OTA_HotelInvCountNotifRQ>
```

**CountType valores**:
- `1`: Initial inventory (inventario inicial)
- `2`: Available for sale (disponible para venta)
- `3`: Out of order (fuera de servicio)
- `4`: Sold (vendido)

### 5.3 OTA_HotelRateAmountNotifRQ/RS (Actualización de Tarifas)

**Propósito**: Actualizar precios de habitaciones.

**Flujo**: PMS Hotel → TravelHub

**Estructura simplificada**:
```xml
<OTA_HotelRateAmountNotifRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z">
  <RateAmountMessages HotelCode="HOTEL123">
    <RateAmountMessage>
      <StatusApplicationControl 
        Start="2026-03-15" 
        End="2026-03-20" 
        InvTypeCode="DELUXE"
        RatePlanCode="BAR"/>
      <Rates>
        <Rate>
          <BaseByGuestAmts>
            <BaseByGuestAmt 
              AmountAfterTax="150.00" 
              CurrencyCode="USD"
              NumberOfGuests="2"/>
          </BaseByGuestAmts>
          <AdditionalGuestAmounts>
            <AdditionalGuestAmount 
              Amount="25.00" 
              AgeQualifyingCode="8"/>
            <!-- AgeQualifyingCode="8" = Adult -->
          </AdditionalGuestAmounts>
        </Rate>
      </Rates>
    </RateAmountMessage>
  </RateAmountMessages>
</OTA_HotelRateAmountNotifRQ>
```

**Elementos clave**:
- `AmountAfterTax`: Precio con impuestos incluidos
- `AmountBeforeTax`: Precio sin impuestos
- `CurrencyCode`: Código de moneda (ISO 4217)
- `NumberOfGuests`: Número de huéspedes
- `AdditionalGuestAmount`: Cargo por huésped adicional


### 5.4 OTA_HotelResRQ/RS (Crear/Modificar Reserva)

**Propósito**: Crear una nueva reserva o modificar una existente.

**Flujo**: TravelHub → PMS Hotel

**Estructura simplificada**:
```xml
<OTA_HotelResRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z" 
                ResStatus="Commit" EchoToken="req_abc123">
  <POS>
    <Source>
      <RequestorID Type="22" ID="TRAVELHUB"/>
      <!-- Type="22" = Tour Operator -->
    </Source>
  </POS>
  <HotelReservations>
    <HotelReservation>
      <RoomStays>
        <RoomStay>
          <RoomTypes>
            <RoomType RoomTypeCode="DELUXE" NumberOfUnits="1"/>
          </RoomTypes>
          <RatePlans>
            <RatePlan RatePlanCode="BAR"/>
          </RatePlans>
          <GuestCounts>
            <GuestCount AgeQualifyingCode="10" Count="2"/>
            <!-- AgeQualifyingCode="10" = Adult -->
          </GuestCounts>
          <TimeSpan Start="2026-03-15" End="2026-03-18"/>
          <BasicPropertyInfo HotelCode="HOTEL123"/>
          <Total AmountAfterTax="450.00" CurrencyCode="USD"/>
        </RoomStay>
      </RoomStays>
      <ResGuests>
        <ResGuest>
          <Profiles>
            <ProfileInfo>
              <Profile>
                <Customer>
                  <PersonName>
                    <GivenName>Juan</GivenName>
                    <Surname>Pérez</Surname>
                  </PersonName>
                  <Telephone PhoneNumber="+57-300-1234567"/>
                  <Email>juan.perez@example.com</Email>
                </Customer>
              </Profile>
            </ProfileInfo>
          </Profiles>
        </ResGuest>
      </ResGuests>
      <ResGlobalInfo>
        <Guarantee>
          <GuaranteesAccepted>
            <GuaranteeAccepted>
              <PaymentCard CardType="1" CardCode="VI" CardNumber="4111111111111111" 
                          ExpireDate="1228" SeriesCode="123">
                <CardHolderName>Juan Pérez</CardHolderName>
              </PaymentCard>
            </GuaranteeAccepted>
          </GuaranteesAccepted>
        </Guarantee>
        <DepositPayments>
          <GuaranteePayment>
            <AmountPercent Amount="450.00" CurrencyCode="USD"/>
          </GuaranteePayment>
        </DepositPayments>
        <CancelPenalties>
          <CancelPenalty>
            <Deadline AbsoluteDeadline="2026-03-13T18:00:00Z"/>
            <AmountPercent Percent="100.00"/>
          </CancelPenalty>
        </CancelPenalties>
      </ResGlobalInfo>
    </HotelReservation>
  </HotelReservations>
</OTA_HotelResRQ>
```

**Respuesta exitosa**:
```xml
<OTA_HotelResRS Version="2015A" TimeStamp="2026-02-25T22:00:01Z" 
                EchoToken="req_abc123">
  <Success/>
  <HotelReservations>
    <HotelReservation>
      <UniqueID Type="14" ID="CONF123456"/>
      <!-- Type="14" = Reservation -->
      <RoomStays>
        <RoomStay>
          <BasicPropertyInfo HotelCode="HOTEL123"/>
          <TimeSpan Start="2026-03-15" End="2026-03-18"/>
        </RoomStay>
      </RoomStays>
    </HotelReservation>
  </HotelReservations>
</OTA_HotelResRS>
```

**ResStatus valores**:
- `Commit`: Confirmar reserva inmediatamente
- `Modify`: Modificar reserva existente
- `Cancel`: Cancelar reserva
- `Ignore`: Ignorar (para testing)

### 5.5 OTA_CancelRQ/RS (Cancelar Reserva)

**Propósito**: Cancelar una reserva existente.

**Flujo**: TravelHub → PMS Hotel

**Estructura simplificada**:
```xml
<OTA_CancelRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z" 
              CancelType="Commit">
  <POS>
    <Source>
      <RequestorID Type="22" ID="TRAVELHUB"/>
    </Source>
  </POS>
  <UniqueID Type="14" ID="CONF123456"/>
  <Verification>
    <PersonName>
      <GivenName>Juan</GivenName>
      <Surname>Pérez</Surname>
    </PersonName>
  </Verification>
</OTA_CancelRQ>
```

**Respuesta**:
```xml
<OTA_CancelRS Version="2015A" TimeStamp="2026-02-25T22:00:01Z">
  <Success/>
  <UniqueID Type="14" ID="CONF123456"/>
  <CancelInfoRS>
    <CancelRule>
      <CancelPenalty>
        <AmountPercent Amount="0.00" CurrencyCode="USD"/>
      </CancelPenalty>
    </CancelRule>
  </CancelInfoRS>
</OTA_CancelRS>
```

### 5.6 OTA_HotelDescriptiveContentNotifRQ/RS (Contenido del Hotel)

**Propósito**: Enviar información descriptiva del hotel (fotos, amenidades, políticas).

**Flujo**: PMS Hotel → TravelHub

**Estructura simplificada**:
```xml
<OTA_HotelDescriptiveContentNotifRQ Version="2015A" TimeStamp="2026-02-25T22:00:00Z">
  <HotelDescriptiveContents>
    <HotelDescriptiveContent HotelCode="HOTEL123" HotelName="Hotel Paradise">
      <HotelInfo>
        <Position Latitude="4.6097" Longitude="-74.0817"/>
        <Services>
          <Service Code="68"/> <!-- WiFi -->
          <Service Code="76"/> <!-- Parking -->
          <Service Code="4"/>  <!-- Restaurant -->
        </Services>
        <CategoryCodes>
          <GuestRoomInfo>
            <Amenities>
              <Amenity RoomAmenityCode="26"/> <!-- Air conditioning -->
              <Amenity RoomAmenityCode="3"/>  <!-- TV -->
              <Amenity RoomAmenityCode="123"/> <!-- Safe -->
            </Amenities>
          </GuestRoomInfo>
        </CategoryCodes>
      </HotelInfo>
      <FacilityInfo>
        <Meetings>
          <Meeting MeetingRoomCapacity="50"/>
        </Meetings>
      </FacilityInfo>
      <Policies>
        <Policy CheckInTime="15:00:00" CheckOutTime="12:00:00">
          <PolicyInfo>
            <CheckInTime>15:00:00</CheckInTime>
            <CheckOutTime>12:00:00</CheckOutTime>
          </PolicyInfo>
        </Policy>
      </Policies>
      <MultimediaDescriptions>
        <MultimediaDescription>
          <ImageItems>
            <ImageItem>
              <ImageFormat>
                <URL>https://cdn.hotel.com/images/room1.jpg</URL>
              </ImageFormat>
            </ImageItem>
          </ImageItems>
        </MultimediaDescription>
      </MultimediaDescriptions>
      <ContactInfos>
        <ContactInfo ContactProfileType="general">
          <Addresses>
            <Address>
              <AddressLine>Calle 123 #45-67</AddressLine>
              <CityName>Bogotá</CityName>
              <PostalCode>110111</PostalCode>
              <CountryName Code="CO">Colombia</CountryName>
            </Address>
          </Addresses>
          <Phones>
            <Phone PhoneNumber="+57-1-1234567" PhoneTechType="1"/>
            <!-- PhoneTechType="1" = Voice -->
          </Phones>
          <Emails>
            <Email>info@hotelparadise.com</Email>
          </Emails>
        </ContactInfo>
      </ContactInfos>
    </HotelDescriptiveContent>
  </HotelDescriptiveContents>
</OTA_HotelDescriptiveContentNotifRQ>
```

---

## 6. Códigos y Listas de Valores OTA

OTA mantiene listas de códigos estandarizados (Code Lists) para garantizar interoperabilidad.

### 6.1 Códigos Principales

#### A. Tipos de Habitación (Room Type Codes)
Definidos por cada hotel, pero con convenciones comunes:
- `SGL`: Single
- `DBL`: Double
- `TWN`: Twin
- `STE`: Suite
- `DELUXE`: Deluxe Room
- `FAMILY`: Family Room

#### B. Planes Tarifarios (Rate Plan Codes)
- `BAR`: Best Available Rate
- `CORP`: Corporate Rate
- `GOV`: Government Rate
- `PKG`: Package Rate
- `PROMO`: Promotional Rate

#### C. Tipos de Tarjeta de Crédito (Card Type)
- `1`: Credit Card
- `2`: Debit Card
- `3`: Voucher

#### D. Códigos de Tarjeta (Card Code)
- `VI`: Visa
- `MC`: MasterCard
- `AX`: American Express
- `DC`: Diners Club
- `DS`: Discover

#### E. Calificadores de Edad (Age Qualifying Code)
- `8`: Adult
- `10`: Adult (alternativo)
- `7`: Child
- `3`: Infant

#### F. Tipos de Identificador (UniqueID Type)
- `14`: Reservation
- `15`: Cancellation number
- `16`: Confirmation number
- `21`: Profile
- `22`: Tour operator

#### G. Tipos de Requestor (RequestorID Type)
- `1`: Customer
- `4`: Travel agent
- `5`: Corporation
- `22`: Tour operator
- `18`: Other

### 6.2 Códigos de Amenidades

**Amenidades de Habitación (Room Amenity Code)**:
- `3`: Television
- `26`: Air conditioning
- `28`: Desk
- `50`: Hairdryer
- `69`: Mini bar
- `123`: In-room safe
- `261`: Coffee/tea maker

**Servicios del Hotel (Hotel Amenity Code)**:
- `4`: Restaurant
- `68`: Internet access
- `71`: Wireless internet
- `76`: Parking
- `88`: Fitness center
- `110`: Swimming pool
- `166`: Business center

### 6.3 Códigos de País y Moneda

OTA utiliza estándares ISO:
- **País**: ISO 3166-1 alpha-2 (CO, BR, MX, AR, PE, CL)
- **Moneda**: ISO 4217 (USD, COP, BRL, MXN, ARS, PEN, CLP)
- **Idioma**: ISO 639-1 (es, en, pt)

---

## 7. Integración OTA en TravelHub

### 7.1 Arquitectura de Integración

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────┐
│   PMS Hotel 1   │         │                  │         │             │
│   (OTA 2015A)   │────────▶│                  │         │             │
└─────────────────┘         │                  │         │             │
                            │  OTA Adapter     │────────▶│  TravelHub  │
┌─────────────────┐         │  (Normalización) │         │  Inventory  │
│   PMS Hotel 2   │         │                  │         │  Service    │
│   (OTA 2019A)   │────────▶│                  │         │             │
└─────────────────┘         │                  │         │             │
                            └──────────────────┘         └─────────────┘
┌─────────────────┐                  │
│   PMS Hotel 3   │                  │
│   (Propietario) │──────────────────┘
└─────────────────┘
```

### 7.2 Componentes de Integración

#### A. OTA Adapter Service
**Responsabilidades**:
- Recibir mensajes OTA de múltiples versiones (2013B, 2015A, 2019A)
- Validar XML contra esquemas XSD
- Normalizar a modelo interno de TravelHub
- Transformar modelo interno a OTA para envío
- Manejo de errores y reintentos
- Logging y auditoría

**Tecnologías sugeridas**:
- **Spring Boot** + **Apache Camel** (Java)
- **FastAPI** + **lxml** (Python)
- **Node.js** + **xml2js**

#### B. Schema Validation
Validar todos los mensajes contra esquemas XSD oficiales:
```bash
# Descargar esquemas desde opentravel.org
wget https://www.opentravel.org/Specifications/OnlineXMLSchemas/OTA2015A.zip
```

**Validación en Java**:
```java
SchemaFactory factory = SchemaFactory.newInstance(XMLConstants.W3C_XML_SCHEMA_NS_URI);
Schema schema = factory.newSchema(new File("OTA_HotelResRQ.xsd"));
Validator validator = schema.newValidator();
validator.validate(new StreamSource(new StringReader(xmlMessage)));
```

#### C. Mapeo de Datos

**Ejemplo: OTA → Modelo Interno TravelHub**

| Campo OTA | Campo TravelHub | Transformación |
|-----------|-----------------|----------------|
| `HotelCode` | `hotelId` | Directo |
| `InvTypeCode` | `roomTypeId` | Lookup en tabla de mapeo |
| `RatePlanCode` | `ratePlanId` | Lookup en tabla de mapeo |
| `AmountAfterTax` | `totalAmount` | Directo |
| `CurrencyCode` | `currency` | Directo |
| `Start` | `checkIn` | Parsear fecha ISO 8601 |
| `End` | `checkOut` | Parsear fecha ISO 8601 |

### 7.3 Flujos de Integración Críticos

#### Flujo 1: Actualización de Inventario (PMS → TravelHub)

```
1. PMS Hotel detecta cambio en inventario
2. PMS genera OTA_HotelInvCountNotifRQ
3. PMS envía POST a https://api.travelhub.com/ota/inventory
4. TravelHub OTA Adapter recibe mensaje
5. Adapter valida XML contra XSD
6. Adapter normaliza a modelo interno
7. Adapter llama a Inventory Service (gRPC/REST)
8. Inventory Service actualiza base de datos
9. Inventory Service publica evento InventoryUpdated
10. Adapter genera OTA_HotelInvCountNotifRS (Success)
11. Adapter responde al PMS
```

**Tiempo esperado**: < 500ms

#### Flujo 2: Crear Reserva (TravelHub → PMS)

```
1. Usuario completa reserva en TravelHub
2. Booking Service valida disponibilidad
3. Booking Service procesa pago
4. Booking Service llama a OTA Adapter
5. Adapter transforma modelo interno a OTA_HotelResRQ
6. Adapter envía POST a PMS endpoint
7. PMS procesa reserva
8. PMS responde OTA_HotelResRS con confirmación
9. Adapter extrae UniqueID (código de confirmación)
10. Adapter actualiza Booking Service
11. Booking Service envía email de confirmación
```

**Tiempo esperado**: < 3 segundos

#### Flujo 3: Sincronización de Contenido (PMS → TravelHub)

```
1. Hotel actualiza fotos/amenidades en PMS
2. PMS genera OTA_HotelDescriptiveContentNotifRQ
3. PMS envía a TravelHub
4. Adapter normaliza contenido
5. Content Service actualiza base de datos
6. Content Service invalida caché de CDN
7. Adapter responde Success
```

**Frecuencia**: Bajo demanda o diario

### 7.4 Manejo de Errores

#### Errores Comunes y Soluciones

| Error OTA | Código | Causa | Solución TravelHub |
|-----------|--------|-------|-------------------|
| Invalid date range | 322 | Check-out antes de check-in | Validar en frontend |
| Room not available | 392 | Inventario agotado | Mostrar "No disponible" |
| Invalid credit card | 385 | Tarjeta rechazada | Solicitar otro método de pago |
| Duplicate reservation | 400 | Reserva ya existe | Verificar UniqueID antes de enviar |
| Authentication failed | 320 | Credenciales inválidas | Renovar token de autenticación |

#### Estrategia de Reintentos

```yaml
retry_policy:
  max_attempts: 3
  backoff: exponential
  initial_delay: 1s
  max_delay: 30s
  retryable_errors:
    - timeout
    - connection_refused
    - 500 (Internal Server Error)
    - 502 (Bad Gateway)
    - 503 (Service Unavailable)
  non_retryable_errors:
    - 400 (Bad Request)
    - 401 (Unauthorized)
    - 404 (Not Found)
```

---

## 8. Ventajas y Desventajas de OTA

### 8.1 Ventajas

✅ **Estándar de la industria**: Adoptado por miles de hoteles y sistemas PMS  
✅ **Interoperabilidad**: Integración con múltiples proveedores sin desarrollo custom  
✅ **Documentación completa**: Esquemas XSD, guías de implementación, ejemplos  
✅ **Versionamiento claro**: Compatibilidad hacia atrás entre versiones  
✅ **Cobertura completa**: Mensajes para todos los aspectos del ciclo de vida de reservas  
✅ **Soporte multi-idioma**: Campos para múltiples idiomas y monedas  
✅ **Comunidad activa**: Foros, grupos de trabajo, actualizaciones regulares  

### 8.2 Desventajas

❌ **Complejidad**: Esquemas XML muy extensos y anidados  
❌ **Verbosidad**: Mensajes XML grandes (3-10 KB por mensaje simple)  
❌ **Curva de aprendizaje**: Requiere tiempo para dominar todos los mensajes  
❌ **Variabilidad de implementación**: Cada PMS implementa subconjuntos diferentes  
❌ **Sincronía obligatoria**: No hay soporte nativo para mensajería asíncrona  
❌ **Limitaciones de personalización**: Difícil agregar campos custom sin romper estándar  
❌ **Performance**: Parsing XML es más lento que JSON  

### 8.3 Comparación con Alternativas

| Aspecto | OTA | REST/JSON Custom | GraphQL |
|---------|-----|------------------|---------|
| **Adopción en hotelería** | Universal | Baja | Muy baja |
| **Facilidad de integración** | Media | Alta | Alta |
| **Flexibilidad** | Baja | Alta | Muy alta |
| **Estandarización** | Alta | Baja | Media |
| **Performance** | Media | Alta | Alta |
| **Documentación** | Excelente | Variable | Buena |
| **Costo de desarrollo** | Medio | Bajo | Medio |

**Recomendación**: Usar OTA para integraciones con PMSs externos (estándar de facto) y REST/JSON para APIs internas de TravelHub.

---

## 9. Mejores Prácticas de Implementación

### 9.1 Validación Estricta
- Validar todos los mensajes contra esquemas XSD antes de procesar
- Rechazar mensajes inválidos con errores descriptivos
- Implementar validación de reglas de negocio adicionales

### 9.2 Idempotencia
- Usar `EchoToken` para correlacionar requests/responses
- Detectar y rechazar mensajes duplicados
- Implementar deduplicación basada en `UniqueID`

### 9.3 Logging y Auditoría
- Registrar todos los mensajes OTA (request + response)
- Incluir timestamps, IDs de correlación, y resultados
- Retener logs por mínimo 12 meses para auditorías

### 9.4 Seguridad
- Usar HTTPS/TLS 1.3 para todas las comunicaciones
- Implementar autenticación mutua (mTLS) para partners críticos
- Validar y sanitizar todos los inputs XML (prevenir XXE attacks)
- No registrar datos sensibles (números de tarjeta completos)

### 9.5 Monitoreo
- Métricas de latencia por tipo de mensaje
- Tasa de errores por hotel/PMS
- Volumen de mensajes por hora/día
- Alertas para degradación de servicio

### 9.6 Versionamiento
- Soportar múltiples versiones OTA simultáneamente (2013B, 2015A, 2019A)
- Detectar versión del mensaje entrante automáticamente
- Migrar gradualmente hoteles a versiones más recientes

### 9.7 Testing
- Crear suite de mensajes OTA de prueba (happy path + edge cases)
- Implementar contract testing con esquemas XSD
- Realizar pruebas de integración con PMSs en ambiente staging
- Simular errores de red y timeouts

---

## 10. Roadmap de Adopción OTA en TravelHub

### Fase 1: Fundamentos (Meses 1-2)
- [ ] Descargar y estudiar especificaciones OTA 2015A
- [ ] Implementar OTA Adapter Service básico
- [ ] Soportar mensajes críticos:
  - `OTA_HotelAvailNotifRQ/RS`
  - `OTA_HotelInvCountNotifRQ/RS`
  - `OTA_HotelRateAmountNotifRQ/RS`
- [ ] Integrar con 3 hoteles piloto
- [ ] Implementar validación XSD

### Fase 2: Reservas (Meses 3-4)
- [ ] Implementar mensajes de reservas:
  - `OTA_HotelResRQ/RS`
  - `OTA_CancelRQ/RS`
  - `OTA_ReadRQ/RS`
- [ ] Integrar con pasarela de pagos
- [ ] Implementar manejo de errores robusto
- [ ] Escalar a 50 hoteles

### Fase 3: Contenido (Meses 5-6)
- [ ] Implementar mensajes de contenido:
  - `OTA_HotelDescriptiveContentNotifRQ/RS`
  - `OTA_HotelDescriptiveInfoRQ/RS`
- [ ] Sincronizar fotos, amenidades, políticas
- [ ] Implementar caché de contenido
- [ ] Escalar a 200 hoteles

### Fase 4: Optimización (Meses 7-9)
- [ ] Migrar a OTA 2.0 (2019A) para nuevas integraciones
- [ ] Implementar soporte JSON (OTA 2.0)
- [ ] Optimizar performance (< 200ms latencia)
- [ ] Implementar circuit breakers y rate limiting
- [ ] Escalar a 500+ hoteles

### Fase 5: Expansión (Meses 10-12)
- [ ] Soportar mensajes de tours/actividades (para Basecamp)
- [ ] Soportar mensajes de vehículos (para expansión futura)
- [ ] Implementar paquetes dinámicos (hotel+vuelo+tour)
- [ ] Escalar a 1,200+ hoteles

---

## 11. Recursos y Referencias

### 11.1 Documentación Oficial
- **Sitio web**: https://opentravel.org
- **Especificaciones**: https://opentravel.org/download-specs/
- **Code Lists**: https://opentravel.org/download-code-list/
- **Guías de implementación**: Disponibles para miembros

### 11.2 Herramientas
- **Validadores XML**: XMLSpy, Oxygen XML Editor
- **Librerías Java**: JAXB, Apache CXF
- **Librerías Python**: lxml, xmlschema
- **Librerías Node.js**: xml2js, fast-xml-parser

### 11.3 Comunidad
- **Foros**: OpenTravel Community Forums
- **LinkedIn**: OpenTravel Alliance Group
- **Eventos**: OpenTravel Advisory Forum (anual)

### 11.4 Proveedores de Soluciones OTA
- **SiteMinder**: Channel manager con soporte OTA completo
- **Cloudbeds**: PMS con APIs OTA
- **RateGain**: Conectividad OTA para hoteles
- **D-EDGE**: Soluciones de distribución hotelera

---

## 12. Conclusión

OpenTravel Alliance (OTA) es el estándar de facto para la integración de sistemas en la industria hotelera. Para TravelHub, adoptar OTA significa:

✅ **Integración rápida** con 1,200+ hoteles existentes  
✅ **Interoperabilidad** con PMSs de mercado (Opera, Cloudbeds, etc.)  
✅ **Reducción de desarrollo custom** (80% de integraciones estandarizadas)  
✅ **Escalabilidad** para expansión a nuevos países y servicios  
✅ **Preparación futura** para adquisiciones (Basecamp, NightStay)  

**Inversión estimada**:
- **Desarrollo inicial**: 4-6 meses (2 desarrolladores)
- **Costo de licencias**: $0 (estándar abierto)
- **Infraestructura**: Incluida en arquitectura de microservicios

**ROI esperado**:
- Reducción del 70% en tiempo de onboarding de hoteles (2 semanas → 2 días)
- Reducción del 60% en tickets de soporte por inconsistencias de datos
- Habilitación de expansión a Brasil y otros mercados

**Recomendación final**: Implementar OTA 2015A como prioridad en Fase 1 de la transformación arquitectónica, comenzando con los 3 mensajes ARI (Availability, Rates, Inventory) que resuelven el 80% de los problemas operacionales actuales.

---

Content was rephrased for compliance with licensing restrictions.

**Referencias:**
[1] OpenTravel Alliance - https://opentravel.org/
[2] OpenTravel 2019A Object Suite - https://www.hospitalitynet.org/news/4096727.html
[3] OpenTravel Specifications - https://opentravel.org/download-specs/
[4] OTA Hotel Messaging Documentation - https://tourisoft.atlassian.net/wiki/spaces/HOT/pages/6520882/
