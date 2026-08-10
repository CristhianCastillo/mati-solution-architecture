# Salesforce REST API - Descripción del Estándar (CRM)

## 1. ¿Qué es Salesforce REST API?

**Salesforce REST API** es una interfaz RESTful que permite a aplicaciones externas interactuar con datos de Salesforce (CRM) mediante operaciones HTTP estándar. Es el método principal para integrar sistemas externos con Salesforce sin usar la interfaz web.

**Características principales**:
- Protocolo REST sobre HTTP/HTTPS
- Formato JSON o XML (JSON preferido)
- Operaciones CRUD completas (Create, Read, Update, Delete)
- Autenticación OAuth 2.0
- Rate limiting basado en licencias
- Versionado de API (v58.0, v59.0, etc.)

**Uso en TravelHub**: Sincronizar datos de hoteles, clientes y oportunidades entre TravelHub y Salesforce.

---

## 2. Arquitectura de Integración

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│  TravelHub      │         │  Salesforce      │         │  Salesforce     │
│  (Backend)      │◄───────▶│  REST API        │◄───────▶│  Database       │
│                 │  HTTPS  │  (OAuth 2.0)     │         │  (Accounts,     │
│                 │  JSON   │                  │         │   Contacts,     │
│                 │         │                  │         │   Opportunities)│
└─────────────────┘         └──────────────────┘         └─────────────────┘
```

---

## 3. Autenticación OAuth 2.0

Salesforce utiliza **OAuth 2.0** como mecanismo de autenticación estándar.

### 3.1 Flujos de Autenticación

#### A. Username-Password Flow (Server-to-Server)

**Uso**: Integraciones backend sin interacción de usuario.

**Flujo**:
```
1. TravelHub envía credenciales a Salesforce
2. Salesforce valida y devuelve access_token
3. TravelHub usa access_token en cada request
4. Token expira después de N horas
5. TravelHub usa refresh_token para renovar
```

**Request**:
```http
POST https://login.salesforce.com/services/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=password
&client_id=3MVG9...ABC123
&client_secret=1234567890ABCDEF
&username=travelhub@company.com
&password=MyPassword123SecurityToken
```

**Response**:
```json
{
  "access_token": "00D5g000000abcd!AR8AQP0jITN80ESEsj5EbaZTFG0RNBaT1cyWk7TrqoDjoNIWQ2ME_sTZzBjfmOE6zMHq6y8PIW4eWze9JksNEkWUl.Cju7m4",
  "instance_url": "https://na1.salesforce.com",
  "id": "https://login.salesforce.com/id/00D5g000000abcd/0055g000000abcd",
  "token_type": "Bearer",
  "issued_at": "1709251200000",
  "signature": "abcdef1234567890"
}
```

#### B. Web Server Flow (OAuth 2.0 Authorization Code)

**Uso**: Aplicaciones web con usuarios interactivos.

**Flujo**:
```
1. Usuario hace clic en "Connect to Salesforce"
2. Redirige a Salesforce login
3. Usuario autoriza acceso
4. Salesforce redirige con authorization_code
5. TravelHub intercambia code por access_token
```

#### C. Refresh Token Flow

**Uso**: Renovar access_token expirado sin re-autenticar.

**Request**:
```http
POST https://login.salesforce.com/services/oauth2/token
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token
&client_id=3MVG9...ABC123
&client_secret=1234567890ABCDEF
&refresh_token=5Aep861...xyz789
```

### 3.2 Configuración en Salesforce

**Pasos**:
1. Setup → App Manager → New Connected App
2. Configurar:
   - **App Name**: TravelHub Integration
   - **API Name**: TravelHub_Integration
   - **Contact Email**: tech@travelhub.com
   - **Enable OAuth Settings**: ✓
   - **Callback URL**: https://api.travelhub.com/oauth/callback
   - **Selected OAuth Scopes**:
     - Full access (full)
     - Perform requests at any time (refresh_token, offline_access)
     - Access and manage your data (api)
3. Obtener **Consumer Key** (client_id) y **Consumer Secret** (client_secret)

---

## 4. Endpoints Principales

### 4.1 Base URL

```
https://{instance}.salesforce.com/services/data/v{version}/
```

**Ejemplo**:
```
https://na1.salesforce.com/services/data/v58.0/
```

### 4.2 Recursos Estándar

#### A. Listar Objetos Disponibles

```http
GET /services/data/v58.0/sobjects/
Authorization: Bearer {access_token}
```

**Response**:
```json
{
  "sobjects": [
    {
      "name": "Account",
      "label": "Account",
      "urls": {
        "sobject": "/services/data/v58.0/sobjects/Account"
      }
    },
    {
      "name": "Contact",
      "label": "Contact",
      "urls": {
        "sobject": "/services/data/v58.0/sobjects/Contact"
      }
    },
    {
      "name": "Opportunity",
      "label": "Opportunity",
      "urls": {
        "sobject": "/services/data/v58.0/sobjects/Opportunity"
      }
    }
  ]
}
```

#### B. Describir Objeto (Metadata)

```http
GET /services/data/v58.0/sobjects/Account/describe/
Authorization: Bearer {access_token}
```

**Response**: Devuelve todos los campos, tipos de datos, relaciones, permisos.

#### C. Crear Registro (POST)

**Crear cuenta (hotel)**:
```http
POST /services/data/v58.0/sobjects/Account/
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "Name": "Hotel Paradise",
  "Type": "Partner",
  "Industry": "Hospitality",
  "Phone": "+57-1-1234567",
  "BillingStreet": "Avenida San Martín 100",
  "BillingCity": "Cartagena",
  "BillingState": "Bolívar",
  "BillingPostalCode": "130001",
  "BillingCountry": "Colombia",
  "Website": "https://hotelparadise.com",
  "NumberOfEmployees": 50,
  "AnnualRevenue": 500000,
  "Custom_Field__c": "Value"
}
```

**Response**:
```json
{
  "id": "0015g00000ABCDEFGHI",
  "success": true,
  "errors": []
}
```

#### D. Leer Registro (GET)

```http
GET /services/data/v58.0/sobjects/Account/0015g00000ABCDEFGHI
Authorization: Bearer {access_token}
```

**Response**:
```json
{
  "attributes": {
    "type": "Account",
    "url": "/services/data/v58.0/sobjects/Account/0015g00000ABCDEFGHI"
  },
  "Id": "0015g00000ABCDEFGHI",
  "Name": "Hotel Paradise",
  "Type": "Partner",
  "Industry": "Hospitality",
  "Phone": "+57-1-1234567",
  "BillingCity": "Cartagena",
  "BillingCountry": "Colombia",
  "Website": "https://hotelparadise.com",
  "CreatedDate": "2026-02-25T22:00:00.000+0000",
  "LastModifiedDate": "2026-02-25T22:00:00.000+0000"
}
```

#### E. Actualizar Registro (PATCH)

```http
PATCH /services/data/v58.0/sobjects/Account/0015g00000ABCDEFGHI
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "Phone": "+57-1-7654321",
  "Website": "https://newhotelparadise.com"
}
```

**Response**:
```http
HTTP/1.1 204 No Content
```

#### F. Eliminar Registro (DELETE)

```http
DELETE /services/data/v58.0/sobjects/Account/0015g00000ABCDEFGHI
Authorization: Bearer {access_token}
```

**Response**:
```http
HTTP/1.1 204 No Content
```

---

## 5. Consultas SOQL (Salesforce Object Query Language)

### 5.1 Query Endpoint

```http
GET /services/data/v58.0/query/?q={SOQL_QUERY}
Authorization: Bearer {access_token}
```

### 5.2 Ejemplos de Consultas

**Obtener todos los hoteles (Accounts de tipo Partner)**:
```sql
SELECT Id, Name, Phone, BillingCity, BillingCountry, Website
FROM Account
WHERE Type = 'Partner' AND Industry = 'Hospitality'
ORDER BY Name ASC
LIMIT 100
```

**URL encoded**:
```http
GET /services/data/v58.0/query/?q=SELECT+Id,Name,Phone+FROM+Account+WHERE+Type='Partner'
```

**Response**:
```json
{
  "totalSize": 150,
  "done": false,
  "nextRecordsUrl": "/services/data/v58.0/query/01g5g00000ABCDEF-2000",
  "records": [
    {
      "attributes": {
        "type": "Account",
        "url": "/services/data/v58.0/sobjects/Account/0015g00000ABCDEFGHI"
      },
      "Id": "0015g00000ABCDEFGHI",
      "Name": "Hotel Paradise",
      "Phone": "+57-1-1234567",
      "BillingCity": "Cartagena",
      "BillingCountry": "Colombia"
    }
  ]
}
```

**Paginación**:
```http
GET /services/data/v58.0/query/01g5g00000ABCDEF-2000
```

**Consulta con relaciones (JOIN)**:
```sql
SELECT Id, Name, 
       (SELECT Id, FirstName, LastName, Email FROM Contacts)
