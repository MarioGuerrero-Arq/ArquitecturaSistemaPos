# API Design Guidelines

## 📋 Propósito

Este documento establece los lineamientos estándar para el diseño, implementación y documentación de APIs RESTful en nuestros proyectos enterprise. Estos lineamientos aplican para:

- APIs internas entre microservicios
- APIs externas para integraciones con terceros
- APIs para aplicaciones frontend (web y móvil)
- APIs expuestas a través de API Gateway (Kong Enterprise)

**Objetivos:**
- Consistencia en el diseño de APIs across todos los servicios
- Facilitar el consumo por parte de equipos internos y externos
- Mejorar la mantenibilidad y evolución de las APIs
- Cumplir con estándares de seguridad enterprise (PCI-DSS, Zero-Trust)
- Habilitar observabilidad y monitoreo efectivo

---

## 🎯 Principios de Diseño

### 1. **API-First Design**
Diseña el contrato de la API antes de implementar la lógica de negocio. El contrato es el acuerdo entre el proveedor y el consumidor.

**Beneficios:**
- Permite desarrollo paralelo (frontend y backend)
- Facilita contract testing
- Documentación como código (OpenAPI/Swagger)

### 2. **Recursos, No Acciones**
Las APIs deben exponer recursos (sustantivos), no acciones (verbos).

**❌ Incorrecto:**
```
POST /getUsers
POST /createOrder
POST /deleteProduct/123
```

**✅ Correcto:**
```
GET /users
POST /orders
DELETE /products/123
```

### 3. **Consistencia**
Mantén consistencia en naming, estructura de respuestas y manejo de errores across todas las APIs.

### 4. **Evolución sin Rupturas**
Las APIs deben evolucionar sin romper clientes existentes. Usa versionado y deprecación controlada.

### 5. **Seguridad por Diseño**
Autenticación, autorización y validación son requisitos desde el día 1, no afterthoughts.

---

## 🌐 Convenciones de URLs y Recursos

### Estructura Base
```
https://api.dominio.com/{version}/{recurso}/{id}/{sub-recurso}
```

**Ejemplos:**
```
https://api.ERLN.com/v1/customers
https://api.ERLN.com/v1/customers/12345
https://api.ERLN.com/v1/customers/12345/orders
https://api.ERLN.com/v1/products/67890/inventory
```

### Naming Conventions

| Elemento | Convención | Ejemplo |
|----------|------------|---------|
| Recursos | Plural, lowercase, kebab-case | `/customers`, `/order-items` |
| IDs | Entero o UUID | `/customers/12345`, `/users/550e8400-e29b-41d4-a716-446655440000` |
| Query params | camelCase | `?pageSize=20&sortBy=createdAt` |
| Headers | Kebab-Case | `X-Request-ID`, `X-Correlation-ID` |

### Jerarquía de Recursos
Usa sub-recursos solo cuando hay relación de pertenencia clara.

**✅ Correcto:**
```
GET /customers/123/orders          # Órdenes del cliente 123
GET /orders/456/items              # Items de la orden 456
```

**❌ Evitar jerarquías profundas:**
```
GET /customers/123/orders/456/items/789  # Demasiado anidado
```

**Alternativa:** Usar query params para filtrar
```
GET /items?orderId=456&customerId=123
```

---

## 🔧 Métodos HTTP

### Mapeo de Operaciones CRUD

| Operación | Método HTTP | Ruta | Código de Éxito |
|-----------|-------------|------|-----------------|
| Crear | `POST` | `/resources` | `201 Created` |
| Leer (colección) | `GET` | `/resources` | `200 OK` |
| Leer (individual) | `GET` | `/resources/{id}` | `200 OK` |
| Actualizar (completo) | `PUT` | `/resources/{id}` | `200 OK` |
| Actualizar (parcial) | `PATCH` | `/resources/{id}` | `200 OK` |
| Eliminar | `DELETE` | `/resources/{id}` | `204 No Content` |

### Idempotencia

| Método | Idempotente | Seguro | Descripción |
|--------|-------------|--------|-------------|
| `GET` | ✅ Sí | ✅ Sí | Múltiples llamadas = mismo resultado |
| `PUT` | ✅ Sí | ❌ No | Múltiples llamadas = mismo resultado |
| `DELETE` | ✅ Sí | ❌ No | Múltiples llamadas = mismo resultado |
| `POST` | ❌ No | ❌ No | Cada llamada puede crear recurso diferente |
| `PATCH` | ❌ No* | ❌ No | Depende de la implementación |

