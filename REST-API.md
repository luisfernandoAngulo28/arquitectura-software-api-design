# API REST - Documentación de Endpoints

## Base URL
```
https://api.telocompro.com/v1
```

## Autenticación
Todas las peticiones requieren un token JWT en el header:
```
Authorization: Bearer {token}
```

---

## Endpoints

### 1. Productos

#### GET /products
Obtiene la lista de productos

**Parámetros de Query:**
- `page` (opcional): Número de página (default: 1)
- `limit` (opcional): Cantidad de resultados (default: 20)
- `category` (opcional): Filtrar por categoría

**Respuesta exitosa (200 OK):**
```json
{
  "data": [
    {
      "id": "prod_123",
      "name": "Coca Cola 2L",
      "price": 12.50,
      "stock": 100,
      "category": "beverages"
    }
  ],
  "pagination": {
    "page": 1,
    "total_pages": 5,
    "total_items": 100
  }
}
```

---

#### GET /products/:id
Obtiene un producto específico

**Parámetros de Ruta:**
- `id`: ID del producto

**Respuesta exitosa (200 OK):**
```json
{
  "id": "prod_123",
  "name": "Coca Cola 2L",
  "description": "Bebida gaseosa de 2 litros",
  "price": 12.50,
  "stock": 100,
  "category": "beverages",
  "images": ["url1", "url2"]
}
```

**Respuesta de error (404 Not Found):**
```json
{
  "error": "Product not found",
  "message": "No se encontró el producto con ID prod_123"
}
```

---

#### POST /products
Crea un nuevo producto (Admin only)

**Headers requeridos:**
```
Authorization: Bearer {admin_token}
Content-Type: application/json
```

**Body:**
```json
{
  "name": "Producto Nuevo",
  "description": "Descripción del producto",
  "price": 25.00,
  "stock": 50,
  "category": "electronics"
}
```

**Respuesta exitosa (201 Created):**
```json
{
  "id": "prod_124",
  "name": "Producto Nuevo",
  "created_at": "2026-02-11T10:00:00Z"
}
```

---

#### PUT /products/:id
Actualiza un producto existente (Admin only)

**Body:**
```json
{
  "price": 27.50,
  "stock": 45
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "prod_124",
  "name": "Producto Nuevo",
  "price": 27.50,
  "stock": 45,
  "updated_at": "2026-02-11T11:00:00Z"
}
```

---

#### DELETE /products/:id
Elimina un producto (Admin only)

**Respuesta exitosa (200 OK):**
```json
{
  "message": "Producto eliminado exitosamente",
  "id": "prod_124"
}
```

---

### 2. Carrito de Compras

#### GET /cart
Obtiene el carrito del usuario autenticado

**Respuesta exitosa (200 OK):**
```json
{
  "id": "cart_456",
  "user_id": "user_789",
  "items": [
    {
      "product_id": "prod_123",
      "quantity": 2,
      "unit_price": 12.50,
      "subtotal": 25.00
    }
  ],
  "total": 25.00
}
```

---

#### POST /cart/items
Agrega un item al carrito

**Body:**
```json
{
  "product_id": "prod_123",
  "quantity": 2
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "message": "Item agregado al carrito",
  "cart_id": "cart_456",
  "item_count": 3
}
```

**Respuesta de error (400 Bad Request):**
```json
{
  "error": "Insufficient stock",
  "message": "Stock disponible: 1, solicitado: 2"
}
```

---

#### PUT /cart/items/:product_id
Actualiza la cantidad de un item en el carrito

**Body:**
```json
{
  "quantity": 5
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "message": "Cantidad actualizada",
  "item": {
    "product_id": "prod_123",
    "quantity": 5,
    "subtotal": 62.50
  }
}
```

---

#### DELETE /cart/items/:product_id
Elimina un item del carrito

**Respuesta exitosa (200 OK):**
```json
{
  "message": "Item eliminado del carrito"
}
```

