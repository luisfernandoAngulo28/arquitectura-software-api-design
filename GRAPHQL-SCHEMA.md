# GraphQL API - Esquema y Documentación

## Endpoint
```
https://api.telocompro.com/graphql
```

## Autenticación
Incluir el token JWT en el header:
```
Authorization: Bearer {token}
```

---

## Schema Definition Language (SDL)

### Types

#### Product
```graphql
type Product {
  id: ID!
  name: String!
  description: String
  price: Float!
  stock: Int!
  category: Category!
  images: [String!]!
  reviews: [Review!]!
  averageRating: Float
  createdAt: DateTime!
  updatedAt: DateTime!
}
```

---

#### Category
```graphql
type Category {
  id: ID!
  name: String!
  slug: String!
  description: String
  products(
    limit: Int = 20
    offset: Int = 0
  ): [Product!]!
  productCount: Int!
}
```

---

#### Review
```graphql
type Review {
  id: ID!
  product: Product!
  user: User!
  rating: Int!
  comment: String
  createdAt: DateTime!
}
```

---

#### User
```graphql
type User {
  id: ID!
  email: String!
  fullName: String!
  phone: String
  avatar: String
  orders: [Order!]!
  reviews: [Review!]!
  createdAt: DateTime!
}
```

---

#### Cart
```graphql
type Cart {
  id: ID!
  user: User!
  items: [CartItem!]!
  total: Float!
  itemCount: Int!
}
```

---

#### CartItem
```graphql
type CartItem {
  product: Product!
  quantity: Int!
  unitPrice: Float!
  subtotal: Float!
}
```

---

#### Order
```graphql
type Order {
  id: ID!
  user: User!
  items: [OrderItem!]!
  status: OrderStatus!
  total: Float!
  deliveryAddress: Address!
  paymentMethod: PaymentMethod!
  notes: String
  createdAt: DateTime!
  updatedAt: DateTime!
  estimatedDelivery: DateTime
  deliveredAt: DateTime
}
```

---

#### OrderItem
```graphql
type OrderItem {
  product: Product!
  quantity: Int!
  unitPrice: Float!
  subtotal: Float!
}
```

---

#### Address
```graphql
type Address {
  id: ID!
  user: User!
  street: String!
  city: String!
  state: String
  zipCode: String
  phone: String!
  isDefault: Boolean!
}
```

---

### Enums

#### OrderStatus
```graphql
enum OrderStatus {
  PENDING
  CONFIRMED
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}
```

---

#### PaymentMethod
```graphql
enum PaymentMethod {
  CASH
  CREDIT_CARD
  DEBIT_CARD
  MUNDO_CUOTAS
  BANK_TRANSFER
}
```

---

### Custom Scalars

```graphql
scalar DateTime
scalar Upload
```

---

## Queries

```graphql
type Query {
  # ============================================
  # PRODUCTS
  # ============================================
  
  """
  Obtiene la lista de productos con paginación y filtros
  """
  products(
    page: Int = 1
    limit: Int = 20
    category: String
    minPrice: Float
    maxPrice: Float
    search: String
    sortBy: ProductSortBy = CREATED_DESC
  ): ProductConnection!
  
  """
  Obtiene un producto específico por ID
  """
  product(id: ID!): Product
  
  """
  Busca productos por texto
  """
  searchProducts(query: String!): [Product!]!
  
  # ============================================
  # CATEGORIES
  # ============================================
  
  """
  Obtiene todas las categorías
  """
  categories: [Category!]!
  
  """
  Obtiene una categoría específica por slug
  """
  category(slug: String!): Category
  
  # ============================================
  # CART
  # ============================================
  
  """
  Obtiene el carrito del usuario autenticado
  """
  myCart: Cart
  
  # ============================================
  # ORDERS
  # ============================================
  
  """
  Obtiene las órdenes del usuario autenticado
  """
  myOrders(
    status: OrderStatus
    limit: Int = 10
  ): [Order!]!
  
  """
  Obtiene una orden específica por ID
  """
  order(id: ID!): Order
  
  # ============================================
  # USER
  # ============================================
  
  """
  Obtiene el perfil del usuario autenticado
  """
  me: User
  
  """
  Obtiene las direcciones del usuario
  """
  myAddresses: [Address!]!
}
```

---

### Tipos de Soporte para Queries

