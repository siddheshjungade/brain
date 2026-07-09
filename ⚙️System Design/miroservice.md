# 🔧 Microservices Interview Preparation — Complete In-Depth Guide
### For: Java Full Stack Engineer | Stack: Java, React, AWS, Microservices

> **This doc covers:** Architecture → Communication → Data → Resilience → Events → Security → Deployment → Observability — all with trade-offs, real patterns, Java/Spring Boot examples, and interview Q&A.

---

# TABLE OF CONTENTS

1. [Fundamentals & Design Principles](#1-fundamentals--design-principles)
2. [Service Decomposition Patterns](#2-service-decomposition-patterns)
3. [Inter-Service Communication](#3-inter-service-communication)
4. [Service Discovery & Load Balancing](#4-service-discovery--load-balancing)
5. [API Gateway Patterns](#5-api-gateway-patterns)
6. [Distributed Data Management](#6-distributed-data-management)
7. [Resilience Patterns](#7-resilience-patterns)
8. [Event-Driven Architecture — CQRS & Saga](#8-event-driven-architecture--cqrs--saga)
9. [Security in Microservices](#9-security-in-microservices)
10. [Deployment — Docker & Kubernetes](#10-deployment--docker--kubernetes)
11. [Observability & Distributed Tracing](#11-observability--distributed-tracing)
12. [Testing Microservices](#12-testing-microservices)

---

---

# 1. Fundamentals & Design Principles

## 🏛️ 1.1 What Are Microservices?

Microservices is an **architectural style** where an application is built as a collection of **small, independently deployable services**, each:
- Running in its own process
- Communicating over lightweight protocols (HTTP/REST, gRPC, messaging)
- Owned by a small team (Conway's Law)
- Having its own database (database per service)
- Deployable independently without affecting others

```
MONOLITH:
┌──────────────────────────────────────────┐
│              E-Commerce App              │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │  Orders  │ │  Users   │ │ Payments │ │
│  └──────────┘ └──────────┘ └──────────┘ │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │Inventory │ │Shipping  │ │Notifications│
│  └──────────┘ └──────────┘ └──────────┘ │
│             Single Database              │
└──────────────────────────────────────────┘
Deploy everything together → scale everything together → one team touches everything

MICROSERVICES:
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Order Service│  │ User Service │  │Payment Service│
│   (Java)     │  │   (Java)     │  │  (Node.js)   │
│  [Orders DB] │  │  [Users DB]  │  │ [Payment DB] │
└──────────────┘  └──────────────┘  └──────────────┘
       ↕                  ↕                 ↕
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│Inventory Svc │  │Shipping Svc  │  │Notification  │
│  (Python)    │  │   (Go)       │  │    Svc       │
│[Inventory DB]│  │ [Shipping DB]│  │  [Notif DB]  │
└──────────────┘  └──────────────┘  └──────────────┘
Deploy independently → scale independently → teams own their services
```

---

## ⚖️ 1.2 Monolith vs Microservices — Honest Trade-offs

| Concern | Monolith | Microservices |
|---------|----------|---------------|
| **Simplicity** | ✅ Simple — one deploy, one codebase | ❌ Complex — many services, many moving parts |
| **Development speed (start)** | ✅ Fast to start | ❌ High initial setup cost |
| **Development speed (scale)** | ❌ Slows as team grows | ✅ Teams work independently |
| **Deployment** | ✅ Simple | ❌ Complex (orchestration needed) |
| **Independent scaling** | ❌ Scale entire app | ✅ Scale only bottleneck service |
| **Technology flexibility** | ❌ One stack | ✅ Best tool per service |
| **Fault isolation** | ❌ One bug can crash all | ✅ One service fails, others continue |
| **Testing** | ✅ Easier end-to-end | ❌ Complex — contract tests, e2e across services |
| **Data consistency** | ✅ ACID transactions | ❌ Eventual consistency, distributed transactions |
| **Network latency** | ✅ In-process calls | ❌ Network hops add latency |
| **Operational overhead** | ✅ Low | ❌ High — monitoring, tracing, service mesh |
| **Team size sweet spot** | 1–50 engineers | 50+ engineers |

### When NOT to use Microservices:
- Small team (< 5 engineers)
- Early-stage startup (requirements unclear)
- Domain not well understood yet
- Simple CRUD application
- No need for independent scaling

> **Martin Fowler's advice:** "Don't start with microservices. Start with a modular monolith. Split when you feel the pain."

---

## 📐 1.3 Design Principles

### Single Responsibility Principle
Each service does ONE thing and does it well.
```
BAD: "User Service" manages users, handles auth, sends emails, tracks activity
GOOD:
  - User Service: manages user profiles
  - Auth Service: authentication and tokens
  - Notification Service: emails, SMS, push
  - Activity Service: tracks user events
```

### Loose Coupling, High Cohesion
```
Loose Coupling:
  - Services don't share databases
  - Services don't call each other's internal methods
  - Changes in Service A don't require changes in Service B
  - Communicate via well-defined APIs/events

High Cohesion:
  - All code related to "Order" lives in Order Service
  - Don't split a single feature across multiple services
```

### Database Per Service (Must-follow)
```
WHY:
- Schema changes in Order DB don't break User Service
- Order Service can use PostgreSQL, User Service can use MongoDB
- Services can evolve independently

CONSEQUENCE:
- No JOIN across service databases
- Joins must be done via API calls or event-driven data replication
- Transactions spanning services need Saga pattern (covered later)
```

### Design for Failure
```
Services WILL fail. Networks WILL have issues. Plan for it:
- Circuit Breakers (prevent cascade failures)
- Timeouts (don't wait forever)
- Retries with backoff (handle transient failures)
- Bulkheads (isolate failures)
- Fallbacks (degrade gracefully)
```

### API First
Design APIs before implementation. Use OpenAPI/Swagger spec as contract.

### Conway's Law
> "Organizations design systems that mirror their own communication structure."

Team per service → service boundaries = team boundaries. If your team structure doesn't match your service structure, services will be chatty and tightly coupled.

---

## 🔢 1.4 The 12-Factor App (Microservices bible)

| Factor | What | Why |
|--------|------|-----|
| **Codebase** | One codebase per service, tracked in VCS | Clear ownership |
| **Dependencies** | Explicitly declare, isolate dependencies | Reproducible builds |
| **Config** | Store config in environment, not code | Same code, different envs |
| **Backing services** | Treat DB, cache as attached resources | Swap without code change |
| **Build/release/run** | Strict separation of stages | Immutable deployments |
| **Processes** | Stateless, share nothing | Horizontal scaling |
| **Port binding** | Self-contained, export via port | No app server dependency |
| **Concurrency** | Scale via process model | Horizontal scaling |
| **Disposability** | Fast startup, graceful shutdown | Rolling deployments |
| **Dev/prod parity** | Keep environments similar | Fewer "works on my machine" bugs |
| **Logs** | Treat as event streams | Centralized logging |
| **Admin processes** | One-off tasks as one-off processes | DB migrations, scripts |

---

---

# 2. Service Decomposition Patterns

## ✂️ 2.1 How to Decompose — Strategies

### Strategy 1: Decompose by Business Capability
Organize around what the business does (most recommended).

```
E-Commerce Business Capabilities:
├── Product Catalog Management
│   └── Product Service
├── Order Management
│   └── Order Service
├── Customer Management
│   └── Customer Service
├── Payment Processing
│   └── Payment Service
├── Inventory Management
│   └── Inventory Service
├── Shipping & Fulfillment
│   └── Shipping Service
└── Notifications
    └── Notification Service
```

### Strategy 2: Decompose by Domain (DDD Bounded Contexts)
Domain-Driven Design: identify bounded contexts = natural service boundaries.

```
Domain: E-Commerce

Bounded Context: "Order" context
  - Order aggregate (Order, OrderItem, OrderStatus)
  - Language: "Order", "LineItem", "Fulfillment"

Bounded Context: "Catalog" context
  - Product aggregate (Product, Category, Price)
  - Language: "Product", "SKU", "Listing"

Bounded Context: "Inventory" context
  - Stock aggregate (StockItem, Warehouse, Reservation)
  - Language: "Stock", "Reservation", "Replenishment"

Same "Product" concept = different meanings in different contexts:
  Catalog:   Product has description, images, attributes
  Inventory: Product has stockLevel, warehouseLocation
  Order:     Product has price, name (snapshot at time of order)
  
→ Each context has its own model, no shared DB
```

### Strategy 3: Decompose by Subdomain
```
Core Domain (competitive advantage — invest most here):
  - Order Management, Recommendation Engine

Supporting Subdomain (necessary but not unique):
  - Inventory Management, Shipping

Generic Subdomain (buy/reuse, don't build):
  - Email delivery (use SendGrid), Payments (use Stripe)
```

### Strategy 4: Strangler Fig Pattern (Monolith → Microservices migration)
```
Step 1: New functionality as new microservice
Step 2: Intercept calls to monolith (via API Gateway)
Step 3: Gradually redirect routes from monolith to new services
Step 4: Monolith "strangled" — replaced piece by piece

┌─────────────────────────────────────────────┐
│                API Gateway                  │
└──────┬──────────────────┬───────────────────┘
       │ /orders →        │ /products →
       ↓                  ↓
┌─────────────┐   ┌──────────────────────────┐
│ Order Svc   │   │  Legacy Monolith         │
│ (new)       │   │  (handles everything     │
└─────────────┘   │   else, shrinking)       │
                  └──────────────────────────┘
```

---

## 🚫 2.2 Service Decomposition Anti-Patterns

### Nano-Services (Too Fine-Grained)
```
BAD: Separate service for every DB table
  UserEmailService (manages emails only)
  UserPhoneService (manages phones only)
  UserAddressService (manages addresses only)
  
Problems:
  - Network hop for every related operation
  - Chatty communication → high latency
  - Hard to maintain transactional integrity
  - Cognitive overhead
```

### Mega-Services (Too Coarse-Grained)
```
BAD: "User Service" manages users, auth, permissions, roles, audit, sessions
  → Becomes a mini-monolith
  → Team coordination required for every change
  → Hard to scale specific parts
```

### Shared Database Anti-Pattern
```
BAD:
  ┌─────────────┐  ┌─────────────┐
  │ Order Svc   │  │ Shipping Svc│
  └──────┬──────┘  └──────┬──────┘
         └────────┬────────┘
              ┌───┴───┐
              │Shared │
              │  DB   │  ← tight coupling!
              └───────┘
  
Problems:
  - Schema change in one service breaks others
  - Database becomes the integration layer
  - Can't scale services independently
  - Can't use different DB technologies
```

---

---

# 3. Inter-Service Communication

## 📡 3.1 Synchronous vs Asynchronous — The Core Decision

```
SYNCHRONOUS (Request-Response):
  Caller sends request → WAITS → gets response
  
  Pros:
    - Simple mental model
    - Immediate response/error feedback
    - Natural for read operations
    - Easier debugging
  
  Cons:
    - Caller blocked while waiting
    - Cascading failures (if B is down, A fails too)
    - Temporal coupling (both must be available)
    - Harder to scale
  
  Use for: queries, reads, operations needing immediate results

ASYNCHRONOUS (Message-Based):
  Caller sends message → continues (fire and forget)
  Consumer processes when ready
  
  Pros:
    - Caller not blocked
    - Temporal decoupling (consumer can be down, message waits)
    - Better fault isolation
    - Natural backpressure (queue buffers load spikes)
  
  Cons:
    - Complex mental model
    - Eventual consistency (no immediate confirmation)
    - Message ordering challenges
    - Debugging across async boundaries
  
  Use for: writes/commands, notifications, workflows, heavy operations
```

---

## 🌐 3.2 REST (HTTP) — Deep Dive

```java
// Spring Boot REST client setup
@Configuration
public class RestClientConfig {

    // Option 1: RestTemplate (blocking, legacy)
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder builder) {
        return builder
            .setConnectTimeout(Duration.ofSeconds(2))
            .setReadTimeout(Duration.ofSeconds(5))
            .errorHandler(new CustomErrorHandler())
            .build();
    }

    // Option 2: WebClient (non-blocking, recommended)
    @Bean
    public WebClient.Builder webClientBuilder() {
        return WebClient.builder()
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE);
    }

    // Option 3: OpenFeign (declarative, recommended for sync calls)
    // See Feign section below
}

// OpenFeign — declarative HTTP client (best for microservices)
@FeignClient(
    name = "inventory-service",
    url = "${services.inventory.url}",
    fallback = InventoryServiceFallback.class
)
public interface InventoryServiceClient {

    @GetMapping("/api/v1/inventory/{productId}")
    InventoryResponse getInventory(@PathVariable String productId);

    @PostMapping("/api/v1/inventory/reserve")
    ReservationResponse reserveItems(@RequestBody ReservationRequest request);

    @PutMapping("/api/v1/inventory/{reservationId}/release")
    void releaseReservation(@PathVariable String reservationId);
}

// Fallback implementation (when inventory service is down)
@Component
public class InventoryServiceFallback implements InventoryServiceClient {

    @Override
    public InventoryResponse getInventory(String productId) {
        // Return cached/default data instead of failing
        return InventoryResponse.builder()
            .productId(productId)
            .available(false)
            .stockLevel(0)
            .source("FALLBACK")
            .build();
    }

    @Override
    public ReservationResponse reserveItems(ReservationRequest request) {
        throw new ServiceUnavailableException("Inventory service unavailable");
    }
}
```

---

## ⚡ 3.3 gRPC — High Performance RPC

### Why gRPC over REST?
```
REST/JSON:
  - Text-based → larger payload
  - HTTP/1.1 → one request per connection (mostly)
  - Schema optional (OpenAPI is convention)
  - Browser native support

gRPC:
  - Binary (Protobuf) → 3-10x smaller payload, faster parsing
  - HTTP/2 → multiplexing, streaming, header compression
  - Schema mandatory (proto file = contract)
  - Bi-directional streaming support
  - Code generation (type-safe clients in any language)
  - NOT browser native (needs gRPC-Web proxy)

Use gRPC for:
  - High-throughput internal service-to-service communication
  - Low latency requirements
  - Polyglot environments (Java→Go→Python)
  - Streaming (real-time updates)
  
Use REST for:
  - External/public APIs (browser friendly)
  - Simple CRUD with less traffic
  - When schema flexibility is needed
```

```protobuf
// inventory.proto
syntax = "proto3";
package com.myapp.inventory;

option java_package = "com.myapp.inventory.grpc";
option java_outer_classname = "InventoryProto";

service InventoryService {
  rpc GetInventory (InventoryRequest) returns (InventoryResponse);
  rpc ReserveItems (ReservationRequest) returns (ReservationResponse);
  rpc WatchInventory (WatchRequest) returns (stream InventoryUpdate); // server streaming
  rpc BulkReserve (stream ReservationRequest) returns (stream ReservationResponse); // bi-di
}

message InventoryRequest {
  string product_id = 1;
}

message InventoryResponse {
  string product_id = 1;
  int32 stock_level = 2;
  bool available = 3;
  string warehouse_location = 4;
}

message ReservationRequest {
  string product_id = 1;
  int32 quantity = 2;
  string order_id = 3;
}

message ReservationResponse {
  string reservation_id = 1;
  bool success = 2;
  string message = 3;
}
```

```java
// gRPC Server (Inventory Service)
@GrpcService
@RequiredArgsConstructor
public class InventoryGrpcService extends InventoryServiceGrpc.InventoryServiceImplBase {

    private final InventoryRepository inventoryRepository;

    @Override
    public void getInventory(InventoryRequest request,
                             StreamObserver<InventoryResponse> responseObserver) {
        try {
            Inventory inventory = inventoryRepository.findByProductId(request.getProductId())
                .orElseThrow(() -> new InventoryNotFoundException(request.getProductId()));

            InventoryResponse response = InventoryResponse.newBuilder()
                .setProductId(inventory.getProductId())
                .setStockLevel(inventory.getStockLevel())
                .setAvailable(inventory.getStockLevel() > 0)
                .setWarehouseLocation(inventory.getWarehouseLocation())
                .build();

            responseObserver.onNext(response);
            responseObserver.onCompleted();
        } catch (Exception ex) {
            responseObserver.onError(Status.NOT_FOUND
                .withDescription(ex.getMessage())
                .asRuntimeException());
        }
    }
}

// gRPC Client (Order Service calling Inventory)
@Service
@RequiredArgsConstructor
public class InventoryGrpcClient {

    private final InventoryServiceGrpc.InventoryServiceBlockingStub blockingStub;

    public InventoryResponse getInventory(String productId) {
        try {
            return blockingStub
                .withDeadlineAfter(5, TimeUnit.SECONDS) // timeout
                .getInventory(InventoryRequest.newBuilder()
                    .setProductId(productId)
                    .build());
        } catch (StatusRuntimeException ex) {
            if (ex.getStatus().getCode() == Status.Code.NOT_FOUND) {
                throw new ProductNotFoundException(productId);
            }
            throw new ServiceUnavailableException("Inventory service error", ex);
        }
    }
}
```

---

## 📨 3.4 Message-Driven Communication (Async)

### When to Use Which Messaging System

| | Kafka | RabbitMQ | AWS SQS/SNS |
|---|---|---|---|
| **Model** | Log-based (pull) | Queue-based (push) | Queue/Topic |
| **Retention** | Days/weeks (replay) | Until consumed | 4 days (SQS) |
| **Throughput** | Millions/sec | Thousands/sec | Thousands/sec |
| **Ordering** | Per partition | Per queue | FIFO queues |
| **Use case** | Event streaming, audit log | Task queues, routing | AWS-native apps |
| **Consumer groups** | Yes (independent) | Competing consumers | SQS: competing |
| **Replay** | Yes | No | No |
| **Managed** | Confluent Cloud | CloudAMQP | AWS managed |

```
Kafka shines when:
  - Need to replay events (rebuild read models, new consumers)
  - High throughput (millions of events/sec)
  - Multiple independent consumers of same event
  - Event sourcing / audit trail

RabbitMQ/SQS shines when:
  - Task distribution to worker pool
  - Complex routing (topic exchanges)
  - Request-reply pattern
  - Lower volume, simpler setup
```

### Event Envelope Pattern
```java
// Standard envelope for all events (consistency across services)
@Data
@Builder
@JsonInclude(JsonInclude.Include.NON_NULL)
public class EventEnvelope<T> {
    private String eventId;           // UUID — idempotency key
    private String eventType;         // "OrderCreated", "PaymentFailed"
    private String source;            // "order-service"
    private String correlationId;     // trace events across services
    private String causationId;       // ID of event that caused this one
    private Instant occurredAt;       // when event happened
    private int version;              // schema version (for evolution)
    private T payload;                // actual event data

    public static <T> EventEnvelope<T> of(String eventType, String source, T payload) {
        return EventEnvelope.<T>builder()
            .eventId(UUID.randomUUID().toString())
            .eventType(eventType)
            .source(source)
            .occurredAt(Instant.now())
            .version(1)
            .payload(payload)
            .build();
    }
}

// Event types
public record OrderCreatedEvent(
    String orderId,
    String customerId,
    List<OrderItemDto> items,
    BigDecimal totalAmount
) {}

public record PaymentProcessedEvent(
    String paymentId,
    String orderId,
    BigDecimal amount,
    String status,
    String failureReason
) {}
```

---

## 🔄 3.5 Synchronous vs Async — Decision Framework

```
Ask these questions:

1. Does the caller need the result immediately?
   YES → Synchronous (REST/gRPC)
   NO  → Asynchronous (Kafka/RabbitMQ)

2. Can the operation fail and we retry later?
   YES → Asynchronous
   NO  → Synchronous

3. Are multiple services interested in this event?
   YES → Async (pub/sub — Kafka topic with multiple consumers)
   NO  → Synchronous (or async with single queue)

4. Is this a query or command?
   QUERY (read data) → Synchronous
   COMMAND (change state) → Consider async

Real examples:
  - "Get product details for search" → REST (query, immediate)
  - "Place an order" → REST for response, Kafka for downstream processing
  - "Order placed → send email" → Kafka (notification can be slightly delayed)
  - "Payment result → update order" → Kafka (async, but must eventually happen)
  - "Check inventory before checkout" → REST/gRPC (immediate, need result)
```

---

---

# 4. Service Discovery & Load Balancing

## 🔍 4.1 Why Service Discovery?

In microservices, services scale up/down dynamically. You can't hardcode IP addresses.

```
STATIC (hardcoded) — doesn't work:
  Order Service: "Inventory is at 192.168.1.10:8080"
  → Inventory restarts → new IP → Order Service broken!
  → Inventory scales to 3 instances → Order only talks to one!

DYNAMIC (service discovery):
  Order Service: "Who has inventory-service?"
  Service Registry: "inventory-service is at 10.0.0.5:8080, 10.0.0.6:8080, 10.0.0.7:8080"
  Order Service: picks one (load balanced)
```

---

## 📋 4.2 Client-Side vs Server-Side Discovery

### Client-Side Discovery (Eureka, Consul)
```
Client:
  1. Query registry: "Where is inventory-service?"
  2. Registry returns: [10.0.0.5:8080, 10.0.0.6:8080]
  3. Client picks one (round-robin, random, least-connections)
  4. Client calls directly

Pros:
  - No single point of failure in routing
  - Client can implement smart load balancing
  - One less network hop

Cons:
  - Each client needs discovery logic
  - Language-specific discovery client
  - Client must handle stale registry data
```

### Server-Side Discovery (AWS ALB, Kubernetes Services, Nginx)
```
Client:
  1. Client calls load balancer/gateway
  2. Load balancer queries registry or uses its own routing table
  3. Load balancer forwards to chosen instance

Pros:
  - Simple client (no discovery logic)
  - Works for any language
  - Centralized routing

Cons:
  - Load balancer is potential bottleneck
  - Extra network hop
  - Must scale the load balancer
```

---

## 🏛️ 4.3 Spring Cloud with Eureka

```yaml
# Eureka Server application.yml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false  # server doesn't register itself
    fetch-registry: false
    service-url:
      defaultZone: http://localhost:8761/eureka/
  server:
    enable-self-preservation: false  # dev only
    eviction-interval-timer-in-ms: 5000
```

```java
// Eureka Server
@SpringBootApplication
@EnableEurekaServer
public class ServiceRegistryApplication {
    public static void main(String[] args) {
        SpringApplication.run(ServiceRegistryApplication.class, args);
    }
}
```

```yaml
# Microservice (client) application.yml
spring:
  application:
    name: order-service  # this name is used for discovery

eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 30
    instance-id: ${spring.application.name}:${server.port}:${random.value}
```

```java
// Microservice (client) bootstrap
@SpringBootApplication
@EnableDiscoveryClient
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

// Using LoadBalancer with RestTemplate
@Configuration
public class RestClientConfig {
    @Bean
    @LoadBalanced  // ← tells RestTemplate to use service name, not IP
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
@RequiredArgsConstructor
public class OrderService {
    private final RestTemplate restTemplate;

    public InventoryResponse checkInventory(String productId) {
        // "inventory-service" resolved via Eureka + load balanced
        return restTemplate.getForObject(
            "http://inventory-service/api/v1/inventory/" + productId,
            InventoryResponse.class
        );
    }
}

// Using OpenFeign (preferred — auto-integrates with service discovery)
@FeignClient(name = "inventory-service") // name matches spring.application.name
public interface InventoryClient {
    @GetMapping("/api/v1/inventory/{productId}")
    InventoryResponse getInventory(@PathVariable String productId);
}
```

---

## ⚖️ 4.4 Load Balancing Strategies

```
Round Robin (default):
  Requests: R1→Instance1, R2→Instance2, R3→Instance3, R4→Instance1...
  Best for: homogeneous instances with similar load

Weighted Round Robin:
  Instance1 (4 CPU) gets 2x requests vs Instance2 (2 CPU)
  Best for: heterogeneous instances

Least Connections:
  New request → instance with fewest active connections
  Best for: varying request durations

Random:
  Simple, works well with many instances
  
IP Hash (Sticky Sessions):
  Same client IP → same instance
  Best for: stateful sessions (avoid if possible in microservices)
  Problem: uneven distribution if many behind NAT

Health-based (Kubernetes):
  Only routes to healthy instances (readiness probe passing)
```

---

## 🌐 4.5 Kubernetes Service Discovery (Production Standard)

```yaml
# In Kubernetes, Service Discovery is built-in via DNS
# Services get a DNS name: <service-name>.<namespace>.svc.cluster.local

# Service definition
apiVersion: v1
kind: Service
metadata:
  name: inventory-service
  namespace: myapp
spec:
  selector:
    app: inventory-service
  ports:
    - port: 8080
      targetPort: 8080
  type: ClusterIP  # internal only

# Order Service calls: http://inventory-service.myapp.svc.cluster.local:8080
# Or simply: http://inventory-service:8080 (within same namespace)
```

---

---

# 5. API Gateway Patterns

## 🚪 5.1 What is an API Gateway?

The API Gateway is the **single entry point** for all client requests. It sits between clients and microservices.

```
┌────────────┐
│   Clients  │ (Browser, Mobile App, Third-party)
└─────┬──────┘
      │
┌─────▼──────────────────────────────────────────────┐
│                  API Gateway                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────┐ │
│  │  Auth    │ │Rate      │ │ Request  │ │ Load  │ │
│  │  (JWT)   │ │Limiting  │ │ Routing  │ │ Bal.  │ │
│  └──────────┘ └──────────┘ └──────────┘ └───────┘ │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────┐ │
│  │  SSL     │ │ Caching  │ │ Logging  │ │ CORS  │ │
│  │  Term.   │ │          │ │ Tracing  │ │       │ │
│  └──────────┘ └──────────┘ └──────────┘ └───────┘ │
└──────┬─────────────┬───────────────┬───────────────┘
       │             │               │
       ▼             ▼               ▼
  Order Service  User Service  Product Service
```

---

## ⚙️ 5.2 Spring Cloud Gateway — Full Implementation

```yaml
# application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: order-service
          uri: lb://order-service          # lb:// = load balanced via discovery
          predicates:
            - Path=/api/v1/orders/**       # route matching
          filters:
            - RewritePath=/api/v1/orders/(?<remaining>.*), /api/v1/orders/${remaining}
            - AddRequestHeader=X-Gateway-Source, api-gateway
            - name: CircuitBreaker
              args:
                name: order-service-cb
                fallbackUri: forward:/fallback/orders
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
                key-resolver: "#{@userKeyResolver}"

        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/v1/users/**
          filters:
            - name: Retry
              args:
                retries: 3
                statuses: BAD_GATEWAY, SERVICE_UNAVAILABLE
                methods: GET
                backoff:
                  firstBackoff: 50ms
                  maxBackoff: 500ms
                  factor: 2

        - id: product-service-v2
          uri: lb://product-service-v2
          predicates:
            - Path=/api/v2/products/**    # version routing

      default-filters:
        - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
        - name: RequestSize
          args:
            maxSize: 5MB

  # Redis for rate limiting
  data:
    redis:
      host: redis
      port: 6379
```

```java
// Gateway configuration with filters
@Configuration
@EnableWebFluxSecurity
public class GatewayConfig {

    // JWT Authentication Filter
    @Bean
    public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
        return http
            .csrf(csrf -> csrf.disable())
            .authorizeExchange(auth -> auth
                .pathMatchers("/api/auth/**", "/actuator/health").permitAll()
                .pathMatchers("/api/v1/admin/**").hasRole("ADMIN")
                .anyExchange().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.jwtAuthenticationConverter(jwtConverter()))
            )
            .build();
    }

    // Rate limiting by user ID
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> exchange.getPrincipal()
            .map(Principal::getName)
            .defaultIfEmpty("anonymous");
    }

    // Custom global filter — adds correlation ID to all requests
    @Bean
    public GlobalFilter correlationIdFilter() {
        return (exchange, chain) -> {
            String correlationId = exchange.getRequest().getHeaders()
                .getFirst("X-Correlation-ID");
            if (correlationId == null) {
                correlationId = UUID.randomUUID().toString();
            }
            final String finalCorrelationId = correlationId;

            ServerWebExchange modifiedExchange = exchange.mutate()
                .request(r -> r.header("X-Correlation-ID", finalCorrelationId))
                .response(r -> {
                    r.getHeaders().add("X-Correlation-ID", finalCorrelationId);
                    return r;
                })
                .build();

            return chain.filter(modifiedExchange)
                .then(Mono.fromRunnable(() ->
                    log.info("Request: {} {} → {}ms correlationId={}",
                        exchange.getRequest().getMethod(),
                        exchange.getRequest().getURI(),
                        System.currentTimeMillis() - startTime,
                        finalCorrelationId)
                ));
        };
    }

    // Fallback controller
    @RestController
    public static class FallbackController {
        @GetMapping("/fallback/orders")
        public ResponseEntity<Map<String, String>> ordersFallback() {
            return ResponseEntity.status(503).body(Map.of(
                "error", "SERVICE_UNAVAILABLE",
                "message", "Order service is temporarily unavailable. Please try again later."
            ));
        }
    }
}
```

---

## 🔀 5.3 Backend For Frontend (BFF) Pattern

Different clients need different data shapes. BFF creates a dedicated gateway per client type.

```
Mobile App  →  Mobile BFF  → [aggregates from multiple services]
Web App     →  Web BFF     → [different aggregation, more data]
Partner API →  Partner BFF → [filtered, rate-limited API]
           
WHY:
  Mobile: needs fewer fields (bandwidth), smaller payloads, offline-first
  Web: needs rich data, server-side rendering friendly
  Partner: needs controlled, versioned, rate-limited access

EXAMPLE — Mobile BFF:
  GET /mobile/v1/home → BFF calls:
    - Product Service (top 5 featured)
    - Order Service (last 3 orders)
    - Notification Service (unread count)
    → Aggregates into single response (one network round trip for mobile)
```

---

## 📊 5.4 API Versioning Strategies

| Strategy | Example | Pros | Cons |
|----------|---------|------|------|
| URI versioning | `/api/v1/users` | Simple, explicit | Breaks REST (resource = same) |
| Header versioning | `Accept: application/vnd.myapp.v2+json` | Clean URLs | Harder to test |
| Query param | `/api/users?version=2` | Flexible | Messy, cacheability issues |
| Subdomain | `v2.api.myapp.com` | Clean separation | DNS management overhead |

**Best practice:** URI versioning for public APIs, Header versioning for internal.

```java
// URI versioning in Spring
@RestController
@RequestMapping("/api/v1/users")
public class UserControllerV1 { ... }

@RestController
@RequestMapping("/api/v2/users")
public class UserControllerV2 {
    // New fields, different response structure
}

// Header versioning
@GetMapping(value = "/users", headers = "API-Version=1")
public List<UserResponseV1> getUsersV1() { ... }

@GetMapping(value = "/users", headers = "API-Version=2")
public List<UserResponseV2> getUsersV2() { ... }
```

---

---

# 6. Distributed Data Management

## 🗄️ 6.1 Database Per Service — Patterns

### Shared Database (Anti-pattern)
```
Problem: All services sharing one DB creates tight coupling
  - Schema changes affect all services
  - One service's bad query degrades others
  - Can't scale services independently
```

### Database Per Service (Correct approach)
```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Order Svc   │  │ Product Svc │  │Customer Svc │
│  ┌────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │
│  │Postgres│ │  │ │ElasticS.│ │  │ │ MongoDB │ │
│  └────────┘ │  │ └─────────┘ │  │ └─────────┘ │
└─────────────┘  └─────────────┘  └─────────────┘
```

**Choosing the right DB per service:**
```
Service              → Best DB
Order Service        → PostgreSQL (ACID, complex queries)
Product Search       → Elasticsearch (full-text search)
User Sessions        → Redis (fast key-value, TTL)
Social Graph         → Neo4j (graph relationships)
Time-series Metrics  → InfluxDB / TimescaleDB
Document Storage     → MongoDB (flexible schema)
Audit/Event Log      → Kafka + S3 (append-only, cheap)
```

---

## 🔗 6.2 Data Consistency Patterns

### Problem: Joining Data Across Services
```
Order Service needs: Order + Customer Name + Product Details
  - Order data: in Orders DB ✓
  - Customer Name: in Customers DB ✗ (different service)
  - Product Details: in Products DB ✗ (different service)
  
NO JOINS across service databases!
```

### Pattern 1: API Composition (Synchronous)
```java
// Order service composes data from multiple service calls
@Service
@RequiredArgsConstructor
public class OrderCompositionService {

    private final OrderRepository orderRepository;
    private final CustomerServiceClient customerClient;
    private final ProductServiceClient productClient;

    public OrderDetailResponse getOrderDetail(String orderId) {
        Order order = orderRepository.findById(orderId).orElseThrow();

        // Parallel calls to other services
        CompletableFuture<CustomerDto> customerFuture =
            CompletableFuture.supplyAsync(() -> customerClient.getCustomer(order.getCustomerId()));

        CompletableFuture<List<ProductDto>> productsFuture =
            CompletableFuture.supplyAsync(() ->
                productClient.getProducts(
                    order.getItems().stream()
                        .map(OrderItem::getProductId)
                        .collect(Collectors.toList())
                )
            );

        CompletableFuture.allOf(customerFuture, productsFuture).join();

        return OrderDetailResponse.builder()
            .orderId(order.getId())
            .customer(customerFuture.join())
            .items(enrichItems(order.getItems(), productsFuture.join()))
            .total(order.getTotalAmount())
            .status(order.getStatus())
            .build();
    }
    
    // Tradeoffs:
    // ✅ Consistent data (fetched at query time)
    // ❌ Latency = sum of slow services (even with parallelism)
    // ❌ Availability = product of all service availabilities (all must be up)
    // ❌ Cascade failures
}
```

### Pattern 2: CQRS with Data Replication (Async)
```
When Order is placed, Order Service publishes event:
OrderCreated { orderId, customerId, customerName, products[{id, name, price}] }

Read Model Service consumes event:
→ Builds denormalized read model: { orderId, customerName, productNames, total }
→ Stores in its own read DB (pre-joined data!)

Query:
→ Read from read DB — no cross-service calls needed!
→ Fast reads, no availability coupling

Tradeoff: eventual consistency (read model may lag by ms/seconds)
→ Acceptable for most read use cases
```

### Pattern 3: Event-Driven Data Replication
```
Customer Service publishes CustomerUpdated event:
→ Order Service consumes and caches customer data it needs
→ Order Service stores {customerId, customerName} snapshot in its own DB

Order query: all data local → fast, no cross-service calls

Tradeoff: stale data (customer name may be slightly outdated in orders)
→ Usually acceptable (orders are historical snapshots anyway)
```

---

## 💥 6.3 Distributed Transactions — The Big Problem

```
Problem: Place Order requires:
  1. Order Service: create order record
  2. Inventory Service: reserve items
  3. Payment Service: charge customer
  4. Notification Service: send confirmation

All must succeed or all must rollback.
You CANNOT use a single ACID transaction (different DBs, different services).
```

### Two-Phase Commit (2PC) — Avoid in Microservices
```
Phase 1 (Prepare):
  Coordinator asks all: "Can you commit?"
  All reply: "Yes, prepared" or "No, abort"

Phase 2 (Commit/Abort):
  If all said yes → "Commit!"
  If any said no → "Abort!"

Problems:
  - Coordinator is single point of failure
  - Participants blocked (locks held) during coordinator failure
  - Doesn't work well across microservices
  - Long-held locks hurt performance
  → DO NOT USE across microservices
```

### Saga Pattern — The Solution (Covered in depth in Section 8)
```
Break distributed transaction into sequence of local transactions,
each publishing events/messages to trigger next step.
Compensation transactions undo work if something fails.
→ Eventual consistency instead of ACID
```

---

## 🌐 6.4 CQRS — Command Query Responsibility Segregation

Separate the **write model** (commands) from the **read model** (queries).

```
Traditional (same model for reads and writes):
  OrderRepository.save(order)     ← write
  OrderRepository.findById(id)    ← read (same model, same DB)

CQRS:
  Command Side:
    OrderCommandRepository.save(order)  ← normalized, write-optimized

  Query Side:
    OrderQueryRepository.findDetailById(id) ← denormalized, read-optimized
    (pre-joined data, optimized for specific query patterns)
```

```java
// Command side — write-optimized, normalized
@Entity
@Table(name = "orders")
public class Order {
    @Id private String id;
    private String customerId; // just the ID
    private OrderStatus status;

    @OneToMany(cascade = CascadeType.ALL)
    private List<OrderItem> items;
}

// Query side — read-optimized, denormalized view
@Document(collection = "order_views") // MongoDB for flexible read models
public class OrderView {
    @Id private String orderId;

    // Denormalized — no join needed!
    private String customerName;
    private String customerEmail;

    private List<OrderItemView> items; // includes productName, imageUrl

    private BigDecimal subtotal;
    private BigDecimal tax;
    private BigDecimal total;

    private String statusLabel; // "In Transit", "Delivered" (human-readable)
    private String estimatedDelivery;

    private Instant lastUpdated;
}

// Event handler builds/maintains read model
@Component
@KafkaListener(topics = {"order-created", "order-updated", "order-shipped"})
public class OrderViewProjection {

    private final OrderViewRepository viewRepository;
    private final CustomerServiceClient customerClient;

    public void handle(EventEnvelope<?> event) {
        switch (event.getEventType()) {
            case "OrderCreated" -> {
                OrderCreatedEvent e = (OrderCreatedEvent) event.getPayload();
                CustomerDto customer = customerClient.getCustomer(e.getCustomerId());

                OrderView view = OrderView.builder()
                    .orderId(e.getOrderId())
                    .customerName(customer.getName())
                    .customerEmail(customer.getEmail())
                    .items(mapItems(e.getItems()))
                    .total(e.getTotalAmount())
                    .statusLabel("Order Placed")
                    .lastUpdated(Instant.now())
                    .build();

                viewRepository.save(view);
            }
            case "OrderShipped" -> {
                OrderShippedEvent e = (OrderShippedEvent) event.getPayload();
                viewRepository.findById(e.getOrderId()).ifPresent(view -> {
                    view.setStatusLabel("In Transit");
                    view.setEstimatedDelivery(e.getEstimatedDelivery());
                    viewRepository.save(view);
                });
            }
        }
    }
}
```

---

---

# 7. Resilience Patterns

## 💡 7.1 Why Resilience?

In a system of 20 microservices, each with 99.9% availability:
`System availability = 0.999^20 = 98%` — 2% downtime!

Services WILL fail. The question is: **will one failure cascade to bring down everything?**

```
WITHOUT resilience patterns:
  Payment Service down →
  Order Service hangs waiting for Payment →
  Thread pool exhausted in Order Service →
  API Gateway times out →
  All users see 500 errors
  (cascade failure / "avalanche")

WITH resilience patterns:
  Payment Service down →
  Circuit Breaker opens →
  Order Service immediately returns fallback →
  Users see "Payment temporarily unavailable, try again"
  (degraded but functional)
```

---

## ⚡ 7.2 Circuit Breaker — The Most Important Pattern

```
States:
CLOSED (normal):
  → Requests flow through
  → Count failures (errors, timeouts)
  → If failures > threshold → trip to OPEN

OPEN (failing):
  → Requests immediately fail (don't even try)
  → Return fallback response
  → After timeout → go to HALF-OPEN

HALF-OPEN (testing):
  → Allow limited requests through (probe)
  → If probes succeed → go to CLOSED
  → If probes fail → go back to OPEN
```

```java
// Resilience4j Circuit Breaker with Spring Boot
// pom.xml: spring-boot-starter-aop, resilience4j-spring-boot3

// application.yml
resilience4j:
  circuitbreaker:
    instances:
      inventory-service:
        sliding-window-type: COUNT_BASED      # or TIME_BASED
        sliding-window-size: 10               # last 10 calls
        failure-rate-threshold: 50            # 50% failure rate → OPEN
        slow-call-rate-threshold: 80          # 80% slow calls → OPEN
        slow-call-duration-threshold: 3s      # "slow" = >3 seconds
        wait-duration-in-open-state: 30s      # stay OPEN for 30s then probe
        permitted-number-of-calls-in-half-open-state: 5
        minimum-number-of-calls: 5            # need at least 5 calls before evaluating
        record-exceptions:
          - java.io.IOException
          - java.util.concurrent.TimeoutException
          - feign.FeignException.ServiceUnavailable
        ignore-exceptions:
          - com.myapp.exception.BusinessException # don't count business errors

  timelimiter:
    instances:
      inventory-service:
        timeout-duration: 3s

  retry:
    instances:
      inventory-service:
        max-attempts: 3
        wait-duration: 500ms
        retry-exceptions:
          - java.io.IOException
        ignore-exceptions:
          - com.myapp.exception.UserNotFoundException

// Service with Circuit Breaker
@Service
@RequiredArgsConstructor
@Slf4j
public class InventoryClientService {

    private final InventoryServiceClient inventoryClient;
    private final InventoryCache inventoryCache;

    @CircuitBreaker(name = "inventory-service", fallbackMethod = "getInventoryFallback")
    @Retry(name = "inventory-service")
    @TimeLimiter(name = "inventory-service")
    public CompletableFuture<InventoryResponse> getInventory(String productId) {
        return CompletableFuture.supplyAsync(() ->
            inventoryClient.getInventory(productId)
        );
    }

    // Fallback — must have same return type + Throwable param
    public CompletableFuture<InventoryResponse> getInventoryFallback(
        String productId, Throwable ex
    ) {
        log.warn("Circuit breaker triggered for inventory-service: {}", ex.getMessage());

        // Strategy 1: Return cached data
        return CompletableFuture.supplyAsync(() ->
            inventoryCache.getLastKnown(productId)
                .orElse(InventoryResponse.unknown(productId))
        );
    }

    // Monitoring circuit breaker state
    @EventListener
    public void onCircuitBreakerStateChange(CircuitBreakerOnStateTransitionEvent event) {
        log.warn("Circuit Breaker '{}': {} → {}",
            event.getCircuitBreakerName(),
            event.getStateTransition().getFromState(),
            event.getStateTransition().getToState()
        );
        // Alert on-call engineer if circuit breaker opens
        alertService.sendAlert("Circuit breaker open: " + event.getCircuitBreakerName());
    }
}
```

---

## 🪣 7.3 Bulkhead Pattern — Isolate Failures

Like watertight compartments in a ship — one flooding compartment doesn't sink the whole ship.

```
WITHOUT Bulkhead:
  All calls share one thread pool
  → Slow Payment Service blocks all threads
  → Order Service, Inventory Service also starved of threads
  → Everything fails

WITH Bulkhead:
  Payment calls → Payment ThreadPool (10 threads)
  Inventory calls → Inventory ThreadPool (20 threads)
  Order processing → Order ThreadPool (50 threads)
  → Payment slow → only Payment pool exhausted
  → Other services unaffected
```

```java
// Thread pool bulkhead configuration
resilience4j:
  bulkhead:
    instances:
      payment-service:
        max-concurrent-calls: 10      # max parallel calls to payment
        max-wait-duration: 100ms      # wait up to 100ms for a slot, then reject

  thread-pool-bulkhead:
    instances:
      inventory-service:
        max-thread-pool-size: 20
        core-thread-pool-size: 10
        queue-capacity: 50
        keep-alive-duration: 20ms

@Service
public class PaymentClientService {

    @Bulkhead(name = "payment-service", type = Bulkhead.Type.SEMAPHORE,
              fallbackMethod = "paymentFallback")
    public PaymentResponse processPayment(PaymentRequest request) {
        return paymentClient.processPayment(request);
    }

    public PaymentResponse paymentFallback(PaymentRequest request, BulkheadFullException ex) {
        throw new ServiceBusyException("Payment service is at capacity. Please retry.");
    }
}
```

---

## ⏱️ 7.4 Timeout Pattern

```java
// Never make a call without a timeout!
// Default: wait forever → threads hang → thread pool exhausted

// Feign with timeout
@FeignClient(
    name = "payment-service",
    configuration = PaymentServiceFeignConfig.class
)
public interface PaymentServiceClient { ... }

public class PaymentServiceFeignConfig {
    @Bean
    public Request.Options requestOptions() {
        return new Request.Options(
            2, TimeUnit.SECONDS,   // connect timeout
            5, TimeUnit.SECONDS,   // read timeout
            true                   // follow redirects
        );
    }
}

// WebClient with timeout
public Mono<PaymentResponse> processPayment(PaymentRequest request) {
    return webClient.post()
        .uri("/api/v1/payments")
        .bodyValue(request)
        .retrieve()
        .bodyToMono(PaymentResponse.class)
        .timeout(Duration.ofSeconds(5))
        .onErrorMap(TimeoutException.class,
            ex -> new ServiceTimeoutException("Payment service timed out"));
}
```

---

## 🔁 7.5 Retry with Exponential Backoff

```java
// application.yml
resilience4j:
  retry:
    instances:
      external-api:
        max-attempts: 4
        wait-duration: 500ms
        enable-exponential-backoff: true
        exponential-backoff-multiplier: 2
        # Retry intervals: 500ms, 1s, 2s → total max wait: 3.5s
        retry-exceptions:
          - java.io.IOException
          - org.springframework.web.client.HttpServerErrorException
        ignore-exceptions:
          - com.myapp.exception.InvalidRequestException  # don't retry 4xx errors!

// KEY RULE: Only retry IDEMPOTENT operations!
// Safe to retry: GET (read), PUT (replace), DELETE (already deleted = ok)
// NOT safe: POST (may create duplicates), non-idempotent operations

// Making non-idempotent operations safe: Idempotency Key
@PostMapping("/api/v1/payments")
public PaymentResponse processPayment(
    @RequestBody PaymentRequest request,
    @RequestHeader("Idempotency-Key") String idempotencyKey
) {
    // Check if this key was already processed
    return idempotencyStore.computeIfAbsent(idempotencyKey, key -> {
        PaymentResponse response = paymentProcessor.process(request);
        return response; // stored in Redis with TTL
    });
    // Client retries with same key → gets same response (no double charge!)
}
```

---

## 🎯 Resilience Interview Questions

**Q: What is the difference between Circuit Breaker and Retry?**
- **Retry**: handles transient failures — keeps trying the same call (with backoff)
- **Circuit Breaker**: handles persistent failures — stops trying after threshold, gives service time to recover, prevents cascade failures

They work together: Retry → Circuit Breaker. Retry tries a few times, if still failing, Circuit Breaker trips and stops all attempts.

**Q: When should you NOT retry?**
- Non-idempotent operations (POST that creates a record) — may cause duplicates
- 4xx errors (BadRequest, Unauthorized) — retrying won't fix a bad request
- When Circuit Breaker is open — retrying into an open circuit wastes resources
- When downstream failure is due to YOUR bad input

**Q: Explain the Bulkhead pattern with a real example.**
A hotel has separate elevators for guests and service staff. If guest elevator breaks (full of people), service operations still work. In microservices: Payment calls use a separate thread pool from Inventory calls. Payment service slowness only exhausts the Payment thread pool — Inventory calls remain fast and functional.

---

---

# 8. Event-Driven Architecture — CQRS & Saga

## 📨 8.1 Event-Driven Architecture Patterns

### Event Types
```
Domain Event: something that happened in the domain
  OrderPlaced, PaymentFailed, ProductOutOfStock
  → Describes PAST (immutable fact)

Integration Event: cross-service communication
  Published to message broker for other services to consume
  
Command: instruction to do something
  PlaceOrder, ProcessPayment, ShipOrder
  → Single recipient, may be rejected
  
Query: request for data
  GetOrderStatus, FindUserById
  → Single recipient, read-only
```

---

## 🔄 8.2 Saga Pattern — Distributed Transaction Coordination

A Saga is a sequence of local transactions where each step publishes an event/message to trigger the next. On failure, **compensation transactions** undo previous steps.

### Choreography Saga (Event-driven, decentralized)
```
No central coordinator — each service reacts to events and publishes its own.

┌─────────────┐  OrderPlaced  ┌─────────────┐
│ Order Svc   │ ──────────→  │Inventory Svc│
│ Creates     │               │ Reserves    │
│ order       │               │ items       │
└─────────────┘               └──────┬──────┘
                                      │ ItemsReserved
                                      ↓
                              ┌─────────────┐
                              │ Payment Svc │
                              │ Charges     │
                              │ customer    │
                              └──────┬──────┘
                                      │ PaymentProcessed
                                      ↓
                              ┌─────────────┐
                              │ Order Svc   │
                              │ Updates     │
                              │ status      │
                              └─────────────┘

FAILURE PATH:
If Payment fails → publishes PaymentFailed
→ Inventory Service listens → releases reservation (compensation)
→ Order Service listens → marks order as failed

Pros:
  ✅ Simple — no central coordinator
  ✅ Loosely coupled — services only know about events
  ✅ Scales well

Cons:
  ❌ Hard to track overall saga state
  ❌ Risk of cyclic dependencies (Service A listens to Service B listens to Service A)
  ❌ Testing is complex
  ❌ Compensating transactions must be carefully designed
```

### Orchestration Saga (Centralized coordinator)
```
Saga Orchestrator controls the flow — tells each service what to do.

┌────────────────────────────────────────────────┐
│               Order Saga Orchestrator           │
│  Step 1: Reserve Inventory  →  Inventory Svc   │
│  Step 2: Process Payment    →  Payment Svc     │
│  Step 3: Notify Customer    →  Notification Svc│
│  Step 4: Confirm Order      →  Order Svc       │
└────────────────────────────────────────────────┘

FAILURE PATH:
  Step 2 (Payment) fails →
  Orchestrator runs compensations in reverse:
    Compensate Step 1: Release Inventory reservation
    Mark Order as failed
    Notify customer of failure

Pros:
  ✅ Clear flow visibility (saga state in one place)
  ✅ Easier to implement complex rollback logic
  ✅ Easy to test orchestrator logic
  ✅ No cyclic dependencies

Cons:
  ❌ Central point → potential bottleneck
  ❌ Orchestrator knows about all services (some coupling)
  ❌ More initial complexity
```

### Complete Orchestration Saga Implementation
```java
// Saga state
public enum OrderSagaStatus {
    STARTED, INVENTORY_RESERVED, PAYMENT_PROCESSED, COMPLETED,
    INVENTORY_FAILED, PAYMENT_FAILED, COMPENSATION_STARTED, COMPENSATED
}

// Saga state persisted in DB for durability
@Entity
@Table(name = "order_sagas")
@Data
public class OrderSaga {
    @Id private String sagaId;
    private String orderId;
    private OrderSagaStatus status;
    private String inventoryReservationId;
    private String paymentId;
    private int retryCount;
    private Instant createdAt;
    private Instant updatedAt;
    private String failureReason;
}

// Saga orchestrator
@Service
@RequiredArgsConstructor
@Slf4j
@Transactional
public class OrderSagaOrchestrator {

    private final OrderSagaRepository sagaRepository;
    private final KafkaTemplate<String, EventEnvelope<?>> kafkaTemplate;

    // Step 1: Start saga when order is created
    public void startSaga(String orderId, OrderCreatedEvent orderEvent) {
        OrderSaga saga = new OrderSaga();
        saga.setSagaId(UUID.randomUUID().toString());
        saga.setOrderId(orderId);
        saga.setStatus(OrderSagaStatus.STARTED);
        saga.setCreatedAt(Instant.now());
        sagaRepository.save(saga);

        // Publish command to inventory service
        kafkaTemplate.send("inventory-commands",
            EventEnvelope.of("ReserveInventory", "order-saga",
                ReserveInventoryCommand.builder()
                    .sagaId(saga.getSagaId())
                    .orderId(orderId)
                    .items(orderEvent.getItems())
                    .build()
            )
        );

        log.info("Saga started: sagaId={}, orderId={}", saga.getSagaId(), orderId);
    }

    // Step 2: Inventory reserved → process payment
    @KafkaListener(topics = "inventory-events")
    public void onInventoryEvent(EventEnvelope<InventoryReservationResult> envelope) {
        InventoryReservationResult result = envelope.getPayload();
        OrderSaga saga = sagaRepository.findBySagaId(result.getSagaId()).orElseThrow();

        if (result.isSuccess()) {
            saga.setInventoryReservationId(result.getReservationId());
            saga.setStatus(OrderSagaStatus.INVENTORY_RESERVED);
            sagaRepository.save(saga);

            // Trigger payment
            kafkaTemplate.send("payment-commands",
                EventEnvelope.of("ProcessPayment", "order-saga",
                    ProcessPaymentCommand.builder()
                        .sagaId(saga.getSagaId())
                        .orderId(saga.getOrderId())
                        .amount(result.getTotalAmount())
                        .build()
                )
            );
        } else {
            saga.setStatus(OrderSagaStatus.INVENTORY_FAILED);
            saga.setFailureReason(result.getFailureReason());
            sagaRepository.save(saga);

            // Compensate: cancel order
            publishOrderFailed(saga, "Inventory unavailable: " + result.getFailureReason());
        }
    }

    // Step 3: Payment processed → complete order
    @KafkaListener(topics = "payment-events")
    public void onPaymentEvent(EventEnvelope<PaymentResult> envelope) {
        PaymentResult result = envelope.getPayload();
        OrderSaga saga = sagaRepository.findBySagaId(result.getSagaId()).orElseThrow();

        if (result.isSuccess()) {
            saga.setPaymentId(result.getPaymentId());
            saga.setStatus(OrderSagaStatus.COMPLETED);
            sagaRepository.save(saga);

            // Complete the saga!
            kafkaTemplate.send("order-commands",
                EventEnvelope.of("ConfirmOrder", "order-saga",
                    ConfirmOrderCommand.builder()
                        .orderId(saga.getOrderId())
                        .paymentId(result.getPaymentId())
                        .build()
                )
            );
        } else {
            saga.setStatus(OrderSagaStatus.PAYMENT_FAILED);
            sagaRepository.save(saga);

            // Compensate: release inventory, cancel order
            startCompensation(saga, result.getFailureReason());
        }
    }

    private void startCompensation(OrderSaga saga, String reason) {
        saga.setStatus(OrderSagaStatus.COMPENSATION_STARTED);
        sagaRepository.save(saga);

        // Release inventory reservation
        if (saga.getInventoryReservationId() != null) {
            kafkaTemplate.send("inventory-commands",
                EventEnvelope.of("ReleaseReservation", "order-saga",
                    ReleaseReservationCommand.of(saga.getInventoryReservationId())
                )
            );
        }

        publishOrderFailed(saga, reason);
    }
}
```

---

## 📦 8.3 Event Sourcing

Instead of storing current state, store a **sequence of events** (the full history).

```
Traditional (State-based):
  orders table: { id, status: "SHIPPED", total: 150, updatedAt: ... }
  (history lost — we only know current state)

Event Sourcing:
  events table:
    { orderId, type: "OrderPlaced",  data: {...}, timestamp: t1 }
    { orderId, type: "PaymentTaken", data: {...}, timestamp: t2 }
    { orderId, type: "OrderShipped", data: {...}, timestamp: t3 }

  Current state = replay all events from beginning
  History completely preserved!
```

```java
// Event store
@Entity
@Table(name = "domain_events")
public class DomainEventRecord {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String aggregateId;    // orderId

    @Column(nullable = false)
    private String aggregateType;  // "Order"

    @Column(nullable = false)
    private String eventType;      // "OrderPlaced"

    @Column(columnDefinition = "jsonb", nullable = false)
    private String eventData;      // JSON payload

    @Column(nullable = false)
    private Long version;          // sequence number for this aggregate

    @Column(nullable = false)
    private Instant occurredAt;
}

// Order aggregate with event sourcing
public class Order {
    private String id;
    private OrderStatus status;
    private List<OrderItem> items;
    private BigDecimal total;
    private final List<DomainEvent> uncommittedEvents = new ArrayList<>();

    // Apply events to rebuild state
    public void apply(OrderPlacedEvent event) {
        this.id = event.getOrderId();
        this.items = event.getItems();
        this.total = event.getTotal();
        this.status = OrderStatus.PENDING;
    }

    public void apply(OrderShippedEvent event) {
        this.status = OrderStatus.SHIPPED;
    }

    public void apply(OrderCancelledEvent event) {
        this.status = OrderStatus.CANCELLED;
    }

    // Rebuild from event history
    public static Order reconstitute(List<DomainEvent> events) {
        Order order = new Order();
        events.forEach(event -> {
            if (event instanceof OrderPlacedEvent e) order.apply(e);
            else if (event instanceof OrderShippedEvent e) order.apply(e);
            else if (event instanceof OrderCancelledEvent e) order.apply(e);
        });
        return order;
    }
}

// Benefits:
// ✅ Complete audit trail (every change recorded)
// ✅ Time-travel queries (what was the state at time T?)
// ✅ Event replay (rebuild projections from scratch)
// ✅ Easy debugging (replay events to reproduce bugs)

// Drawbacks:
// ❌ Complex implementation
// ❌ Query complexity (must project events into queryable form)
// ❌ Event schema evolution (old events must still be processable)
// ❌ Storage grows indefinitely (snapshots help)
```

---

## 📬 8.4 Outbox Pattern — Guaranteed Event Publishing

```
Problem: Service saves to DB AND publishes to Kafka.
What if save succeeds but Kafka publish fails?
→ DB updated, event never published → inconsistency!

What if Kafka publish succeeds but DB save fails?
→ Event published for something that didn't actually happen!

OUTBOX PATTERN — atomic write + reliable publishing:

Step 1: Save to DB AND write event to "outbox" table IN SAME TRANSACTION
  BEGIN;
    INSERT INTO orders (...) VALUES (...);
    INSERT INTO outbox_events (type, payload, status) VALUES ('OrderCreated', '...', 'PENDING');
  COMMIT;

Step 2: Separate process reads outbox table, publishes to Kafka, marks as SENT

Result: DB save and event are ALWAYS consistent (same transaction)
        Publishing is at-least-once (safe with idempotent consumers)
```

```java
@Entity
@Table(name = "outbox_events")
public class OutboxEvent {
    @Id private String id;
    private String aggregateType;
    private String aggregateId;
    private String eventType;

    @Column(columnDefinition = "jsonb")
    private String payload;

    private OutboxStatus status; // PENDING, PUBLISHED, FAILED

    private Instant createdAt;
    private Instant publishedAt;
    private int retryCount;
}

@Service
@RequiredArgsConstructor
@Transactional // both DB + outbox in same transaction
public class OrderService {

    private final OrderRepository orderRepository;
    private final OutboxEventRepository outboxRepository;
    private final ObjectMapper objectMapper;

    public OrderResponse placeOrder(OrderRequest request) {
        Order order = createOrder(request);
        orderRepository.save(order);

        // Write event to outbox IN SAME TRANSACTION
        OutboxEvent outboxEvent = OutboxEvent.builder()
            .id(UUID.randomUUID().toString())
            .aggregateType("Order")
            .aggregateId(order.getId())
            .eventType("OrderCreated")
            .payload(objectMapper.writeValueAsString(
                OrderCreatedEvent.from(order)
            ))
            .status(OutboxStatus.PENDING)
            .createdAt(Instant.now())
            .build();
        outboxRepository.save(outboxEvent);

        return OrderResponse.from(order);
    }
}

// Outbox publisher — polls for unpublished events
@Component
@RequiredArgsConstructor
@Slf4j
public class OutboxEventPublisher {

    private final OutboxEventRepository outboxRepository;
    private final KafkaTemplate<String, String> kafkaTemplate;

    @Scheduled(fixedDelay = 100) // every 100ms
    @Transactional
    public void publishPendingEvents() {
        List<OutboxEvent> pending = outboxRepository
            .findTop100ByStatusOrderByCreatedAtAsc(OutboxStatus.PENDING);

        for (OutboxEvent event : pending) {
            try {
                kafkaTemplate.send(
                    getTopicForEvent(event.getEventType()),
                    event.getAggregateId(),
                    event.getPayload()
                ).get(5, TimeUnit.SECONDS); // wait for ack

                event.setStatus(OutboxStatus.PUBLISHED);
                event.setPublishedAt(Instant.now());
            } catch (Exception ex) {
                log.error("Failed to publish outbox event {}: {}", event.getId(), ex.getMessage());
                event.setRetryCount(event.getRetryCount() + 1);
                if (event.getRetryCount() >= 5) {
                    event.setStatus(OutboxStatus.FAILED);
                }
            }
            outboxRepository.save(event);
        }
    }
}
```

---

---

# 9. Security in Microservices

## 🔐 9.1 Authentication Patterns

### Pattern 1: JWT at Gateway (Most Common)
```
Client → API Gateway (validates JWT) → Services (trust gateway)
                ↓
        X-User-ID: 123
        X-User-Roles: ADMIN,USER
        X-Tenant-ID: acme-corp
        
Services receive pre-validated user info in headers.
No JWT validation in every service — faster, simpler.

Risk: If service is exposed directly (bypass gateway) → no auth!
Mitigation: Network policy (services unreachable except from gateway)
```

```java
// Gateway validates JWT, adds user context headers
@Component
public class JwtToHeadersGatewayFilter implements GatewayFilter {

    private final JwtDecoder jwtDecoder;

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String authHeader = exchange.getRequest().getHeaders().getFirst("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            return chain.filter(exchange);
        }

        Jwt jwt = jwtDecoder.decode(authHeader.substring(7));

        ServerWebExchange mutatedExchange = exchange.mutate()
            .request(req -> req
                .header("X-User-ID", jwt.getSubject())
                .header("X-User-Email", jwt.getClaimAsString("email"))
                .header("X-User-Roles", String.join(",", jwt.getClaimAsStringList("roles")))
                .header("X-Tenant-ID", jwt.getClaimAsString("tenantId"))
                .headers(headers -> headers.remove("Authorization")) // don't forward raw JWT
            )
            .build();

        return chain.filter(mutatedExchange);
    }
}

// Downstream service reads user context from headers
@RestController
@RequestMapping("/api/v1/orders")
public class OrderController {

    @GetMapping
    public List<OrderResponse> getMyOrders(
        @RequestHeader("X-User-ID") String userId,
        @RequestHeader("X-User-Roles") String roles
    ) {
        return orderService.getOrdersForUser(userId);
    }
}

// Or inject via custom annotation
@Component
public class CurrentUserResolver implements HandlerMethodArgumentResolver {
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUser.class);
    }

    @Override
    public Object resolveArgument(MethodParameter parameter, ...,
                                  NativeWebRequest webRequest, ...) {
        HttpServletRequest request = (HttpServletRequest) webRequest.getNativeRequest();
        return UserContext.builder()
            .userId(request.getHeader("X-User-ID"))
            .email(request.getHeader("X-User-Email"))
            .roles(Arrays.asList(request.getHeader("X-User-Roles").split(",")))
            .tenantId(request.getHeader("X-Tenant-ID"))
            .build();
    }
}

// Usage in controllers
@GetMapping
public List<OrderResponse> getMyOrders(@CurrentUser UserContext user) {
    return orderService.getOrdersForUser(user.getUserId());
}
```

### Pattern 2: Service-to-Service Authentication (mTLS)
```
Problem: Service B receives request claiming to be from Service A.
How does Service B know it's really Service A and not a malicious actor?

Solution: Mutual TLS (mTLS)
  - Service A has a client certificate (signed by your CA)
  - Service B validates the certificate
  - Both sides authenticate each other
  - In Kubernetes: handled by service mesh (Istio/Linkerd) automatically
```

### Pattern 3: OAuth2 / OIDC
```
Use when:
  - Users federated from external identity provider (Google, Okta, Azure AD)
  - Fine-grained scopes needed (read:orders, write:products)
  - Token refresh, revocation needed

Flow:
  1. User → Auth Service → gets JWT (access_token + refresh_token)
  2. User → API Gateway with access_token
  3. Gateway validates token against JWKS endpoint
  4. Gateway forwards user info to services
```

---

## 🔒 9.2 Secrets Management

```yaml
# NEVER do this (secrets in application.yml committed to Git!):
spring:
  datasource:
    password: myS3cretP@ssword123  # ← NEVER!

# DO THIS — externalize secrets:

# Option 1: Environment Variables (simple, 12-factor)
spring:
  datasource:
    password: ${DB_PASSWORD}  # inject from environment

# Option 2: AWS Secrets Manager (Production recommended)
# pom: spring-cloud-starter-aws-secrets-manager-config
aws:
  secretsmanager:
    secret-name: /myapp/prod/database
    # Secret value: {"username": "admin", "password": "secret"}
spring:
  datasource:
    username: ${username}     # auto-resolved from Secrets Manager
    password: ${password}

# Option 3: HashiCorp Vault
spring:
  cloud:
    vault:
      host: vault.myapp.com
      port: 8200
      token: ${VAULT_TOKEN}
      kv:
        enabled: true
        backend: secret
        application-name: order-service
```

---

## 🛡️ 9.3 Multi-Tenancy Patterns

```java
// Tenant isolation strategies:

// 1. Schema per tenant (separate schema in same DB)
@Component
public class TenantSchemaInterceptor implements PhysicalNamingStrategy {
    @Override
    public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment context) {
        String tenantId = TenantContext.getCurrentTenant();
        return Identifier.toIdentifier(tenantId + "_" + name.getText());
    }
}

// 2. Row-level tenant isolation (Postgres RLS)
// Hibernate Filter for automatic tenant filtering
@Entity
@FilterDef(name = "tenantFilter", parameters = @ParamDef(name = "tenantId", type = String.class))
@Filter(name = "tenantFilter", condition = "tenant_id = :tenantId")
public class Order {
    @Column(nullable = false)
    private String tenantId;  // every row has tenant_id
}

// Enable filter automatically in every request
@Component
public class TenantFilterAspect {
    @Autowired
    private EntityManager entityManager;

    @Before("@annotation(org.springframework.web.bind.annotation.RequestMapping)")
    public void applyTenantFilter(JoinPoint joinPoint) {
        Session session = entityManager.unwrap(Session.class);
        session.enableFilter("tenantFilter")
               .setParameter("tenantId", TenantContext.getCurrentTenant());
    }
}
```

---

---

# 10. Deployment — Docker & Kubernetes

## 🐳 10.1 Docker — Optimized Spring Boot Image

```dockerfile
# Multi-stage build — optimized image
# Stage 1: Build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY mvnw .
COPY .mvn .mvn
COPY pom.xml .

# Download dependencies (cached layer if pom.xml unchanged)
RUN ./mvnw dependency:go-offline -q

COPY src src
RUN ./mvnw package -DskipTests -q

# Extract layers (Spring Boot layered jars for smaller image updates)
RUN java -Djarmode=layertools -jar target/*.jar extract

# Stage 2: Runtime — minimal image
FROM eclipse-temurin:21-jre-alpine

# Security: don't run as root
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

WORKDIR /app

# Copy layers separately (better Docker cache reuse)
COPY --from=builder /app/dependencies/ ./
COPY --from=builder /app/spring-boot-loader/ ./
COPY --from=builder /app/snapshot-dependencies/ ./
COPY --from=builder /app/application/ ./

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD wget -q --spider http://localhost:8080/actuator/health || exit 1

EXPOSE 8080

# JVM tuning for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport \
               -XX:MaxRAMPercentage=75.0 \
               -XX:+UseG1GC \
               -XX:MaxGCPauseMillis=200 \
               -Djava.security.egd=file:/dev/./urandom"

ENTRYPOINT ["sh", "-c", "java ${JAVA_OPTS} org.springframework.boot.loader.launch.JarLauncher"]
```

```yaml
# docker-compose.yml for local development
version: '3.8'

services:
  order-service:
    build: ./order-service
    ports:
      - "8081:8080"
    environment:
      SPRING_PROFILES_ACTIVE: dev
      DB_URL: jdbc:postgresql://postgres:5432/orders
      DB_USERNAME: orders_user
      DB_PASSWORD: orders_pass
      KAFKA_BOOTSTRAP_SERVERS: kafka:9092
      EUREKA_URL: http://eureka:8761/eureka/
    depends_on:
      postgres:
        condition: service_healthy
      kafka:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 5s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orders_user
      POSTGRES_PASSWORD: orders_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U orders_user -d orders"]
      interval: 10s
      timeout: 5s
      retries: 5

  kafka:
    image: confluentinc/cp-kafka:7.4.0
    environment:
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1

volumes:
  postgres_data:
```

---

## ☸️ 10.2 Kubernetes — Production Deployment

```yaml
# order-service deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
  namespace: myapp
  labels:
    app: order-service
    version: v1.5.2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: order-service
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # allow 1 extra pod during update
      maxUnavailable: 0    # never have fewer than 3 healthy pods (zero downtime)
  template:
    metadata:
      labels:
        app: order-service
        version: v1.5.2
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/actuator/prometheus"
    spec:
      serviceAccountName: order-service-sa

      # Anti-affinity: spread pods across nodes
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchExpressions:
                    - key: app
                      operator: In
                      values: [order-service]
                topologyKey: kubernetes.io/hostname

      containers:
        - name: order-service
          image: myregistry/order-service:v1.5.2
          imagePullPolicy: Always
          ports:
            - containerPort: 8080

          # Resource limits — ALWAYS set these!
          resources:
            requests:
              memory: "512Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"

          # Liveness probe — is the service alive? (restart if fails)
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 60  # give time to start
            periodSeconds: 10
            failureThreshold: 3

          # Readiness probe — is the service ready for traffic? (remove from LB if fails)
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 5
            failureThreshold: 3

          # Startup probe — for slow-starting apps (prevents premature liveness failures)
          startupProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            failureThreshold: 30
            periodSeconds: 10  # up to 5 minutes to start

          env:
            - name: SPRING_PROFILES_ACTIVE
              value: "prod"
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:           # from Kubernetes Secret
                  name: order-db-secret
                  key: password
            - name: JAVA_OPTS
              value: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

          envFrom:
            - configMapRef:
                name: order-service-config  # non-sensitive config

          # Graceful shutdown
          lifecycle:
            preStop:
              exec:
                command: ["/bin/sh", "-c", "sleep 10"]  # give time for LB to remove

      terminationGracePeriodSeconds: 60  # time for graceful shutdown

---
# Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: order-service-hpa
  namespace: myapp
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: order-service
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70  # scale when avg CPU > 70%
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80

---
# Pod Disruption Budget — maintain availability during voluntary disruptions
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: order-service-pdb
spec:
  selector:
    matchLabels:
      app: order-service
  minAvailable: 2  # always keep at least 2 pods running during node drain
```

---

## 🚀 10.3 Deployment Strategies

### Blue-Green Deployment
```
BLUE (current): order-service v1.5 (all traffic → blue)
GREEN (new):    order-service v1.6 (deployed but no traffic)

Test green thoroughly → switch load balancer to green (instant cutover)
If issues → switch back to blue (instant rollback)

Pros: ✅ Instant rollback, ✅ Zero downtime
Cons: ❌ Double infrastructure cost during deployment
```

### Canary Deployment
```
Phase 1: 95% traffic → v1.5, 5% traffic → v1.6
Phase 2: 80% → v1.5, 20% → v1.6
Phase 3: 0% → v1.5, 100% → v1.6 (after monitoring confirms v1.6 is healthy)

In Kubernetes with Istio:
  virtualservice splits traffic by percentage
  Argo Rollouts automates progressive delivery

Pros: ✅ Gradual exposure, ✅ Real-user validation
Cons: ❌ Complex setup, ❌ Two versions running simultaneously
```

### Rolling Update (Kubernetes default)
```
Pod 1: v1.5 → v1.6
Pod 2: v1.5 → v1.6  (after Pod 1 passes readiness)
Pod 3: v1.5 → v1.6  (after Pod 2 passes readiness)

Pros: ✅ No downtime, ✅ Built into Kubernetes, ✅ No extra infrastructure
Cons: ❌ Slow, ❌ Both versions serve traffic simultaneously (API must be backward compatible)
```

---

---

# 11. Observability & Distributed Tracing

## 🔭 11.1 The Three Pillars

```
LOGS: What happened?
  "2024-01-15 14:32:01 ERROR Order 123 payment failed: InsufficientFunds"
  → Use: debugging specific incidents

METRICS: How is the system performing?
  "Order creation rate: 1500/min, P99 latency: 450ms, Error rate: 0.1%"
  → Use: alerting, capacity planning, SLO monitoring

TRACES: Where did the time go?
  "Request to /api/orders took 620ms:
     - Order Service: 50ms
     - Inventory Service call: 200ms
     - Payment Service call: 350ms  ← bottleneck!"
  → Use: latency debugging, dependency mapping
```

---

## 📊 11.2 Distributed Tracing with Spring Boot

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 0.1   # sample 10% in production (100% is too expensive)

  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans

logging:
  pattern:
    console: "%d{HH:mm:ss} [%X{traceId}/%X{spanId}] [%X{userId}] %-5level %logger{30} - %msg%n"
```

```java
// Trace context automatically propagated via HTTP headers:
// traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01

// Manual instrumentation when needed
@Service
@RequiredArgsConstructor
public class OrderService {

    private final Tracer tracer;
    private final OrderRepository orderRepository;

    public OrderResponse processOrder(OrderRequest request) {
        // Create custom span for a specific operation
        Span span = tracer.nextSpan()
            .name("db.query.findExistingOrders")
            .tag("customerId", request.getCustomerId())
            .start();

        try (Tracer.SpanInScope scope = tracer.withSpan(span)) {
            List<Order> existing = orderRepository.findByCustomerIdAndStatus(
                request.getCustomerId(), OrderStatus.PENDING
            );
            span.tag("resultCount", String.valueOf(existing.size()));
            // ... rest of logic
        } finally {
            span.end();
        }
    }
}
```

---

## 📈 11.3 Metrics, Alerts, and SLOs

```java
// Define SLOs (Service Level Objectives)
// Availability SLO: 99.9% requests succeed
// Latency SLO: P95 < 500ms, P99 < 1s

// Prometheus metrics exposed via /actuator/prometheus
// Key metrics to monitor:

// 1. Request rate (RED Method: Rate, Errors, Duration)
Counter.builder("http.requests.total")
    .tag("method", request.getMethod())
    .tag("path", request.getUri())
    .tag("status", String.valueOf(response.getStatus()))
    .register(meterRegistry)
    .increment();

// 2. Latency histogram
Timer.builder("http.request.duration")
    .tag("endpoint", endpoint)
    .publishPercentiles(0.50, 0.90, 0.95, 0.99)
    .publishPercentileHistogram()
    .register(meterRegistry);

// 3. Business metrics
Gauge.builder("orders.active.count",
    () -> orderRepository.countByStatus(OrderStatus.PENDING))
    .register(meterRegistry);

Counter.builder("orders.completed.total")
    .tag("result", "success")
    .register(meterRegistry);
```

```yaml
# Grafana Alert Rule (PromQL)

# Alert: Error rate > 1% for 5 minutes
- alert: HighErrorRate
  expr: |
    sum(rate(http_requests_total{status=~"5.."}[5m]))
    /
    sum(rate(http_requests_total[5m])) > 0.01
  for: 5m
  labels:
    severity: critical
  annotations:
    summary: "High error rate: {{ $value | humanizePercentage }}"

# Alert: P99 latency > 1 second
- alert: HighLatency
  expr: histogram_quantile(0.99, rate(http_request_duration_seconds_bucket[5m])) > 1
  for: 2m
  labels:
    severity: warning
  annotations:
    summary: "P99 latency is {{ $value }}s"
```

---

## 📝 11.4 Structured Logging (Centralized)

```java
// Logback configuration for JSON structured logging
// logback-spring.xml
/*
<appender name="JSON_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
  <encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <customFields>{"service":"order-service","env":"prod","version":"1.5.2"}</customFields>
    <includeMdcKeyName>traceId</includeMdcKeyName>
    <includeMdcKeyName>spanId</includeMdcKeyName>
    <includeMdcKeyName>userId</includeMdcKeyName>
    <includeMdcKeyName>tenantId</includeMdcKeyName>
  </encoder>
</appender>
*/

// Structured log output:
// {
//   "timestamp": "2024-01-15T14:32:01.123Z",
//   "level": "ERROR",
//   "service": "order-service",
//   "traceId": "4bf92f3577b34da6",
//   "userId": "user-123",
//   "message": "Payment failed for order 456",
//   "orderId": "456",
//   "errorType": "InsufficientFunds"
// }

// MDC for request context
@Component
public class RequestContextFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws IOException, ServletException {
        try {
            MDC.put("requestId", getOrCreate(request, "X-Request-ID"));
            MDC.put("userId", request.getHeader("X-User-ID"));
            MDC.put("tenantId", request.getHeader("X-Tenant-ID"));
            chain.doFilter(request, response);
        } finally {
            MDC.clear(); // Always clear MDC after request
        }
    }
}
```

---

---

# 12. Testing Microservices

## 🧪 12.1 The Testing Pyramid for Microservices

```
         /\
        /  \  E2E Tests (few — slow, expensive, brittle)
       /----\
      /      \ Integration Tests (some — test service boundaries)
     /--------\
    /          \ Contract Tests (many — verify API contracts)
   /------------\
  /              \ Unit Tests (most — fast, cheap, isolated)
 /________________\
```

---

## 📜 12.2 Contract Testing with Pact

Contract tests verify that services honor the API contract they agreed upon.

```
CONSUMER-DRIVEN CONTRACT TESTING:

1. Consumer (Order Service) writes test defining what it expects from Provider (Inventory Service)
   → Generates a "Pact" file (contract)

2. Pact file published to Pact Broker (shared contract registry)

3. Provider (Inventory Service) verifies it can fulfill the contract
   → Tests run against real provider code (no mocking provider)

4. If provider changes break the contract → CI fails → developer notified BEFORE deploy
```

```java
// Consumer test (Order Service defines what it expects)
@ExtendWith(PactConsumerTestExt.class)
@PactTestFor(providerName = "inventory-service", port = "8083")
class InventoryClientContractTest {

    @Pact(consumer = "order-service")
    public RequestResponsePact getInventoryPact(PactDslWithProvider builder) {
        return builder
            .given("product product-123 has 10 items in stock")
            .uponReceiving("a request for inventory of product-123")
                .path("/api/v1/inventory/product-123")
                .method("GET")
                .headers(Map.of("Content-Type", "application/json"))
            .willRespondWith()
                .status(200)
                .body(LambdaDsl.newJsonBody(body -> {
                    body.stringType("productId", "product-123");
                    body.integerType("stockLevel", 10);
                    body.booleanType("available", true);
                }).build())
            .toPact();
    }

    @Test
    @PactTestFor(pactMethod = "getInventoryPact")
    void testGetInventory(MockServer mockServer) {
        InventoryServiceClient client = createClientFor(mockServer.getUrl());
        InventoryResponse response = client.getInventory("product-123");

        assertThat(response.getProductId()).isEqualTo("product-123");
        assertThat(response.isAvailable()).isTrue();
    }
}

// Provider verification (Inventory Service verifies it fulfills contracts)
@Provider("inventory-service")
@PactBroker(url = "http://pact-broker:9292")
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class InventoryServiceContractVerificationTest {

    @LocalServerPort
    private int port;

    @TestTarget
    public final Target target = new HttpTarget(port);

    @State("product product-123 has 10 items in stock")
    public void setupInventoryState() {
        // Set up test data for this state
        inventoryRepository.save(Inventory.builder()
            .productId("product-123")
            .stockLevel(10)
            .build());
    }
}
```

---

## 🔗 12.3 Integration Testing with Testcontainers

```java
// Full integration test with all dependencies in Docker
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Testcontainers
@ActiveProfiles("integration-test")
class OrderServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("orders_test");

    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.4.0")
    );

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);

    // Mock dependent services (Inventory, Payment) with WireMock
    @Container
    static WireMockContainer wireMock = new WireMockContainer("wiremock/wiremock:3.0.0")
        .withMappingFromResource("inventory-service-stubs.json");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", () -> redis.getMappedPort(6379));
        registry.add("services.inventory.url", wireMock::getBaseUrl);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private KafkaConsumer<String, String> testConsumer;

    @Test
    void placeOrder_ShouldCreateOrderAndPublishEvent() throws Exception {
        // Arrange — WireMock stub for inventory service
        stubFor(get(urlEqualTo("/api/v1/inventory/product-1"))
            .willReturn(okJson("""
                {"productId":"product-1","stockLevel":5,"available":true}
            """)));

        OrderRequest request = OrderRequest.builder()
            .customerId("customer-123")
            .items(List.of(new OrderItemRequest("product-1", 2)))
            .build();

        // Act
        ResponseEntity<OrderResponse> response = restTemplate.postForEntity(
            "/api/v1/orders", request, OrderResponse.class
        );

        // Assert HTTP
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(response.getBody().getOrderId()).isNotNull();

        // Assert Kafka event was published
        testConsumer.subscribe(List.of("order-created"));
        ConsumerRecords<String, String> records = testConsumer.poll(Duration.ofSeconds(10));
        assertThat(records.count()).isEqualTo(1);
        assertThat(records.iterator().next().value()).contains("order-created");

        // Assert WireMock was called
        verify(getRequestedFor(urlEqualTo("/api/v1/inventory/product-1")));
    }
}
```

---

---

# 📝 Quick Reference — Microservices Interview Traps & Trade-offs

## Key Trade-offs Summary

```
1. SYNCHRONOUS vs ASYNCHRONOUS COMMUNICATION
   Sync:  Simple ✅, Tight coupling ❌, Cascade failures ❌
   Async: Resilient ✅, Complex ❌, Eventual consistency ❌

2. CHOREOGRAPHY vs ORCHESTRATION SAGA
   Choreography: Decoupled ✅, Hard to trace ❌
   Orchestration: Visible ✅, Central coupling ❌

3. API COMPOSITION vs CQRS
   Composition: Consistent ✅, Latency/availability risk ❌
   CQRS: Fast reads ✅, Eventual consistency ❌, Complexity ❌

4. MICROSERVICES vs MONOLITH
   Microservices: Scale/deploy independently ✅, Operational complexity ❌
   Monolith: Simple ✅, Scale everything together ❌

5. FINE-GRAINED vs COARSE-GRAINED SERVICES
   Fine-grained: Flexible ✅, Chatty ❌, Distributed transactions ❌
   Coarse-grained: Fewer hops ✅, Harder to deploy independently ❌
```

## Most Common Interview Scenarios

**Q: How do you handle a distributed transaction across 3 services?**
Use Saga pattern. Prefer Orchestration for complex flows (visible, testable). Each service does its local transaction and publishes event. If any step fails, compensation transactions undo previous steps. Accept eventual consistency — this is a fundamental trade-off.

**Q: Service A calls Service B which is down. What happens?**
Without resilience: A hangs → thread pool exhausts → A becomes unresponsive.
With patterns: Circuit Breaker trips → A returns fallback immediately → users see degraded (but functional) experience → B recovers → Circuit Breaker closes → normal operation resumes.

**Q: How do you ensure events are not lost between DB write and Kafka publish?**
Outbox Pattern: write to DB and outbox table in one ACID transaction. Separate polling process reads outbox and publishes to Kafka. Guarantees at-least-once delivery. Consumers must be idempotent.

**Q: How do you prevent double-charging in a payment retry scenario?**
Idempotency Keys: client sends a unique key with payment request. Payment service stores processed keys (in Redis with TTL). If same key received again → return cached response, don't charge again.

**Q: How do you achieve zero-downtime deployment?**
Rolling updates in Kubernetes + readiness probes (remove pod from load balancer before updating) + graceful shutdown (wait for in-flight requests) + backward-compatible API changes (don't break consumers during transition).

**Q: How do you debug a slow request spanning 5 services?**
Distributed tracing (Zipkin/Jaeger): every request gets a TraceID propagated via HTTP headers. View waterfall chart showing time spent in each service and each database call. Identify the bottleneck service/query.

**Q: What is eventual consistency and when is it acceptable?**
Data changes are not immediately reflected everywhere — it takes time for all copies to converge. Acceptable when: reading data that's slightly stale is OK (product catalog, user profiles, recommendations). Not acceptable when: financial operations, inventory reservation (overselling risk), security permissions.

---

## 🗺️ Microservices Decision Framework

```
Should I use microservices?
  → Team size < 10? → Consider modular monolith first
  → Clear domain boundaries? → Yes = good candidate
  → Need independent scaling? → Strong reason for microservices
  → Different tech stacks per service? → Polyglot = microservices

How to decompose?
  → Start with business capabilities (most natural boundaries)
  → Apply DDD bounded contexts for complex domains
  → "Can this be deployed/changed without touching other services?" = boundary test

How should services communicate?
  → Need result immediately? → REST/gRPC
  → Fire and forget? → Kafka/RabbitMQ
  → Multiple consumers? → Kafka pub/sub
  → High throughput internal? → gRPC

How to handle failures?
  → Add Circuit Breaker (always)
  → Set timeouts (always, never wait forever)
  → Use Bulkheads for critical paths
  → Design fallbacks (what to return when dependency is down)

How to handle distributed data?
  → Sagas for distributed transactions
  → CQRS for read optimization
  → Outbox for reliable event publishing
  → Event Sourcing for full audit trail (when needed)
```

---

*Document Version: 1.0 | Created for Microservices Interview Preparation*
*Covers: Architecture, Communication, Resilience, Data Management, Events, Security, Deployment, Observability, Testing*
*Frameworks: Spring Boot, Spring Cloud, Kafka, Docker, Kubernetes*
Microservices_Interview_Prep.md
Displaying Microservices_Interview_Prep.md.