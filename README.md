# Especificación general del API REST para MDT Tahoe

**Versión:** 1.0  
**Estado:** Borrador  
**Formato:** REST sobre HTTPS  
**Codificación:** UTF-8  
**Tipo de contenido:** `application/json`

---

## 1. Objetivo

Este documento define las reglas generales para consumir un API REST destinado a integraciones servidor a servidor.


## 2. Ambientes

### 2.1 Desarrollo

```text
https://api-dev.example.com/v1
```

Características:

- Accesible desde Internet.
- Requiere token de Firebase enviado como Bearer.
- No debe contener datos reales o sensibles.
- No tiene límites de consumo pero el servidor tiene recursos limitados.

### 2.2 Producción

```text
https://api.example.com/v1
```

Características:

- Solo acepta solicitudes provenientes de direcciones IP previamente autorizadas.
- Requiere token de Firebase enviado como Bearer.
- Tiene límites de consumo por token

---

## 3. Convenciones generales

### 3.1 Protocolo

Todas las solicitudes deberán realizarse mediante HTTPS.

Las solicitudes recibidas por HTTP deberán ser rechazadas o redirigidas a HTTPS, según la política de infraestructura.

### 3.2 Formato de datos

Todas las solicitudes que contengan un cuerpo deberán usar:

```http
Content-Type: application/json; charset=utf-8
```

El cliente deberá indicar que espera respuestas JSON:

```http
Accept: application/json
```

### 3.3 Codificación

Los documentos JSON deberán estar codificados en UTF-8.

### 3.4 URL base

La URL general de un recurso tendrá la siguiente forma:

```text
{base_url}/{version}/{resource}
```

Ejemplo:

```text
https://api.example.com/v1/getTickets
```

### 3.5 Métodos HTTP

| Método | Uso |
|---|---|
| `GET` | Consultar uno o más recursos |
| `POST` | Crear un recurso o ejecutar una operación |

---

## 4. Autenticación y autorización



## 4.1 Bearer token firmado

El cliente deberá incluir el token en el encabezado:

```http
Authorization: Bearer {token}
```

En donde el token es el de Firebase


### 4.1.1 Validaciones del servidor

El servidor deberá:

1. Verificar la autenticidad del token.
2. validar que el consumidor tenga permiso para usar el endpoint.


---

## 5. Encabezados de solicitud

### 5.1 Encabezados obligatorios

```http
Authorization: Bearer {token}
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-Id: {uuid}
```

### 5.2 Encabezados recomendados

```http
X-API-Version: 1
User-Agent: EthicsCodeTahoe/1.0
```

| Encabezado | Descripción |
|---|---|
| `Authorization` | Bearer token firmado |
| `Content-Type` | Formato del cuerpo enviado |
| `Accept` | Formato esperado de respuesta |
| `X-Request-Id` | UUID único para trazabilidad |
| `X-API-Version` | Versión solicitada del contrato |
| `User-Agent` | Identificación informativa del cliente |

`User-Agent` no deberá utilizarse como mecanismo de autenticación.

---

## 6. Formato de las solicitudes

### 6.1 Solicitud POST

```http
POST /v1/incidents HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJjbGllbnRJZCI6...gYv3Q
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-Id: 930938b4-85cb-43aa-8c80-b0a3de401298
```

```json
{
  "externalId": "EXT-90001",
  "type": "emergency",
  "description": "Descripción del incidente"
}
```
(Esto es solo un ejemplo)

### 6.2 Solicitud GET

```http
GET /v1/incidents/INC-10025 HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJjbGllbnRJZCI6...BRx9w
Accept: application/json
X-Request-Id: a35de62e-f642-4773-bde5-6c179244a3a5
```

### 6.3 Parámetros de consulta

Los parámetros deberán enviarse en la URL:

```text
GET /v1/incidents?status=open&page=1&pageSize=50
```

---

## 7. Formato general de respuestas

Todas las respuestas, exitosas o no, deberán ser documentos JSON.