*Nota: PATCH puede ser idempotente si se implementa correctamente.

### Métodos Adicionales

**HEAD:** Obtener metadatos de un recurso (headers) sin el body.
```
HEAD /products/123
```

**OPTIONS:** Obtener métodos soportados y CORS headers.
```
OPTIONS /products/123
```

---

## 📊 Códigos de Estado HTTP

### Códigos de Éxito (2xx)

| Código | Uso | Descripción |
|--------|-----|-------------|
| `200 OK` | GET, PUT, PATCH | Respuesta exitosa con body |
| `201 Created` | POST | Recurso creado exitosamente |
| `202 Accepted` | POST, PUT, DELETE | Operación aceptada pero no completada (async) |
| `204 No Content` | DELETE, PUT | Éxito sin body en respuesta |

### Códigos de Error del Cliente (4xx)

| Código | Uso | Descripción |
|--------|-----|-------------|
| `400 Bad Request` | Todos | Request malformado o datos inválidos |
| `401 Unauthorized` | Todos | Autenticación requerida o inválida |
| `403 Forbidden` | Todos | Autenticado pero sin permisos |
| `404 Not Found` | GET, PUT, DELETE | Recurso no existe |
| `405 Method Not Allowed` | Todos | Método HTTP no soportado |
| `409 Conflict` | POST, PUT | Conflicto con estado actual (duplicado) |
| `422 Unprocessable Entity` | POST, PUT, PATCH | Validación de negocio falló |
| `429 Too Many Requests` | Todos | Rate limit excedido |

### Códigos de Error del Servidor (5xx)

| Código | Uso | Descripción |
|--------|-----|-------------|
| `500 Internal Server Error` | Todos | Error inesperado del servidor |
| `502 Bad Gateway` | Todos | Error en servicio upstream |
| `503 Service Unavailable` | Todos | Servicio temporalmente no disponible |
| `504 Gateway Timeout` | Todos | Timeout en servicio upstream |

### Reglas Importantes

✅ **SIEMPRE** usa códigos de estado estándar HTTP
✅ **NUNCA** retornes `200 OK` con error en el body
✅ **NUNCA** retornes `500` para errores de validación (usa `400` o `422`)
✅ **SIEMPRE** incluye body descriptivo en errores 4xx y 5xx

---

## 🔢 Versionado de APIs

### Estrategia de Versionado

**Opción Recomendada: URL Path Versioning**
```
https://api.ERLN.com/v1/customers
https://api.ERLN.com/v2/customers
```

**Ventajas:**
- Fácil de entender y usar
- Compatible con API Gateway (Kong)
- Permite routing basado en versión
- Documentación clara por versión

**Alternativas (no recomendadas para nuestro caso):**
- Header versioning: `Accept: application/vnd.ERLN.v1+json`
- Query param: `?version=1`

### Política de Versionado

1. **Versión mayor (v1 → v2):** Cambios breaking (eliminar campos, cambiar tipos)
2. **Versión menor:** Cambios no-breaking (agregar campos opcionales)
3. **No versionar:** Cambios internos que no afectan contrato

### Deprecación de APIs

Cuando deprecas una versión:

1. **Anunciar con 6 meses de anticipación** mínimo
2. **Incluir header `Sunset`** en respuestas:
   ```
   Sunset: Sat, 31 Dec 2025 23:59:59 GMT
   Deprecation: true
   Link: <https://api.ERLN.com/v2/customers>; rel="successor-version"
   ```
3. **Monitorear uso** de versión deprecada
4. **Comunicar a consumidores** registrados
5. **Mantener documentación** accesible hasta fin de vida

---

## 🔐 Autenticación y Autorización

### Estándares de Autenticación

**Recomendado: OAuth 2.0 + OpenID Connect (OIDC)**

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Flujo recomendado:**
1. Cliente obtiene token de Authorization Server (Entra ID)
2. Cliente incluye token en header `Authorization`
3. API Gateway (Kong) valida token
4. API recibe claims del usuario en headers o JWT decodificado

### JWT (JSON Web Tokens)