```graphql
enum ProductSortBy {
  CREATED_DESC
  CREATED_ASC
  PRICE_ASC
  PRICE_DESC
  NAME_ASC
  NAME_DESC
  RATING_DESC
}

type ProductConnection {
  edges: [ProductEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type ProductEdge {
  node: Product!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

## Mutations

```graphql
type Mutation {
  # ============================================
  # AUTHENTICATION
  # ============================================
  
  """
  Registra un nuevo usuario
  """
  register(input: RegisterInput!): AuthPayload!
  
  """
  Inicia sesión
  """
  login(input: LoginInput!): AuthPayload!
  
  """
  Cierra sesión (invalida el token)
  """
  logout: Boolean!
  
  # ============================================
  # CART OPERATIONS
  # ============================================
  
  """
  Agrega un producto al carrito
  """
  addToCart(
    productId: ID!
    quantity: Int!
  ): Cart!
  
  """
  Remueve un producto del carrito
  """
  removeFromCart(productId: ID!): Cart!
  
  """
  Actualiza la cantidad de un producto en el carrito
  """
  updateCartItemQuantity(
    productId: ID!
    quantity: Int!
  ): Cart!
  
  """
  Vacía el carrito completo
  """
  clearCart: Cart!
  
  # ============================================
  # ORDER OPERATIONS
  # ============================================
  
  """
  Crea una nueva orden a partir del carrito
  """
  createOrder(input: CreateOrderInput!): Order!
  
  """
  Cancela una orden
  """
  cancelOrder(orderId: ID!): Order!
  
  # ============================================
  # USER OPERATIONS
  # ============================================
  
  """
  Actualiza el perfil del usuario
  """
  updateProfile(input: UpdateProfileInput!): User!
  
  """
  Agrega una nueva dirección
  """
  addAddress(input: AddressInput!): Address!
  
  """
  Actualiza una dirección existente
  """
  updateAddress(
    id: ID!
    input: AddressInput!
  ): Address!
  
  """
  Elimina una dirección
  """
  deleteAddress(id: ID!): Boolean!
  
  # ============================================
  # REVIEWS
  # ============================================
  
  """
  Crea una reseña para un producto
  """
  createReview(input: CreateReviewInput!): Review!
  
  """
  Actualiza una reseña existente
  """
  updateReview(
    id: ID!
    input: UpdateReviewInput!
  ): Review!
  
  """
  Elimina una reseña
  """
  deleteReview(id: ID!): Boolean!
}
```

---

## Input Types

```graphql
input RegisterInput {
  email: String!
  password: String!
  fullName: String!
  phone: String
}

input LoginInput {
  email: String!
  password: String!
}

input CreateOrderInput {
  deliveryAddressId: ID!
  paymentMethod: PaymentMethod!
  notes: String
}

input UpdateProfileInput {
  fullName: String
  phone: String
  avatar: Upload
}

input AddressInput {
  street: String!
  city: String!
  state: String
  zipCode: String
  phone: String!
  isDefault: Boolean
}

input CreateReviewInput {
  productId: ID!
  rating: Int!
  comment: String
}