---

#### DELETE /cart
Vacía el carrito completo

**Respuesta exitosa (200 OK):**
```json
{
  "message": "Carrito vaciado exitosamente"
}
```

---

### 3. Órdenes

#### GET /orders
Obtiene las órdenes del usuario autenticado

**Parámetros de Query:**
- `status` (opcional): Filtrar por estado (pending, confirmed, processing, shipped, delivered, cancelled)
- `page` (opcional): Número de página
- `limit` (opcional): Cantidad de resultados

**Respuesta exitosa (200 OK):**
```json
{
  "data": [
    {
      "id": "order_555",
      "status": "delivered",
      "total": 37.50,
      "created_at": "2026-02-11T10:30:00Z",
      "delivered_at": "2026-02-13T14:20:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "total_pages": 2,
    "total_items": 15
  }
}
```

---

#### GET /orders/:id
Obtiene el detalle de una orden específica

**Respuesta exitosa (200 OK):**
```json
{
  "id": "order_555",
  "status": "delivered",
  "items": [
    {
      "product_id": "prod_123",
      "product_name": "Coca Cola 2L",
      "quantity": 3,
      "unit_price": 12.50,
      "subtotal": 37.50
    }
  ],
  "total": 37.50,
  "delivery_address": {
    "street": "Av. Arce 123",
    "city": "La Paz",
    "phone": "77777777"
  },
  "payment_method": "credit_card",
  "created_at": "2026-02-11T10:30:00Z",
  "delivered_at": "2026-02-13T14:20:00Z"
}
```

---

#### POST /orders
Crea una nueva orden

**Body:**
```json
{
  "delivery_address_id": "addr_111",
  "payment_method": "credit_card",
  "notes": "Tocar el timbre"
}
```

**Respuesta exitosa (201 Created):**
```json
{
  "id": "order_555",
  "status": "pending",
  "total": 37.50,
  "created_at": "2026-02-11T10:30:00Z",
  "estimated_delivery": "2026-02-14T18:00:00Z"
}
```

**Respuesta de error (400 Bad Request):**
```json
{
  "error": "Empty cart",
  "message": "El carrito está vacío. Agrega productos antes de crear una orden."
}
```

---

#### PATCH /orders/:id/cancel
Cancela una orden (solo si está en estado 'pending' o 'confirmed')

**Respuesta exitosa (200 OK):**
```json
{
  "id": "order_555",
  "status": "cancelled",
  "cancelled_at": "2026-02-11T11:00:00Z"
}
```

**Respuesta de error (400 Bad Request):**
```json
{
  "error": "Cannot cancel order",
  "message": "No se puede cancelar una orden que ya fue enviada."
}
```

---

### 4. Usuarios

#### POST /auth/register
Registra un nuevo usuario

**Body:**
```json
{
  "email": "usuario@ejemplo.com",
  "password": "password123",
  "full_name": "Juan Pérez",
  "phone": "77777777"
}
```