**Estructura del JWT:**
```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "key-id-123"
  },
  "payload": {
    "sub": "user-123",
    "iss": "https://auth.ERLN.com",
    "aud": "api.ERLN.com",
    "exp": 1735689599,
    "iat": 1735685999,
    "scope": "read:orders write:orders",
    "roles": ["customer", "premium"]
  }
}
```

**Claims estándar:**
- `sub`: Subject (ID del usuario)
- `iss`: Issuer (Authorization Server)
- `aud`: Audience (API destino)
- `exp`: Expiration time
- `iat`: Issued at time
- `scope`: Permisos OAuth

### Autorización Basada en Roles (RBAC)

**Headers de autorización:**
```http
X-User-ID: user-123
X-User-Roles: customer,premium
X-User-Scopes: read:orders,write:orders
```

**Validación en código:**
```java
@PreAuthorize("hasRole('PREMIUM') and hasAuthority('write:orders')")
@PostMapping("/orders")
public Order createOrder(@RequestBody OrderRequest request) {
    // Solo usuarios premium con permiso write:orders
}
```

### Seguridad de Tokens

✅ **SIEMPRE** usa HTTPS (TLS 1.2+)
✅ **SIEMPRE** valida firma del JWT
✅ **SIEMPRE** verifica expiración (`exp`)
✅ **SIEMPRE** valida audience (`aud`)
✅ **NUNCA** almacenes tokens en localStorage (usa httpOnly cookies)
✅ **NUNCA** incluyas datos sensibles en JWT payload (es solo base64)

---

## ❌ Manejo de Errores

### Estructura Estándar de Respuesta de Error

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "One or more fields failed validation",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "value": "not-an-email"
      },
      {
        "field": "age",
        "message": "Must be greater than 0",
        "value": -5
      }
    ],
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-abc-123",
    "path": "/v1/customers"
  }
}
```

### Campos Obligatorios

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `code` | String | Código de error machine-readable (UPPER_SNAKE_CASE) |
| `message` | String | Mensaje humano legible |
| `timestamp` | String | Timestamp ISO 8601 |
| `requestId` | String | ID único del request (para tracing) |
| `path` | String | Ruta del endpoint |

### Campos Opcionales

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `details` | Array | Lista de errores específicos (validación) |
| `cause` | String | Causa raíz (solo en desarrollo) |
| `retryAfter` | Integer | Segundos para retry (429, 503) |

### Códigos de Error Estándar

```java
public enum ErrorCode {
    // Validación
    VALIDATION_ERROR,
    INVALID_FIELD,
    MISSING_REQUIRED_FIELD,
    
    // Negocio
    RESOURCE_NOT_FOUND,
    RESOURCE_ALREADY_EXISTS,
    INSUFFICIENT_PERMISSIONS,
    QUOTA_EXCEEDED,
    
    // Integración
    UPSTREAM_SERVICE_ERROR,
    UPSTREAM_TIMEOUT,
    
    // Sistema
    INTERNAL_ERROR,
    SERVICE_UNAVAILABLE
}
```

### Ejemplos de Respuestas de Error

**400 Bad Request (Validación):**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "value": "invalid-email"
      }
    ],
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-abc-123",
    "path": "/v1/customers"
  }
}
```