input UpdateReviewInput {
  rating: Int
  comment: String
}
```

---

## Respuestas de Autenticación

```graphql
type AuthPayload {
  token: String!
  user: User!
}
```

---

## Subscriptions (Real-time)

```graphql
type Subscription {
  """
  Se suscribe a cambios de estado de una orden específica
  """
  orderStatusChanged(orderId: ID!): Order!
  
  """
  Se suscribe a nuevos productos agregados
  """
  productAdded(categoryId: ID): Product!
  
  """
  Se suscribe a cambios de stock de un producto
  """
  productStockChanged(productId: ID!): Product!
}
```

---

## Ejemplos de Uso

### Consulta 1: Obtener productos con detalles de categoría

**Query:**
```graphql
query GetProducts {
  products(page: 1, limit: 10, category: "beverages") {
    edges {
      node {
        id
        name
        price
        stock
        images
        category {
          name
          slug
        }
        averageRating
      }
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
    }
    totalCount
  }
}
```

**Respuesta:**
```json
{
  "data": {
    "products": {
      "edges": [
        {
          "node": {
            "id": "prod_123",
            "name": "Coca Cola 2L",
            "price": 12.50,
            "stock": 100,
            "images": ["https://cdn.example.com/cocacola.jpg"],
            "category": {
              "name": "Bebidas",
              "slug": "beverages"
            },
            "averageRating": 4.5
          }
        }
      ],
      "pageInfo": {
        "hasNextPage": true,
        "hasPreviousPage": false
      },
      "totalCount": 45
    }
  }
}
```

---

### Consulta 2: Obtener carrito con productos anidados

**Query:**
```graphql
query MyCart {
  myCart {
    id
    items {
      product {
        id
        name
        price
        images
        stock
      }
      quantity
      subtotal
    }
    total
    itemCount
  }
}
```

**Respuesta:**
```json
{
  "data": {
    "myCart": {
      "id": "cart_456",
      "items": [
        {
          "product": {
            "id": "prod_123",
            "name": "Coca Cola 2L",
            "price": 12.50,
            "images": ["https://cdn.example.com/cocacola.jpg"],
            "stock": 100
          },
          "quantity": 2,
          "subtotal": 25.00
        }
      ],
      "total": 25.00,
      "itemCount": 2
    }
  }
}
```

---

### Consulta 3: Obtener orden con todos los detalles

**Query:**
```graphql
query GetOrder($orderId: ID!) {
  order(id: $orderId) {
    id
    status
    total
    items {
      product {
        id
        name
        images
      }
      quantity
      unitPrice
      subtotal
    }
    deliveryAddress {
      street
      city
      phone
    }
    paymentMethod
    createdAt
    estimatedDelivery
    deliveredAt
  }
}
```

**Variables:**
```json
{
  "orderId": "order_555"
}
```

**Respuesta:**
```json
{
  "data": {
    "order": {
      "id": "order_555",
      "status": "DELIVERED",
      "total": 37.50,
      "items": [
        {
          "product": {
            "id": "prod_123",
            "name": "Coca Cola 2L",
            "images": ["https://cdn.example.com/cocacola.jpg"]
          },
          "quantity": 3,
          "unitPrice": 12.50,
          "subtotal": 37.50
        }
      ],
      "deliveryAddress": {
        "street": "Av. Arce 123",
        "city": "La Paz",
        "phone": "77777777"
      },
      "paymentMethod": "CREDIT_CARD",
      "createdAt": "2026-02-11T10:30:00Z",
      "estimatedDelivery": "2026-02-14T18:00:00Z",
      "deliveredAt": "2026-02-13T14:20:00Z"
    }
  }
}
```

---

### Mutación 1: Agregar producto al carrito

**Mutation:**
```graphql
mutation AddToCart($productId: ID!, $quantity: Int!) {
  addToCart(productId: $productId, quantity: $quantity) {
    id
    items {
      product {
        name
      }
      quantity
    }
    total
    itemCount
  }
}
```

**Variables:**
```json
{
  "productId": "prod_123",
  "quantity": 2
}
```

**Respuesta:**
```json
{
  "data": {
    "addToCart": {
      "id": "cart_456",
      "items": [
        {
          "product": {
            "name": "Coca Cola 2L"
          },
          "quantity": 2
        }
      ],
      "total": 25.00,
      "itemCount": 2
    }
  }
}
```

---

### Mutación 2: Crear orden

**Mutation:**
```graphql
mutation CreateOrder($input: CreateOrderInput!) {
  createOrder(input: $input) {
    id
    status
    total
    items {
      product {
        name
      }
      quantity
    }
    deliveryAddress {
      street
      city
    }
    estimatedDelivery
    createdAt
  }
}
```

**Variables:**
```json
{
  "input": {
    "deliveryAddressId": "addr_111",
    "paymentMethod": "CREDIT_CARD",
    "notes": "Tocar el timbre"
  }
}
```

**Respuesta:**
```json
{
  "data": {
    "createOrder": {
      "id": "order_555",
      "status": "PENDING",
      "total": 37.50,
      "items": [
        {
          "product": {
            "name": "Coca Cola 2L"
          },
          "quantity": 3
        }
      ],
      "deliveryAddress": {
        "street": "Av. Arce 123",
        "city": "La Paz"
      },
      "estimatedDelivery": "2026-02-14T18:00:00Z",
      "createdAt": "2026-02-11T10:30:00Z"
    }
  }
}
```

---

### Mutación 3: Crear reseña

**Mutation:**
```graphql
mutation CreateReview($input: CreateReviewInput!) {
  createReview(input: $input) {
    id
    rating
    comment
    user {
      fullName
    }
    createdAt
  }
}
```

**Variables:**
```json
{
  "input": {
    "productId": "prod_123",
    "rating": 5,
    "comment": "Excelente producto, llegó en perfectas condiciones"
  }
}
```

---

### Subscription: Cambio de estado de orden

**Subscription:**
```graphql
subscription OrderUpdates($orderId: ID!) {
  orderStatusChanged(orderId: $orderId) {
    id
    status
    updatedAt
  }
}
```

**Variables:**
```json
{
  "orderId": "order_555"
}
```

**Stream de Respuestas:**
```json
// Primera actualización
{
  "data": {
    "orderStatusChanged": {
      "id": "order_555",
      "status": "CONFIRMED",
      "updatedAt": "2026-02-11T10:35:00Z"
    }
  }
}

