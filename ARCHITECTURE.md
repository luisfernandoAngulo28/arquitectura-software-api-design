# Arquitectura del Sistema TeLoCompro

## Diagrama de Alto Nivel

```
┌─────────────────────────────────────────────────────────────────┐
│                      CLIENTE / FRONTEND                         │
│                                                                 │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐      │
│  │   Web App    │   │  Mobile App  │   │ Admin Panel  │      │
│  │   (React)    │   │(React Native)│   │   (React)    │      │
│  └──────┬───────┘   └──────┬───────┘   └──────┬───────┘      │
└─────────┼──────────────────┼──────────────────┼───────────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
                    ┌────────▼────────┐
                    │   API GATEWAY   │
                    │  (Kong/Nginx)   │
                    └────────┬────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼──────┐    ┌─────▼──────┐    ┌─────▼──────┐
    │  REST API  │    │ GraphQL API│    │ WebSocket  │
    │  Service   │    │  Service   │    │  Service   │
    │ (Node.js)  │    │ (Node.js)  │    │ (Node.js)  │
    └─────┬──────┘    └─────┬──────┘    └─────┬──────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼───────┐   ┌──────▼────────┐   ┌────▼────────┐
    │  Business   │   │   Business    │   │  Business   │
    │   Logic     │   │     Logic     │   │    Logic    │
    │   Layer     │   │     Layer     │   │    Layer    │
    │             │   │               │   │             │
    │ - Products  │   │- Shopping Cart│   │ - Orders    │
    │- Categories │   │  - Payments   │   │ - Delivery  │
    │  - Users    │   │  - Credits    │   │-Notifications│
    └─────┬───────┘   └──────┬────────┘   └────┬────────┘
          │                  │                  │
          └──────────────────┼──────────────────┘
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
    ┌─────▼──────┐    ┌──────▼──────┐    ┌─────▼──────┐
    │ PostgreSQL │    │    Redis    │    │  MongoDB   │
    │ (Primary)  │    │   (Cache)   │    │   (Logs)   │
    └────────────┘    └─────────────┘    └────────────┘


SERVICIOS EXTERNOS:
┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
│  Firebase  │  │   Sentry   │  │   Stripe   │  │   AWS S3   │
│ Analytics  │  │ Monitoring │  │  Payments  │  │  Storage   │
└────────────┘  └────────────┘  └────────────┘  └────────────┘
```

---

## Componentes Principales

### 1. Frontend Layer

#### Web App (React)
- **Propósito:** Aplicación web para usuarios finales
- **Tecnologías:** 
  - React 18
  - TypeScript
  - Redux Toolkit (State management)
  - React Router (Navegación)
  - Material-UI (Componentes)
- **Funcionalidades:**
  - Catálogo de productos
  - Carrito de compras
  - Gestión de órdenes
  - Perfil de usuario
  - Historial de compras

#### Mobile App (React Native)
- **Propósito:** Aplicación móvil nativa para iOS y Android
- **Tecnologías:**
  - React Native 0.71+
  - TypeScript
  - Redux Toolkit
  - React Navigation
  - Native Base (UI Components)
- **Funcionalidades:**
  - Todas las funcionalidades del Web App
  - Notificaciones push
  - Geolocalización
  - Cámara (escaneo de códigos)

#### Admin Panel (React)
- **Propósito:** Panel de administración para gestión del sistema
- **Tecnologías:**
  - React 18
  - TypeScript
  - Material-UI
  - React Admin
- **Funcionalidades:**
  - Gestión de productos
  - Gestión de categorías
  - Gestión de usuarios
  - Monitoreo de órdenes
  - Reportes y analytics
  - Configuración del sistema

---

### 2. API Gateway

**Tecnología:** Kong o Nginx

**Propósito:** Punto único de entrada para todas las peticiones del cliente