**404 Not Found:**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Customer with ID 99999 not found",
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-def-456",
    "path": "/v1/customers/99999"
  }
}
```

**429 Too Many Requests:**
```json
{
  "error": {
    "code": "QUOTA_EXCEEDED",
    "message": "Rate limit exceeded. Try again later",
    "retryAfter": 60,
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-ghi-789",
    "path": "/v1/customers"
  }
}
```

**500 Internal Server Error:**
```json
{
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "An unexpected error occurred",
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-jkl-012",
    "path": "/v1/customers"
  }
}
```

**NUNCA** expongas stack traces o detalles internos en producción.

---

## 📄 Paginación, Filtrado y Ordenamiento

### Paginación

**Estrategia recomendada: Cursor-based pagination** (para datasets grandes)

**Request:**
```http
GET /v1/orders?cursor=eyJpZCI6MTIzfQ&limit=20
```

**Response:**
```json
{
  "data": [
    { "id": 124, "total": 500.00 },
    { "id": 125, "total": 750.00 }
  ],
  "pagination": {
    "nextCursor": "eyJpZCI6MTI1fQ",
    "previousCursor": "eyJpZCI6MTI0fQ",
    "hasMore": true,
    "limit": 20
  }
}
```

**Alternativa: Offset-based pagination** (para datasets pequeños)

**Request:**
```http
GET /v1/orders?page=2&pageSize=20
```

**Response:**
```json
{
  "data": [
    { "id": 21, "total": 500.00 },
    { "id": 22, "total": 750.00 }
  ],
  "pagination": {
    "page": 2,
    "pageSize": 20,
    "totalPages": 50,
    "totalItems": 1000
  }
}
```

### Filtrado

**Query params para filtrar:**
```http
GET /v1/orders?status=pending&customerId=123&minTotal=100&maxTotal=1000
```

**Convenciones:**
- Usa nombres de campos en camelCase
- Para rangos: `min{Field}` y `max{Field}`
- Para múltiples valores: `status=pending,processing`
- Para fechas: formato ISO 8601 `createdAt=2025-01-01T00:00:00Z`

### Ordenamiento

**Query params para ordenar:**
```http
GET /v1/orders?sortBy=createdAt&sortOrder=desc
```

**Múltiples criterios:**
```http
GET /v1/orders?sortBy=createdAt,desc&sortBy=total,asc
```

**Campos permitidos:**
- Define whitelist de campos ordenables en documentación
- Valida campos permitidos en backend
- Default: `sortBy=id&sortOrder=asc`

---

## 🚦 Rate Limiting

### Headers de Rate Limiting

**Respuesta incluye headers:**
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1735689599
```

| Header | Descripción |
|--------|-------------|
| `X-RateLimit-Limit` | Límite total en ventana de tiempo |
| `X-RateLimit-Remaining` | Requests restantes en ventana actual |
| `X-RateLimit-Reset` | Timestamp Unix cuando se resetea ventana |

### Estrategias de Rate Limiting

**Por API Key:**
```
1000 requests/hour por API key
```

**Por IP:**
```
100 requests/minute por IP
```

**Por Usuario:**
```
500 requests/hour por usuario autenticado
```

**Por Endpoint:**
```
GET /products: 2000/hour
POST /orders: 100/hour
```

### Configuración en Kong Enterprise

```yaml
plugins:
  - name: rate-limiting
    config:
      minute: 100
      hour: 1000
      policy: redis
      redis_host: redis.ERLN.com
      limit_by: consumer
      fault_tolerant: true
```

### Respuesta 429 Too Many Requests

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
Retry-After: 60
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1735689599

{
  "error": {
    "code": "QUOTA_EXCEEDED",
    "message": "Rate limit exceeded. Try again in 60 seconds",
    "retryAfter": 60,
    "timestamp": "2025-01-15T14:30:00Z",
    "requestId": "req-abc-123",
    "path": "/v1/customers"
  }
}
```

---

## 📖 Documentación (OpenAPI/Swagger)

### Obligatorio para Todas las APIs

**Requisitos:**
1. Archivo OpenAPI 3.0+ en formato YAML o JSON
2. Documentación interactiva con Swagger UI
3. Ejemplos de request/response para cada endpoint
4. Esquemas de validación completos
5. Descripción de códigos de error

### Estructura del Archivo OpenAPI

```yaml
openapi: 3.0.3
info:
  title: Customer API
  version: 1.0.0
  description: API para gestión de clientes
  contact:
    name: API Support
    email: api-support@ERLN.com
  license:
    name: Proprietary

servers:
  - url: https://api.ERLN.com/v1
    description: Production
  - url: https://api-staging.ERLN.com/v1
    description: Staging

paths:
  /customers:
    get:
      summary: List customers
      operationId: listCustomers
      tags:
        - Customers
      parameters:
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: pageSize
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/CustomerListResponse'
              example:
                data:
                  - id: 123
                    name: "John Doe"
                    email: "john@example.com"
                pagination:
                  page: 1
                  pageSize: 20
                  totalItems: 100
        '401':
          $ref: '#/components/responses/Unauthorized'
        '429':
          $ref: '#/components/responses/RateLimited'

components:
  schemas:
    Customer:
      type: object
      required:
        - id
        - name
        - email
      properties:
        id:
          type: integer
          example: 123
        name:
          type: string
          example: "John Doe"
        email:
          type: string
          format: email
          example: "john@example.com"
        createdAt:
          type: string
          format: date-time
          
  responses:
    Unauthorized:
      description: Authentication required
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
    RateLimited:
      description: Rate limit exceeded
      content:
        application/json:
          schema:
            $ref: '#/components/schemas/Error'