FROM Account
WHERE Type = 'Partner'
```

---

## 6. Operaciones Batch (Composite API)

### 6.1 Composite Request

**Ejecutar múltiples operaciones en una sola request**:

```http
POST /services/data/v58.0/composite/
Authorization: Bearer {access_token}
Content-Type: application/json

{
  "allOrNone": false,
  "compositeRequest": [
    {
      "method": "POST",
      "url": "/services/data/v58.0/sobjects/Account/",
      "referenceId": "refAccount1",
      "body": {
        "Name": "Hotel A",
        "Type": "Partner"
      }
    },
    {
      "method": "POST",
      "url": "/services/data/v58.0/sobjects/Contact/",
      "referenceId": "refContact1",
      "body": {
        "FirstName": "Juan",
        "LastName": "Pérez",
        "AccountId": "@{refAccount1.id}",
        "Email": "juan@hotela.com"
      }
    }
  ]
}
```

**Response**:
```json
{
  "compositeResponse": [
    {
      "referenceId": "refAccount1",
      "httpStatusCode": 201,
      "body": {
        "id": "0015g00000XYZ123",
        "success": true
      }
    },
    {
      "referenceId": "refContact1",
      "httpStatusCode": 201,
      "body": {
        "id": "0035g00000ABC456",
        "success": true
      }
    }
  ]
}
```

---

## 7. Implementación en TravelHub

### 7.1 Cliente Salesforce

```python
import requests
from typing import Dict, List, Optional
from datetime import datetime, timedelta