La estructura general será:

```json
{
  "code": 200,
  "data": {}
}
```

| Campo | Tipo | Obligatorio | Descripción |
|---|---:|---:|---|
| `code` | integer | Sí | Resultado lógico: `200` o `401` |
| `data` | cualquier JSON | No | Contenido propio de la respuesta |

Valores permitidos:

| `code` | Significado |
|---:|---|
| `200` | Solicitud procesada correctamente |
| `401` | Solicitud rechazada o no autorizada |

El campo `data` puede contener:

- un objeto;
- una colección;
- un valor escalar;
- información complementaria de la operación.

El campo `data` deberá omitirse cuando no exista contenido que devolver.

---

## 8. Respuestas exitosas

### 8.1 Objeto individual

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
X-Request-Id: a35de62e-f642-4773-bde5-6c179244a3a5
```

```json
{
  "code": 200,
  "data": {
    "id": "INC-10025",
    "status": "open",
    "createdAt": "2026-07-22T13:30:00Z"
  }
}
```

### 8.2 Colección

```json
{
  "code": 200,
  "data": {
    "items": [
      {
        "id": "INC-10025",
        "status": "open"
      },
      {
        "id": "INC-10026",
        "status": "closed"
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 50,
      "totalItems": 2,
      "totalPages": 1
    }
  }
}
```

### 8.3 Operación sin contenido adicional

```json
{
  "code": 200
}
```

### 8.4 Recurso creado

Cuando se cree un recurso, se recomienda responder HTTP `200` para conservar el contrato indicado:

```http
HTTP/1.1 200 OK
```

```json
{
  "code": 200,
  "data": {
    "id": "INC-10027"
  }
}
```

---

## 9. Respuestas de rechazo

Todas las fallas de autenticación, autorización, validación o procesamiento se expresarán mediante:

```json
{
  "code": 401
}
```

Opcionalmente, `data` podrá contener un mensaje general no sensible:

```json
{
  "code": 401,
  "data": {
    "message": "Request could not be authorized",
    "requestId": "930938b4-85cb-43aa-8c80-b0a3de401298"
  }
}
```

No deberán exponerse detalles como:

- llave inexistente;
- firma esperada;
- dirección IP requerida;
- nombres de tablas;
- trazas de ejecución;
- excepciones;
- configuración interna;
- existencia o inexistencia de registros sensibles.

### 9.1 Causas posibles de `401`

- dirección IP no autorizada;
- encabezado `Authorization` ausente;
- formato de token inválido;
- llave revocada;
- firma incorrecta;
- JSON inválido;
- consumidor sin permiso para el recurso;
- solicitud inválida conforme a las reglas del endpoint;
- error interno que no deba revelarse al consumidor.

401 significa que no se confía en el dispositivo que hace el request. La aplicación debería indicarlo al usuario y cerrar su sesión.

---

## 10. Relación entre HTTP y el campo `code`

El API deberá utilizar preferentemente la siguiente correspondencia:

| Resultado | HTTP | JSON `code` |
|---|---:|---:|
| Operación exitosa | `200` | `200` |
| Solicitud rechazada | `401` | `401` |

Ejemplo de rechazo:

```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json; charset=utf-8
Cache-Control: no-store
X-Request-Id: 930938b4-85cb-43aa-8c80-b0a3de401298
```

```json
{
  "code": 401
}
```

No se deberá devolver HTML, incluso ante errores producidos por proxies, gateways o servidores web.

---

## 11. Fechas y horas

Todas las fechas incluidas en `data` deberán usar ISO 8601 en UTC:

```text
2026-07-22T13:30:00Z
```

---

## 12. Identificadores

Se recomienda utilizar identificadores opacos, UUID o códigos no secuenciales:

```text
INC-10025
930938b4-85cb-43aa-8c80-b0a3de401298
```

Los identificadores enviados por el cliente deberán validarse y no deberán conceder acceso a un recurso sin comprobar previamente la autorización del consumidor.

---

## 13. Paginación

Los endpoints que devuelvan colecciones deberán aceptar:

| Parámetro | Tipo | Valor predeterminado |
|---|---:|---:|
| `page` | integer | `1` |
| `pageSize` | integer | `50` |

Se recomienda establecer un máximo de `100` elementos por página.

Ejemplo:

```text
GET /v1/incidents?page=1&pageSize=50
```

Respuesta:

```json
{
  "code": 200,
  "data": {
    "items": [],
    "pagination": {
      "page": 1,
      "pageSize": 50,
      "totalItems": 0,
      "totalPages": 0
    }
  }
}
```

---

## 14. Idempotencia

Las operaciones `POST` que puedan repetirse por reintentos deberán aceptar:

```http
Idempotency-Key: 00b6581a-f7a6-4f64-98f1-fb7a962d9c48
```

El servidor deberá asociar la llave con:

- consumidor;
- endpoint;
- resultado original;
- periodo de retención.

Si se recibe nuevamente la misma llave con un cuerpo diferente, la solicitud deberá rechazarse con 401. 

---

## 15. Límites de consumo

El API podrá aplicar límites por:

- dirección IP;
- `numero de teléfono`;
- endpoint;
- unidad de tiempo;
- tamaño de payload;
- número de solicitudes concurrentes.

Aun cuando el contrato público utilice solamente `200` y `401`, se recomienda incluir encabezados informativos:

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 998
X-RateLimit-Reset: 1784727060
```

No deberá utilizarse el Bearer token como sustituto de límites de consumo.

---

## 16. Tamaño de las solicitudes

Salvo que un endpoint indique otra cosa:

```text
Tamaño máximo recomendado del cuerpo: 2 MiB
```

---

## 17. Versionamiento

La versión principal deberá formar parte de la URL:

```text
/v1
```

Los cambios incompatibles deberán publicarse bajo una nueva versión:

```text
/v2
```

Los cambios compatibles podrán agregarse sin modificar la versión, siempre que:

- no se eliminen campos existentes;
- no se cambie el tipo de un campo;
- no se vuelva obligatorio un campo previamente opcional;
- los clientes toleren propiedades adicionales.

---

## 18. Rotación y revocación de llaves

Pendiente: acordar la política de rotación de llaves
---

## 19. Auditoría

El servidor deberá registrar como mínimo:

- fecha y hora UTC;
- `Número de teléfono`;
- dirección IP observada;
- método;
- ruta;
- resultado;
- `X-Request-Id`;
- identificador interno de operación;
- duración;
- causa interna del rechazo;
- hash o referencia del nonce.

Nunca deberán registrarse:

- llaves secretas;
- tokens completos;
- datos personales innecesarios;
- cuerpos sensibles completos, salvo política expresa y protegida.

---


## 20. TBD

---


## 21. Requisitos mínimos de seguridad

- HTTPS obligatorio.
- TLS 1.2 o superior.
- Bearer token Firebase Token.
- Rotación y revocación de llaves.
- Rate limiting.
- Registros de auditoría.
- Respuestas JSON uniformes.
- Ningún secreto dentro de repositorios o archivos de configuración no protegidos.
- Protección contra repetición de solicitudes.
- Validación estricta de todos los datos recibidos.

---

## 22. Referencia de contrato

### Solicitud general

```http
{METHOD} {PATH} HTTP/1.1
Host: {HOST}
Authorization: Bearer {SIGNED_TOKEN}
Content-Type: application/json; charset=utf-8
Accept: application/json
X-Request-Id: {UUID}
```

```json
{
  "...": "payload definido por cada endpoint"
}
```

### Respuesta exitosa general

```json
{
  "code": 200,
  "data": {
    "...": "contenido de la respuesta"
  }
}
```

### Respuesta exitosa sin contenido

```json
{
  "code": 200
}
```

### Respuesta de rechazo general

```json
{
  "code": 401
}
```

---

## 23. Observaciones de implementación

El esquema de firma descrito sigue el patrón general de integraciones que utilizan un identificador público y una llave pública para verificar la vigencia del token. No debe considerarse compatible automáticamente con una implementación específica de terceros.

Antes de implementar el API, ambas partes deberán aprobar de manera explícita:

- reglas exactas de JSON canónico;
- inclusión y orden de parámetros de consulta;
- representación de cuerpos vacíos;
- algoritmo y formato de salida;
- ventana de tiempo;
- formato de Base64 URL-safe;
- manejo de caracteres Unicode;
- procedimiento de rotación de llaves;
- endpoints y permisos asignados a cada consumidor.

# 24. Métodos del API

## 24.1 Validate number
Permite validar que el teléfono del usuario está registrado como respondiente. 

```text
{base_url}/{version}/validateNumber
```

Método: POST

Payload:
```json
{
  "phone_number": "+524454455454"
}
```

Respuesta: 
```json
{
  "code": 200
}
```

| `code` | Significado |
|---:|---|
| `200` | Teléfono registrado |
| `401` | Teléfono no registrado o bloqueado o token Firebase inválido |

## 24.2 Acknowledge Message
El dispositivo acusa recibo de mensaje enviado por FCM (Ver especificación de mensajes). Cada mensaje es identificado con un message_id que es un uuid enviado por el backend. 

```text
{base_url}/{version}/ack
```

Método: POST

Payload:
```json
{
  "message_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479"
}
```

Respuesta: 
```json
{
  "code": 200
}
```

| `code` | Significado |
|---:|---|
| `200` | Ack recibido |
| `401` | Teléfono no registrado o bloqueado o token Firebase inválido |

## 24.3 Get ticket list
Regresa la lista de incidentes asignados al dispositivo.

```text
{base_url}/{version}/getTicketList
```

Método: POST

Payload:
```json
{
}
```

Respuesta: 
```json
{
  "code": 200,
  "data": {
    "tickets": [
      {
        "id": "12322443322",
        "description": "Incendio con personas atrapadas en Mercado Central de San José",
        "priority": 4,
        "lat": 9.232332334,
        "lng": -8.2233022922,
        "state": "Asignado",
        "created": "2026-07-22T16:09:11Z",
        "summary": "Acudir a sitio para apoyo a bomberos",
        "tasks": [
          {
            "id": 2332000221,
            "description": "Caso creado y despachado por central",
            "created": "2026-07-22T16:09:11Z",
            "generator": "911 San José"
          },
          {
            "id": 2332000223,
            "description": "Despacho generado por IA",
            "created": "2026-07-22T17:09:11Z",
            "generator": "IA"
          }          
        ],
        "selectableStates": [
          {
            "id": 10,
            "name": "En Ruta"
          },
          {
            "id": 20,
            "name": "En traslado"
          },
          {
            "id": 70,
            "name": "En Sitio"
          }
        ]
      }
    ]
  }
}
```

| `priority` | Significado |
|---:|---|
| `1` | Nula |
| `2` | Baja |
| `3` | Media |
| `4` | Alta |
| `5` | Crítica |



| `code` | Significado |
|---:|---|
| `200` | Ack recibido |
| `401` | Teléfono no registrado o bloqueado o token Firebase inválido |

Nota: la operativa normal sugiere que una unidad no debe tener más de un ticket asignado, así que la lista de tickets solo debe tener cero o un ticket, pero no queremos restringirlo en el API por si cambian las condiciones de uso. Cada ticket es "autocontenido" y muestra las taras visibles para cada MDT, para cada ticket.

## 24.4 Update location
Actualiza la ubicación del dispositivo

```text
{base_url}/{version}/updateLocation
```

Método: POST

Payload:
```json
{
  "lat": 19.2233244,
  "lng": -99.22301992,
  "epoch": 1784740445,
  "accuracy": 4
}
```

Respuesta: 
```json
{
  "code": 200
}