```

### Herramientas Recomendadas

**Generación de documentación:**
- **Springdoc OpenAPI** (para Spring Boot)
- **Swashbuckle** (para .NET)
- **OpenAPI Generator** (para generar clientes/servidores)

**Validación:**
- **Swagger Editor** (online)
- **OpenAPI CLI** (local)
- **Spectral** (linting de OpenAPI)

**Hosting:**
- **Swagger UI** (self-hosted)
- **Redoc** (documentación más elegante)
- **Kong Dev Portal** (integrado con API Gateway)

---

## 🔒 Seguridad

### Validación de Input

**SIEMPRE** valida input en backend, nunca confíes en validación frontend.

```java
@PostMapping("/customers")
public ResponseEntity<Customer> createCustomer(
    @Valid @RequestBody CreateCustomerRequest request) {
    // @Valid触发Bean Validation
    // Validación adicional de negocio aquí
}

public class CreateCustomerRequest {
    @NotBlank
    @Size(min = 2, max = 100)
    private String name;
    
    @NotBlank
    @Email
    private String email;
    
    @Positive
    private Integer age;
}
```

### Prevención de Inyección

**SQL Injection:**
- ✅ Usa prepared statements / parameterized queries
- ✅ Usa ORM (Hibernate, Entity Framework)
- ❌ NUNCA concatenes strings para construir queries

**NoSQL Injection:**
- ✅ Valida y sanitiza input
- ✅ Usa drivers con parameterization
- ❌ NUNCA pases input directo a queries

**Command Injection:**
- ✅ Evita ejecutar comandos del sistema
- ✅ Si es necesario, usa whitelisting de comandos
- ❌ NUNCA pases input a `Runtime.exec()` o similar

### Protección contra ataques comunes

**XSS (Cross-Site Scripting):**
- ✅ Escapa output en respuestas
- ✅ Usa Content-Security-Policy headers
- ✅ Valida y sanitiza input

**CSRF (Cross-Site Request Forgery):**
- ✅ Usa tokens CSRF para operaciones state-changing
- ✅ Valida Origin/Referer headers
- ✅ Usa SameSite cookies

**Mass Assignment:**
- ✅ Usa DTOs (Data Transfer Objects)
- ✅ Define whitelist de campos actualizables
- ❌ NUNCA bindees request directo a entidad de base de datos

### Headers de Seguridad

**Headers obligatorios en todas las respuestas:**
```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

### Datos Sensibles

**NUNCA** incluyas en logs o respuestas:
- ❌ Contraseñas
- ❌ Números de tarjeta de crédito (PCI-DSS)
- ❌ Tokens de sesión
- ❌ Información personal sensible (PII) sin encriptación

**SIEMPRE** encripta:
- ✅ Datos de tarjeta (PCI-DSS)
- ✅ Información personal sensible
- ✅ Credentials en tránsito (TLS) y en reposo

---

## ⚡ Performance y Caching

### HTTP Caching

**Cache-Control headers:**

**Recursos estáticos (imágenes, CSS, JS):**
```http
Cache-Control: public, max-age=31536000, immutable
```

**API responses (datos dinámicos):**
```http
Cache-Control: private, no-cache, no-store, must-revalidate
```

**API responses cacheables:**
```http
Cache-Control: public, max-age=300
ETag: "abc123"
Last-Modified: Wed, 15 Jan 2025 10:30:00 GMT
```

### ETag y Conditional Requests

**Request:**
```http
GET /v1/products/123
If-None-Match: "abc123"
```

**Response (304 Not Modified):**
```http
HTTP/1.1 304 Not Modified
ETag: "abc123"
```

**Beneficio:** Reduce bandwidth y mejora performance

### Compression

**Habilita gzip/br compression:**
```http
Accept-Encoding: gzip, deflate, br
Content-Encoding: gzip
```

**Configuración en Kong:**
```yaml
plugins:
  - name: compression
    config:
      level: 5
      types:
        - application/json
        - application/xml
        - text/plain
```

### Timeouts

**Timeouts recomendados:**

| Tipo | Timeout | Descripción |
|------|---------|-------------|
| Connection | 5s | Tiempo para establecer conexión |
| Read | 30s | Tiempo para recibir respuesta |
| Write | 30s | Tiempo para enviar request |