class SalesforceClient:
    def __init__(self, client_id: str, client_secret: str, username: str, password: str):
        self.client_id = client_id
        self.client_secret = client_secret
        self.username = username
        self.password = password
        self.access_token = None
        self.instance_url = None
        self.token_expires_at = None
        
        self._authenticate()
    
    def _authenticate(self):
        """Autenticar usando Username-Password flow"""
        url = "https://login.salesforce.com/services/oauth2/token"
        
        data = {
            "grant_type": "password",
            "client_id": self.client_id,
            "client_secret": self.client_secret,
            "username": self.username,
            "password": self.password
        }
        
        response = requests.post(url, data=data)
        response.raise_for_status()
        
        result = response.json()
        self.access_token = result["access_token"]
        self.instance_url = result["instance_url"]
        self.token_expires_at = datetime.now() + timedelta(hours=2)
    
    def _ensure_authenticated(self):
        """Verificar que el token no haya expirado"""
        if datetime.now() >= self.token_expires_at:
            self._authenticate()
    
    def _get_headers(self) -> Dict:
        """Headers para requests"""
        self._ensure_authenticated()
        return {
            "Authorization": f"Bearer {self.access_token}",
            "Content-Type": "application/json"
        }
    
    def query(self, soql: str) -> List[Dict]:
        """Ejecutar consulta SOQL"""
        url = f"{self.instance_url}/services/data/v58.0/query/"
        params = {"q": soql}
        
        response = requests.get(url, headers=self._get_headers(), params=params)
        response.raise_for_status()
        
        result = response.json()
        records = result["records"]
        
        # Manejar paginación
        while not result["done"]:
            next_url = f"{self.instance_url}{result['nextRecordsUrl']}"
            response = requests.get(next_url, headers=self._get_headers())
            response.raise_for_status()
            result = response.json()
            records.extend(result["records"])
        
        return records
    
    def create_record(self, sobject: str, data: Dict) -> str:
        """Crear registro"""
        url = f"{self.instance_url}/services/data/v58.0/sobjects/{sobject}/"
        
        response = requests.post(url, headers=self._get_headers(), json=data)
        response.raise_for_status()
        
        result = response.json()
        return result["id"]
    
    def get_record(self, sobject: str, record_id: str, fields: Optional[List[str]] = None) -> Dict:
        """Obtener registro"""
        url = f"{self.instance_url}/services/data/v58.0/sobjects/{sobject}/{record_id}"
        
        params = {}
        if fields:
            params["fields"] = ",".join(fields)
        
        response = requests.get(url, headers=self._get_headers(), params=params)
        response.raise_for_status()
        
        return response.json()
    
    def update_record(self, sobject: str, record_id: str, data: Dict) -> bool:
        """Actualizar registro"""
        url = f"{self.instance_url}/services/data/v58.0/sobjects/{sobject}/{record_id}"
        
        response = requests.patch(url, headers=self._get_headers(), json=data)
        response.raise_for_status()
        
        return response.status_code == 204
    
    def delete_record(self, sobject: str, record_id: str) -> bool:
        """Eliminar registro"""
        url = f"{self.instance_url}/services/data/v58.0/sobjects/{sobject}/{record_id}"
        
        response = requests.delete(url, headers=self._get_headers())
        response.raise_for_status()
        
        return response.status_code == 204

# Uso
sf = SalesforceClient(
    client_id="3MVG9...ABC123",
    client_secret="1234567890ABCDEF",
    username="travelhub@company.com",
    password="MyPassword123SecurityToken"
)

# Crear hotel en Salesforce
hotel_id = sf.create_record("Account", {
    "Name": "Hotel Paradise",
    "Type": "Partner",
    "Industry": "Hospitality",
    "Phone": "+57-1-1234567",
    "BillingCity": "Cartagena",
    "BillingCountry": "Colombia"
})