**Funciones:**
- **Rate Limiting:** Limitar peticiones por usuario/IP
- **Autenticación/Autorización:** Validar tokens JWT
- **Enrutamiento:** Dirigir peticiones al servicio correcto
- **Load Balancing:** Distribuir carga entre instancias
- **Logging Centralizado:** Registrar todas las peticiones
- **CORS:** Configuración de políticas de origen cruzado
- **SSL/TLS Termination:** Manejo de certificados
- **Request/Response Transformation:** Modificar headers, body
- **Circuit Breaking:** Prevenir cascadas de fallos

**Configuración de Rutas:**
```
/api/v1/rest/*     → REST API Service
/api/v1/graphql    → GraphQL API Service
/api/v1/ws/*       → WebSocket Service
```

---

### 3. Backend Services

#### REST API Service
- **Tecnología:** Node.js + Express
- **Puerto:** 3001
- **Responsabilidades:**
  - Endpoints RESTful tradicionales
  - CRUD operations estándar
  - Compatible con clientes legacy
  - Documentación OpenAPI/Swagger

**Estructura:**
```
rest-api-service/
├── src/
│   ├── controllers/
│   ├── routes/
│   ├── middlewares/
│   ├── validators/
│   └── app.js
├── tests/
└── package.json
```

---

#### GraphQL API Service
- **Tecnología:** Node.js + Apollo Server
- **Puerto:** 3002
- **Responsabilidades:**
  - Resolución de queries y mutations
  - Subscriptions para tiempo real
  - DataLoader para N+1 optimization
  - Schema stitching

**Estructura:**
```
graphql-api-service/
├── src/
│   ├── schema/
│   │   ├── types/
│   │   ├── queries/
│   │   ├── mutations/
│   │   └── subscriptions/
│   ├── resolvers/
│   ├── dataloaders/
│   └── server.js
├── tests/
└── package.json
```

---

#### WebSocket Service
- **Tecnología:** Node.js + Socket.io
- **Puerto:** 3003
- **Responsabilidades:**
  - Conexiones bidireccionales persistentes
  - Notificaciones en tiempo real
  - Chat de soporte
  - Actualizaciones de estado de órdenes
  - Notificaciones de stock

**Eventos:**
```javascript
// Cliente → Servidor
- 'join:order' - Unirse a sala de orden
- 'send:message' - Enviar mensaje de chat
- 'track:order' - Rastrear orden

// Servidor → Cliente
- 'order:status_changed' - Cambio de estado
- 'notification:new' - Nueva notificación
- 'message:received' - Mensaje de chat
- 'stock:updated' - Actualización de stock
```

---

### 4. Business Logic Layer

**Arquitectura:** Clean Architecture con Feature Modules