// Segunda actualización
{
  "data": {
    "orderStatusChanged": {
      "id": "order_555",
      "status": "PROCESSING",
      "updatedAt": "2026-02-11T11:00:00Z"
    }
  }
}
```

---

## Manejo de Errores

GraphQL siempre devuelve un código HTTP 200, pero los errores se incluyen en el campo `errors`:

```json
{
  "errors": [
    {
      "message": "Product not found",
      "locations": [
        {
          "line": 2,
          "column": 3
        }
      ],
      "path": ["product"],
      "extensions": {
        "code": "NOT_FOUND",
        "productId": "prod_999"
      }
    }
  ],
  "data": {
    "product": null
  }
}
```

### Códigos de Error Comunes

| Código | Descripción |
|--------|-------------|
| `UNAUTHENTICATED` | Usuario no autenticado |
| `FORBIDDEN` | Sin permisos |
| `NOT_FOUND` | Recurso no encontrado |
| `BAD_USER_INPUT` | Datos de entrada inválidos |
| `INTERNAL_SERVER_ERROR` | Error del servidor |
| `INSUFFICIENT_STOCK` | Stock insuficiente |
| `EMPTY_CART` | Carrito vacío |

---

## Ventajas de GraphQL sobre REST

### 1. Sin Over-fetching
El cliente solicita exactamente los datos que necesita, nada más.

**Ejemplo:**
```graphql
# Solo nombre y precio (2 campos)
query {
  product(id: "prod_123") {
    name
    price
  }
}
```

vs REST que devuelve todos los campos del producto (20+ campos).

---

### 2. Sin Under-fetching
Una sola query puede obtener datos relacionados de múltiples recursos.

**GraphQL (1 request):**
```graphql
query {
  order(id: "order_555") {
    id
    status
    items {
      product {
        name
        category {
          name
        }
      }
    }
    user {
      fullName
    }
  }
}
```

**REST requeriría 4 requests:**
1. `GET /orders/555`
2. `GET /products/123`
3. `GET /categories/1`
4. `GET /users/789`

---

### 3. Fuertemente Tipado
El esquema define claramente qué datos están disponibles y de qué tipos son.

**Ventajas:**
- Validación automática
- Auto-completado en IDEs
- Documentación auto-generada
- Detección temprana de errores

---

### 4. Versionado No Necesario
Se pueden agregar nuevos campos sin romper clientes existentes.

**Evolución del esquema:**
```graphql
type Product {
  name: String!
  price: Float!
  stock: Int!
  # Nuevo campo agregado (no rompe clientes existentes)
  discount: Float
}
```

---

### 5. Introspección
El esquema es consultable, permitiendo herramientas como GraphiQL, Apollo Studio.

**Query de introspección:**
```graphql
query {
  __schema {
    types {
      name
      description
    }
  }
}
```

---

### 6. Optimización de Queries
Técnicas como DataLoader previenen el problema N+1.

---

## Herramientas Recomendadas

### Clientes
- **Apollo Client** (React, Angular, Vue)
- **Relay** (React, optimizado)
- **urql** (React, ligero)
- **graphql-request** (Cliente HTTP simple)

### Playground/IDE
- **Apollo Studio** - https://studio.apollographql.com/
- **GraphiQL** - https://github.com/graphql/graphiql
- **Altair** - https://altair.sirmuel.design/
- **Insomnia** - Soporte para GraphQL

### Monitoreo
- **Apollo Studio** (Métricas y trazas)
- **GraphQL Inspector** (Diff de esquemas)
- **Sentry** (Error tracking)

---

## Mejores Prácticas

### 1. Nombrado de Campos
- Usar `camelCase` para campos y argumentos
- Usar `PascalCase` para tipos y enums
- Nombres descriptivos

### 2. Paginación
- Usar Cursor-based pagination (Relay spec)
- Limitar resultados por defecto
- Proporcionar `totalCount`

### 3. Nullability
- Campos requeridos: `String!`
- Campos opcionales: `String`
- Pensar cuidadosamente en la nullability

### 4. Documentación
- Documentar todos los tipos y campos con `"""`
- Incluir ejemplos en la descripción

### 5. Autenticación
- Usar contexto de GraphQL para el usuario
- Validar permisos en resolvers
- No exponer datos sensibles

---

## GraphQL Playground

Accede a la documentación interactiva y prueba queries:

**URL:** `https://api.telocompro.com/graphql`

**Características:**
- Auto-completado de queries
- Documentación interactiva del esquema
- Historial de queries
- Variables y headers configurables

---

**Última actualización:** Febrero 2026  
**Versión del esquema:** 1.0.0