# Consultar hoteles
hotels = sf.query("""
    SELECT Id, Name, Phone, BillingCity
    FROM Account
    WHERE Type = 'Partner' AND Industry = 'Hospitality'
""")

# Actualizar teléfono
sf.update_record("Account", hotel_id, {
    "Phone": "+57-1-7654321"
})
```

### 7.2 Sincronización Bidireccional

```python
def sync_hotel_to_salesforce(hotel: Dict):
    """Sincronizar hotel de TravelHub a Salesforce"""
    
    # Buscar si ya existe en Salesforce
    existing = sf.query(f"""
        SELECT Id FROM Account
        WHERE External_ID__c = '{hotel['id']}'
    """)
    
    salesforce_data = {
        "Name": hotel["name"],
        "Type": "Partner",
        "Industry": "Hospitality",
        "Phone": hotel["phone"],
        "BillingStreet": hotel["address"],
        "BillingCity": hotel["city"],
        "BillingCountry": hotel["country"],
        "Website": hotel["website"],
        "External_ID__c": hotel["id"]  # Campo custom para mapeo
    }
    
    if existing:
        # Actualizar
        sf.update_record("Account", existing[0]["Id"], salesforce_data)
        return existing[0]["Id"]
    else:
        # Crear
        return sf.create_record("Account", salesforce_data)

def sync_salesforce_to_travelhub():
    """Sincronizar cambios de Salesforce a TravelHub"""
    
    # Obtener hoteles modificados en las últimas 24 horas
    yesterday = (datetime.now() - timedelta(days=1)).isoformat()
    
    hotels = sf.query(f"""
        SELECT Id, Name, Phone, BillingCity, BillingCountry, External_ID__c
        FROM Account
        WHERE Type = 'Partner' 
          AND Industry = 'Hospitality'
          AND LastModifiedDate >= {yesterday}
    """)
    
    for hotel in hotels:
        # Actualizar en TravelHub
        update_hotel_in_travelhub(
            hotel_id=hotel["External_ID__c"],
            name=hotel["Name"],
            phone=hotel["Phone"],
            city=hotel["BillingCity"],
            country=hotel["BillingCountry"]
        )
```

---

## 8. Rate Limiting

Salesforce impone límites de API basados en el tipo de licencia:

| Licencia | Límite Diario | Límite por Request |
|----------|---------------|-------------------|
| **Enterprise** | 1,000 requests/usuario/día | - |
| **Unlimited** | 5,000 requests/usuario/día | - |
| **Developer** | 15,000 requests/org/día | - |

**Manejo de rate limiting**:
```python
def handle_rate_limit(response):
    if response.status_code == 403:
        error = response.json()[0]
        if error["errorCode"] == "REQUEST_LIMIT_EXCEEDED":
            # Esperar hasta el reset
            retry_after = int(response.headers.get("Retry-After", 3600))
            time.sleep(retry_after)
            return True
    return False
```

---

## 9. Ventajas y Desventajas

### Ventajas

✅ **RESTful estándar**: Fácil de integrar  
✅ **OAuth 2.0**: Autenticación segura  
✅ **SOQL potente**: Consultas complejas con relaciones  
✅ **Composite API**: Operaciones batch eficientes  
✅ **Versionado**: Compatibilidad hacia atrás  
✅ **Documentación excelente**: Ejemplos y referencias completas  

### Desventajas

❌ **Rate limiting estricto**: Límites bajos en licencias básicas  
❌ **Complejidad de objetos**: Muchos campos y relaciones  
❌ **Latencia**: Respuestas pueden ser lentas (200-500ms)  
❌ **Costo**: Licencias caras para alto volumen  
❌ **Curva de aprendizaje**: SOQL, metadata, permisos  

---

## 10. Recomendaciones para TravelHub

**1. Usar campos custom para mapeo**:
- `External_ID__c`: ID de TravelHub
- `Last_Sync_Date__c`: Última sincronización

**2. Implementar sincronización incremental**:
- Solo sincronizar registros modificados
- Usar `LastModifiedDate` para filtrar

**3. Caché local**:
- Cachear datos de Salesforce por 1 hora
- Reducir llamadas a API

**4. Webhooks (Salesforce → TravelHub)**:
- Usar Platform Events o Outbound Messages
- Notificaciones en tiempo real de cambios

**5. Monitoreo de rate limits**:
- Trackear uso diario de API
- Alertas al 80% del límite

---

Content was rephrased for compliance with licensing restrictions.

**Referencias:**
[1] Salesforce REST API Guide - https://manualzz.com/doc/o/lnueu/salesforce-rest-api-developer-guide-index
[2] Salesforce API Authentication - https://reintech.io/blog/authenticating-with-salesforce-api-best-practices
[3] Salesforce REST API Overview - https://hevodata.com/learn/salesforce-rest-api/