**Configuración en cliente HTTP:**
```java
HttpClient client = HttpClient.newBuilder()
    .connectTimeout(Duration.ofSeconds(5))
    .build();

HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://api.ERLN.com/v1/customers"))
    .timeout(Duration.ofSeconds(30))
    .GET()
    .build();
```

---

## 🧪 Testing de APIs

### Niveles de Testing

**1. Unit Tests:**
- Test lógica de negocio aislada
- Mock dependencias externas
- Cobertura objetivo: 80%+

**2. Integration Tests:**
- Test integración entre componentes
- Usa base de datos en memoria (H2)
- Test endpoints con requests reales

**3. Contract Tests:**
- Valida que API cumple contrato OpenAPI
- Usa Pact o Spring Cloud Contract
- Previene breaking changes

**4. End-to-End Tests:**
- Test flujo completo (frontend → backend → DB)
- Usa ambiente de staging
- Test casos de uso reales

### Herramientas Recomendadas

**Testing manual:**
- **Postman** - Colecciones de tests
- **Insomnia** - Alternativa a Postman
- **curl** - Testing desde terminal

**Testing automatizado:**
- **JUnit 5** (Java) / **xUnit** (.NET) - Unit tests
- **RestAssured** (Java) - API testing
- **WireMock** - Mock de servicios externos
- **Testcontainers** - Bases de datos en Docker

**Performance testing:**
- **JMeter** - Load testing
- **Gatling** - Performance testing con código
- **k6** - Load testing moderno

**Security testing:**
- **OWASP ZAP** - Security scanning
- **Burp Suite** - Penetration testing
- **Snyk** - Dependency vulnerability scanning

### Ejemplo de Test con RestAssured

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class CustomerApiTest {
    
    @LocalServerPort
    private int port;
    
    @BeforeEach
    void setUp() {
        RestAssured.port = port;
        RestAssured.basePath = "/v1";
    }
    
    @Test
    void shouldCreateCustomer() {
        given()
            .contentType(ContentType.JSON)
            .body("""
                {
                    "name": "John Doe",
                    "email": "john@example.com"
                }
                """)
        .when()
            .post("/customers")
        .then()
            .statusCode(201)
            .body("id", notNullValue())
            .body("name", equalTo("John Doe"))
            .body("email", equalTo("john@example.com"));
    }
    
    @Test
    void shouldReturn404WhenCustomerNotFound() {
        given()
            .pathParam("id", 99999)
        .when()
            .get("/customers/{id}")
        .then()
            .statusCode(404)
            .body("error.code", equalTo("RESOURCE_NOT_FOUND"));
    }
}
```

---

## 📊 Monitoreo y Observabilidad

### Métricas Obligatorias

**Métricas de negocio:**
- Requests por endpoint
- Tiempo de respuesta (p50, p95, p99)
- Tasa de errores (4xx, 5xx)
- Throughput (requests/second)

**Métricas de sistema:**
- CPU usage
- Memory usage
- Database connections
- Thread pool usage

**Métricas de negocio específicas:**
- Órdenes creadas por hora
- Clientes registrados por día
- Conversión de carrito

### Logging Estructurado

**Formato JSON obligatorio:**
```json
{
  "timestamp": "2025-01-15T14:30:00.123Z",
  "level": "INFO",
  "logger": "com.ERLN.api.CustomerController",
  "message": "Customer created successfully",
  "traceId": "abc123def456",
  "spanId": "span789",
  "userId": "user-123",
  "requestId": "req-abc-123",
  "method": "POST",
  "path": "/v1/customers",
  "status": 201,
  "duration": 45
}
```

**Campos obligatorios:**
- `timestamp` - ISO 8601
- `level` - DEBUG, INFO, WARN, ERROR
- `message` - Mensaje descriptivo
- `traceId` - Para distributed tracing
- `requestId` - ID único del request

**NUNCA** loggees:
- ❌ Datos sensibles (passwords, tokens, PII)
- ❌ Stack traces completos en producción (solo mensajes)
- ❌ Queries SQL con datos sensibles

### Distributed Tracing

**Implementación con OpenTelemetry:**

```java
@Configuration
public class TracingConfig {
    