**Respuesta exitosa (201 Created):**
```json
{
  "id": "user_789",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

---

#### POST /auth/login
Inicia sesión

**Body:**
```json
{
  "email": "usuario@ejemplo.com",
  "password": "password123"
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "user_789",
    "email": "usuario@ejemplo.com",
    "full_name": "Juan Pérez"
  }
}
```

---

#### GET /users/me
Obtiene el perfil del usuario autenticado

**Respuesta exitosa (200 OK):**
```json
{
  "id": "user_789",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Pérez",
  "phone": "77777777",
  "created_at": "2026-01-15T09:00:00Z"
}
```

---

#### PUT /users/me
Actualiza el perfil del usuario

**Body:**
```json
{
  "full_name": "Juan Carlos Pérez",
  "phone": "78888888"
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "user_789",
  "email": "usuario@ejemplo.com",
  "full_name": "Juan Carlos Pérez",
  "phone": "78888888",
  "updated_at": "2026-02-11T12:00:00Z"
}
```

---

## Códigos de Respuesta HTTP

| Código | Significado | Uso |
|--------|-------------|-----|
| 200 | OK | Solicitud exitosa |
| 201 | Created | Recurso creado exitosamente |
| 204 | No Content | Acción exitosa sin contenido de respuesta |
| 400 | Bad Request | Error en los datos enviados |
| 401 | Unauthorized | No autenticado / Token inválido |
| 403 | Forbidden | Sin permisos (ej. usuario normal intentando eliminar productos) |
| 404 | Not Found | Recurso no encontrado |
| 409 | Conflict | Conflicto (ej. email ya registrado) |
| 422 | Unprocessable Entity | Validación fallida |
| 429 | Too Many Requests | Límite de tasa excedido |
| 500 | Internal Server Error | Error del servidor |
| 503 | Service Unavailable | Servicio temporalmente no disponible |

---

## Límites de Tasa (Rate Limiting)

| Tipo de Usuario | Límite |
|-----------------|--------|
| Usuario anónimo | 100 requests / hora |
| Usuario autenticado | 1000 requests / hora |
| Admin | Sin límite |

**Respuesta cuando se excede el límite (429):**
```json
{
  "error": "Rate limit exceeded",
  "message": "Has excedido el límite de peticiones. Intenta en 30 minutos.",
  "retry_after": 1800
}
```

---

## Paginación

Todos los endpoints que devuelven listas soportan paginación:

**Parámetros:**
- `page`: Número de página (default: 1)
- `limit`: Resultados por página (default: 20, max: 100)

**Estructura de respuesta:**
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total_pages": 10,
    "total_items": 195,
    "has_next": true,
    "has_prev": true
  }
}
```

---

## Filtrado y Ordenamiento

### Filtrado
Usar parámetros de query específicos por endpoint.

Ejemplo: `GET /products?category=beverages&price_min=10&price_max=50`

### Ordenamiento
Usar parámetro `sort`:
- `GET /products?sort=price:asc` - Orden ascendente por precio
- `GET /products?sort=created_at:desc` - Más recientes primero

---

## Versionado de la API

La API utiliza versionado en la URL:
- Versión actual: `v1`
- Base URL: `https://api.telocompro.com/v1`

Cuando se lance una nueva versión, se mantendrá la anterior por al menos 6 meses.

---

## Manejo de Errores

Todas las respuestas de error siguen el mismo formato:

```json
{
  "error": "Error name",
  "message": "Descripción legible para el usuario",
  "details": {
    "field": "Detalles específicos (opcional)"
  },
  "timestamp": "2026-02-11T10:30:00Z",
  "request_id": "req_abc123"
}
```

---

## Ejemplos de Uso

### Flujo completo: Crear una orden

```bash
# 1. Registrarse
curl -X POST https://api.telocompro.com/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "nuevo@ejemplo.com",
    "password": "pass123",
    "full_name": "María García"
  }'

# 2. Agregar productos al carrito
curl -X POST https://api.telocompro.com/v1/cart/items \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "product_id": "prod_123",
    "quantity": 2
  }'

# 3. Crear la orden
curl -X POST https://api.telocompro.com/v1/orders \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "delivery_address_id": "addr_111",
    "payment_method": "credit_card"
  }'

# 4. Consultar el estado de la orden
curl -X GET https://api.telocompro.com/v1/orders/order_555 \
  -H "Authorization: Bearer {token}"
```

---

## Webhooks (Próximamente)

En una futura versión se añadirá soporte para webhooks que notifiquen eventos importantes:
- `order.created`
- `order.status_changed`
- `payment.completed`
- `delivery.completed`

---

## Documentación Interactiva

Swagger UI: `https://api.telocompro.com/docs`

OpenAPI Spec: `https://api.telocompro.com/openapi.json`

---

**Última actualización:** Febrero 2026  
**Versión de la API:** v1.0.0