**Principios:**
- **Separation of Concerns:** Cada módulo tiene su responsabilidad
- **Dependency Inversion:** Dependencias apuntan hacia abstracciones
- **Single Responsibility:** Una razón para cambiar
- **DRY (Don't Repeat Yourself):** Código reutilizable

#### Módulos:

##### Products & Categories Management
- Crear, leer, actualizar, eliminar productos
- Gestión de categorías
- Gestión de inventario
- Búsqueda y filtrado avanzado
- Recomendaciones de productos

##### Shopping Cart & Checkout
- Agregar/remover items del carrito
- Actualizar cantidades
- Aplicar códigos de descuento
- Calcular totales con impuestos
- Validación de stock antes de checkout

##### Order Processing & Tracking
- Creación de órdenes
- Máquina de estados (FSM) para status
- Tracking de entrega
- Cancelación de órdenes
- Historial de órdenes

##### User Management & Authentication
- Registro y login
- Gestión de perfiles
- Gestión de direcciones
- Preferencias de usuario
- Autenticación JWT
- Refresh tokens

##### Payment Processing
- Integración con Stripe
- Gestión de métodos de pago
- Procesamiento de pagos
- Webhooks de confirmación
- Manejo de reembolsos

##### Credit Line Management
- Solicitud de crédito (Mundo Cuotas)
- Verificación de límite de crédito
- Gestión de pagos a plazos
- Historial de crédito

##### Delivery Management
- Cálculo de costos de envío
- Asignación de repartidores
- Tracking en tiempo real
- Confirmación de entrega

##### Notifications & Communications
- Templates de emails
- Push notifications (Firebase)
- SMS notifications
- In-app notifications
- Email marketing

---

### 5. Data Layer

#### PostgreSQL (Primary Database)
- **Versión:** 14+
- **Propósito:** Base de datos transaccional principal
- **Datos almacenados:**
  - Usuarios
  - Productos y categorías
  - Órdenes y order items
  - Carritos
  - Direcciones
  - Métodos de pago
  - Transacciones

**Schema Principal:**
```sql
-- Tablas principales
users
products
categories
orders
order_items
carts
cart_items
addresses
payments
reviews
```

**Índices Importantes:**
```sql
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_products_price ON products(price);
```

**Optimizaciones:**
- Primary-Replica replication
- Connection pooling (PgBouncer)
- Índices optimizados
- Query optimization
- Regular VACUUM

---

#### Redis (Cache & Session Store)
- **Versión:** 7+
- **Propósito:** Cache de alta velocidad y almacenamiento de sesiones
- **Datos almacenados:**
  - Cache de queries frecuentes
  - Sesiones de usuario
  - Rate limiting data
  - Real-time data (stock disponible)
  - Carritos temporales
  - Tokens de verificación

**Estructura de Keys:**
```
user:session:{userId}        → TTL: 24h
product:cache:{productId}    → TTL: 1h
cart:temp:{sessionId}        → TTL: 7d
ratelimit:{userId}           → TTL: 1h
verification:email:{token}   → TTL: 15min
```

**Configuración:**
```
maxmemory-policy: allkeys-lru
maxmemory: 2gb
appendonly: yes
```

---

#### MongoDB (Logs & Analytics)
- **Versión:** 6+
- **Propósito:** Almacenamiento de logs y datos analíticos
- **Datos almacenados:**
  - Application logs
  - Access logs
  - Error logs
  - User activity tracking
  - Analytics events
  - A/B testing data
  - Search queries

**Collections:**
```javascript
- app_logs
- access_logs
- error_logs
- user_events
- search_logs
- analytics_events
```

**Ventajas:**
- Schema flexible para logs variados
- Excelente performance para escrituras
- Agregaciones potentes para analytics
- Time-series collections

---

### 6. Servicios Externos

#### Firebase
- **Cloud Messaging:** Push notifications a móviles
- **Analytics:** Tracking de eventos y comportamiento
- **Remote Config:** Configuración dinámica sin deploy
- **A/B Testing:** Experimentos de UX

#### Sentry
- **Error Tracking:** Monitoreo de errores en producción
- **Performance Monitoring:** Detección de cuellos de botella
- **Release Tracking:** Asociar errores a versiones
- **Alertas:** Notificaciones en Slack/Email

#### Stripe
- **Payment Processing:** Procesamiento de tarjetas
- **Webhooks:** Confirmaciones de pago
- **Subscriptions:** Gestión de suscripciones
- **Fraud Detection:** Detección de fraude

#### AWS S3
- **Object Storage:** Almacenamiento de imágenes de productos
- **CDN Integration:** CloudFront para entrega rápida
- **Backup Storage:** Respaldos de base de datos
- **Static Assets:** CSS, JS, fuentes

---

## Flujo de Datos - Ejemplo: Crear Orden

### Flujo Completo

```
1. Usuario (Mobile App)
   ↓
   GraphQL Mutation: createOrder(...)
   ↓
2. API Gateway
   ↓
   - Valida token JWT ✓
   - Rate limiting check ✓
   - Rutea a GraphQL Service
   ↓
3. GraphQL API Service
   ↓
   - Resolver de createOrder
   - Valida input
   ↓
4. Business Logic Layer - OrderService
   ↓
   - Valida usuario autenticado ✓
   - Obtiene items del carrito (Redis cache)
   - Valida stock disponible (PostgreSQL) ✓
   - Valida método de pago ✓
   - Calcula total con impuestos
   ↓
5. PostgreSQL - Transacción
   ↓
   BEGIN TRANSACTION;
   - INSERT INTO orders (...)
   - INSERT INTO order_items (...)
   - UPDATE products SET stock = stock - qty
   - DELETE FROM carts WHERE user_id = ...
   COMMIT;
   ↓
6. Redis
   ↓
   - CLEAR cart:{userId}
   - SET order:{orderId} (cache por 1h)
   ↓
7. Servicios Externos
   ↓
   - Firebase: Send push notification
   - Stripe: Charge payment (webhook)
   - MongoDB: Log order_created event
   ↓
8. Response a Cliente
   ↓
   {
     id: "order_555",
     status: "CONFIRMED",
     total: 37.50
   }
   ↓
9. WebSocket Service
   ↓
   - Emit 'order:status_changed' event
   - Usuarios conectados reciben actualización en tiempo real
```

### Diagrama de Secuencia

```
User          Gateway       GraphQL       OrderService    PostgreSQL    Redis    External
 │               │             │               │              │          │         │
 ├──createOrder──>│             │               │              │          │         │
 │               ├─validate────>│               │              │          │         │
 │               │<─OK──────────┤               │              │          │         │
 │               ├─route────────>│               │              │          │         │
 │               │              ├──createOrder──>│              │          │         │
 │               │              │               ├─getCart──────>│          │         │
 │               │              │               │<─items────────┤          │         │
 │               │              │               ├─validateStock─>│         │         │
 │               │              │               │<─OK────────────┤         │         │
 │               │              │               ├─saveOrder──────>│         │         │
 │               │              │               │<─orderId───────┤         │         │
 │               │              │               ├─clearCart──────>│         │         │
 │               │              │               ├─cacheOrder─────>│         │         │
 │               │              │               ├─chargePayment──┼─────────>│         │
 │               │              │               ├─sendNotif───────┼──────────────────>│
 │               │              │<─order────────┤                 │         │         │
 │               │<─order───────┤                                 │         │         │
 │<─order────────┤                                                │         │         │
```

---

## Escalabilidad y Resiliencia

### Horizontal Scaling

**Estrategia:** Múltiples instancias de cada servicio detrás del API Gateway

**Implementación:**
- **Container Orchestration:** Kubernetes o Docker Swarm
- **Load Balancer:** Nginx o AWS ALB
- **Service Discovery:** Consul o Kubernetes DNS
- **Auto-scaling:** Basado en métricas (CPU, memoria, requests/seg)

**Configuración de Auto-scaling:**
```yaml
# Kubernetes HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: graphql-api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: graphql-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

### Fault Tolerance

#### Circuit Breaker Pattern
**Propósito:** Prevenir cascadas de fallos

**Implementación:** Librería `opossum`

```javascript
const circuitBreaker = require('opossum');

const options = {
  timeout: 3000,        // 3s timeout
  errorThresholdPercentage: 50,
  resetTimeout: 30000   // 30s antes de reintentar
};

const breaker = circuitBreaker(externalService, options);

breaker.fallback(() => {
  return { status: 'unavailable', message: 'Service temporarily down' };
});
```

#### Retry Policies
**Estrategia:** Backoff exponencial

```javascript
const retry = require('async-retry');

await retry(async bail => {
  const res = await fetch('https://api.example.com/data');
  if (res.status === 500) {
    throw new Error('Server error');
  }
  return res;
}, {
  retries: 3,
  factor: 2,
  minTimeout: 1000,
  maxTimeout: 5000
});
```

#### Fallback Responses
Cuando un servicio externo falla, devolver respuesta alternativa:

```javascript
async function getProductRecommendations(userId) {
  try {
    return await aiRecommendationService.get(userId);
  } catch (error) {
    logger.error('AI service down, using fallback');
    // Fallback: productos más vendidos
    return await getTopSellingProducts();
  }
}
```

---

### High Availability

#### Database Replication
**PostgreSQL Primary-Replica:**
- 1 Primary (escrituras)
- 2+ Replicas (lecturas)
- Failover automático con Patroni o pg_auto_failover

**Redis Cluster:**
- 3+ master nodes
- Cada master tiene 1+ réplicas
- Sharding automático de keys

#### Multi-AZ Deployment
**AWS:**
- Aplicaciones en múltiples Availability Zones
- RDS Multi-AZ para PostgreSQL
- ElastiCache Multi-AZ para Redis
- S3 (replicación automática)

#### Backup Strategy
**Frecuencia:**
- PostgreSQL: Backup completo diario + WAL archiving continuo
- MongoDB: Backup diario
- Redis: RDB snapshots cada 6h

**Retención:**
- Backups diarios: 30 días
- Backups semanales: 3 meses
- Backups mensuales: 1 año

**Pruebas de Restauración:**
- Mensual en ambiente de staging

---

### Security

#### Autenticación
- **JWT (JSON Web Tokens):** Tokens firmados con RS256
- **Access Token:** TTL 15 minutos
- **Refresh Token:** TTL 7 días (stored in httpOnly cookie)
- **Token Rotation:** Refresh token se rota en cada uso

#### Autorización
- **RBAC (Role-Based Access Control):**
  - Roles: `user`, `admin`, `superadmin`
  - Permisos granulares por recurso

**Ejemplo:**
```javascript
const permissions = {
  user: ['read:products', 'write:cart', 'write:orders'],
  admin: ['read:*', 'write:products', 'write:categories'],
  superadmin: ['*']
};
```

#### Rate Limiting
**Límites por rol:**
- Usuario anónimo: 100 req/hora
- Usuario autenticado: 1000 req/hora
- Admin: Sin límite

**Implementación:** Redis + token bucket algorithm

#### Encriptación
- **En tránsito:** TLS 1.3
- **En reposo:** 
  - Contraseñas: bcrypt (cost factor 12)
  - Datos sensibles: AES-256-GCM
  - Database: PostgreSQL encryption at rest

#### Sanitización de Inputs
- **SQL Injection Prevention:** Prepared statements, ORM (Sequelize/Prisma)
- **XSS Prevention:** Sanitizar HTML, Content Security Policy
- **CSRF Prevention:** CSRF tokens, SameSite cookies

#### CORS
**Configuración:**
```javascript
const corsOptions = {
  origin: [
    'https://telocompro.com',
    'https://admin.telocompro.com'
  ],
  credentials: true,
  optionsSuccessStatus: 200
};
```

#### Security Headers
```javascript
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      styleSrc: ["'self'", 'https://fonts.googleapis.com'],
      imgSrc: ["'self'", 'https://cdn.telocompro.com']
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

---

## Monitoreo y Observabilidad

### Métricas (Metrics)
**Herramienta:** Prometheus + Grafana

**Métricas clave:**
- Request rate (req/s)
- Error rate (%)
- Response time (p50, p95, p99)
- CPU usage (%)
- Memory usage (%)
- Database connections
- Cache hit rate (%)

### Logs
**Stack:** ELK (Elasticsearch, Logstash, Kibana) o Loki

**Niveles:**
- ERROR: Errores que requieren atención
- WARN: Situaciones anómalas pero manejables
- INFO: Eventos importantes
- DEBUG: Información detallada para debugging

**Formato estructurado (JSON):**
```json
{
  "timestamp": "2026-02-11T10:30:00Z",
  "level": "ERROR",
  "service": "order-service",
  "requestId": "req_abc123",
  "userId": "user_789",
  "message": "Failed to charge payment",
  "error": {
    "type": "PaymentError",
    "code": "INSUFFICIENT_FUNDS"
  }
}
```

### Traces (Distributed Tracing)
**Herramienta:** Jaeger o Zipkin

**Propósito:** Rastrear requests a través de múltiples servicios

**Ejemplo de trace:**
```
Trace ID: abc123xyz
├─ API Gateway (50ms)
├─ GraphQL Service (120ms)
│  ├─ OrderService.createOrder (80ms)
│  │  ├─ PostgreSQL query (15ms)
│  │  ├─ Redis get (2ms)
│  │  └─ Stripe API call (50ms)
│  └─ NotificationService.send (30ms)
└─ Response (200ms total)
```

### Alertas
**Herramienta:** Prometheus Alertmanager

**Alertas configuradas:**
- Error rate > 5% por 5 minutos
- Response time p95 > 1s por 5 minutos
- CPU > 80% por 10 minutos
- Database connections > 80% por 5 minutos
- Disk space < 20%

**Canales de notificación:**
- Slack (equipo de desarrollo)
- PagerDuty (on-call engineer)
- Email (gerencia)

---

## Entornos de Despliegue

### Development
- Local (Docker Compose)
- Datos de prueba (seeds)
- Logs verbose

### Staging
- Réplica de producción
- Testing de nuevas features
- Performance testing
- Datos similares a producción (anonimizados)

### Production
- Multi-AZ deployment
- Auto-scaling habilitado
- Backups automáticos
- Monitoreo 24/7

---

## CI/CD Pipeline

```
┌───────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
│ Git Push  │────>│  Build   │────>│  Test    │────>│  Deploy  │
└───────────┘     └──────────┘     └──────────┘     └──────────┘
                       │                │                  │
                       ├─ Lint          ├─ Unit Tests     ├─ Staging
                       ├─ Compile       ├─ Integration    └─ Production
                       └─ Docker Build  └─ E2E Tests         (manual approval)
```

**Herramientas:**
- **Source Control:** GitHub
- **CI/CD:** GitHub Actions o GitLab CI
- **Container Registry:** Docker Hub o AWS ECR
- **Deployment:** Kubernetes + Helm charts

---

## Tech Stack Resumen

| Capa | Tecnología |
|------|------------|
| **Frontend Web** | React, TypeScript, Redux, Material-UI |
| **Frontend Mobile** | React Native, TypeScript, Redux |
| **API Gateway** | Kong / Nginx |
| **Backend** | Node.js, Express, Apollo Server |
| **Databases** | PostgreSQL, Redis, MongoDB |
| **Authentication** | JWT, bcrypt |
| **Payments** | Stripe |
| **Cloud Storage** | AWS S3, CloudFront |
| **Notifications** | Firebase Cloud Messaging |
| **Monitoring** | Sentry, Prometheus, Grafana |
| **Logging** | ELK Stack / Loki |
| **Tracing** | Jaeger |
| **Container** | Docker |
| **Orchestration** | Kubernetes |
| **CI/CD** | GitHub Actions |

---

## Próximos Pasos / Roadmap

### Fase 1 (Q1 2026) - MVP
- ✅ Infraestructura básica
- ✅ Autenticación y autorización
- ✅ Catálogo de productos
- ✅ Carrito y checkout
- ✅ Procesamiento de órdenes

### Fase 2 (Q2 2026) - Mejoras
- 🔄 Sistema de reseñas
- 🔄 Recomendaciones con ML
- 🔄 Chat de soporte en vivo
- 🔄 Programa de fidelización

### Fase 3 (Q3 2026) - Escalabilidad
- 📋 Microservicios independientes
- 📋 Event-driven architecture (Kafka)
- 📋 GraphQL Federation
- 📋 Multi-región deployment

### Fase 4 (Q4 2026) - Innovación
- 📋 Búsqueda por voz
- 📋 AR para visualización de productos
- 📋 Personalización con AI
- 📋 Blockchain para trazabilidad

---

**Última actualización:** Febrero 2026  
**Versión de la arquitectura:** 1.0.0  
**Autor:** Equipo de Arquitectura TeLoCompro