    @Bean
    public OpenTelemetry openTelemetry() {
        return OpenTelemetrySdk.builder()
            .setTracerProvider(
                SdkTracerProvider.builder()
                    .addSpanProcessor(
                        BatchSpanProcessor.builder(
                            OtlpGrpcSpanExporter.builder()
                                .setEndpoint("http://jaeger:4317")
                                .build())
                    .build())
                .build())
            .build();
    }
}
```

**Headers de tracing:**
```http
X-Trace-ID: abc123def456
X-Span-ID: span789
X-Parent-Span-ID: parent123
```

### Alertas

**Alertas críticas (P1):**
- Error rate > 5% por 5 minutos
- Latencia p99 > 5s por 5 minutos
- Servicio caído (health check falla)

**Alertas de warning (P2):**
- Error rate > 1% por 15 minutos
- Latencia p95 > 2s por 15 minutos
- Uso de CPU > 80% por 30 minutos

**Integración:**
- **ServiceNow** - Ticketing automático
- **PagerDuty** - On-call rotation
- **Slack/Teams** - Notificaciones en canal

---

## ✅ Checklist de Revisión de API

Antes de desplegar una API, verifica:

### Diseño
- [ ] URLs siguen convenciones (plural, lowercase, kebab-case)
- [ ] Métodos HTTP usados correctamente
- [ ] Códigos de estado HTTP estándar
- [ ] Versionado implementado
- [ ] Paginación, filtrado y ordenamiento implementados

### Seguridad
- [ ] Autenticación obligatoria (OAuth2/OIDC)
- [ ] Autorización basada en roles (RBAC)
- [ ] Validación de input en backend
- [ ] Headers de seguridad incluidos
- [ ] Datos sensibles encriptados
- [ ] Rate limiting configurado

### Documentación
- [ ] Archivo OpenAPI 3.0+ completo
- [ ] Ejemplos de request/response
- [ ] Códigos de error documentados
- [ ] Swagger UI accesible
- [ ] Changelog de versiones

### Testing
- [ ] Unit tests (cobertura 80%+)
- [ ] Integration tests
- [ ] Contract tests
- [ ] Performance tests (si aplica)
- [ ] Security scan (OWASP ZAP)

### Monitoreo
- [ ] Métricas configuradas (Prometheus)
- [ ] Logging estructurado (JSON)
- [ ] Distributed tracing (OpenTelemetry)
- [ ] Alertas configuradas
- [ ] Dashboards en Grafana

### Performance
- [ ] Timeouts configurados
- [ ] Caching habilitado (si aplica)
- [ ] Compression habilitada (gzip/br)
- [ ] Connection pooling configurado
- [ ] Database queries optimizadas

### Deploy
- [ ] Health check endpoint (`/health`)
- [ ] Graceful shutdown implementado
- [ ] Circuit breakers configurados
- [ ] Retry policies configuradas
- [ ] Fallbacks implementados

---

## 📚 Referencias

### Estándares
- [RFC 7231 - HTTP/1.1 Semantics and Content](https://tools.ietf.org/html/rfc7231)
- [RFC 8259 - The JavaScript Object Notation (JSON) Data Interchange Format](https://tools.ietf.org/html/rfc8259)
- [OpenAPI Specification 3.0](https://swagger.io/specification/)
- [JSON API Specification](https://jsonapi.org/)

### Guías de Estilo
- [Microsoft REST API Guidelines](https://github.com/microsoft/api-guidelines)
- [Google API Design Guide](https://cloud.google.com/apis/design)
- [Zalando RESTful API and Event Scheme Guidelines](https://opensource.zalando.com/restful-api-guidelines/)

### Seguridad
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [PCI-DSS Requirements](https://www.pcisecuritystandards.org/)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

### Libros
- "RESTful Web APIs" - Leonard Richardson, Mike Amundsen, Sam Ruby
- "API Design Patterns" - JJ Geewax
- "Build APIs You Won't Hate" - Phil Sturgeon

---

## 📞 Contacto y Soporte

**API Guild:**
- Email: api-guild@ERLN.com
- Slack: #api-guild
- Reuniones: Quincenal (miércoles 10:00 AM)

**Revisiones de API:**
- Agenda revisión con API Guild antes de desplegar
- Checklist de revisión obligatorio
- Aprobación requerida para APIs públicas

---

**Última actualización:** 2025-01-15  
**Versión:** 1.0  
**Mantenido por:** API Guild  
**Aprobado por:** Architecture Review Board