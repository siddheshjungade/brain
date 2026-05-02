---
title: Scaling to Million Users
tags:
  - system-desing
  - basic
---


# 🚀 Scaling from Zero to Millions of Users

> Based on *System Design Interview* by Alex Xu — Chapter 1

---

## 1. Single Server Setup

Everything runs on one server: web app, database, cache, etc.

```mermaid
graph LR
    Users -->|1: HTTP Request| DNS
    DNS -->|2: IP Address| Users
    Users -->|3: Request| WebServer[Web Server<br/>IP: x.x.x.x]
    WebServer -->|4: Query| DB[(Database)]
    subgraph Single Server
        WebServer
        DB
    end
```

**Flow:**
1. User enters domain → DNS returns IP address
2. Browser sends HTTP request to the web server
3. Web server returns HTML/JSON response

---

## 2. Database Separation

Separate the **web/mobile traffic tier** from the **data tier** so they can scale independently.

```mermaid
graph LR
    Users -->|Request| WebServer[Web Tier<br/>Server]
    WebServer -->|Read/Write| DB[(Database)]
```

### Which Database to Choose?

| Relational (SQL) | Non-Relational (NoSQL) |
|---|---|
| MySQL, PostgreSQL, Oracle | MongoDB, CouchDB, Cassandra, HBase |
| Structured data, ACID | Unstructured, flexible schema |
| Joins, relationships | Super-low latency, massive data |

**Choose NoSQL when:**
- Need super-low latency
- Data is unstructured or no relational data
- Only need to serialize/deserialize (JSON, XML, YAML)
- Need to store massive amounts of data

---

## 3. Vertical Scaling vs Horizontal Scaling

| Vertical Scaling (Scale Up) | Horizontal Scaling (Scale Out) |
|---|---|
| Add more CPU/RAM to existing server | Add more servers |
| Simple, but has hard limits | No hard limit, more complex |
| Single point of failure | Better fault tolerance |
| ❌ Not ideal for large-scale apps | ✅ Preferred for large-scale apps |

---

## 4. Load Balancer

Distributes incoming traffic across multiple web servers.

```mermaid
graph TD
    Users -->|Public IP| LB[Load Balancer]
    LB -->|Private IP| WS1[Web Server 1]
    LB -->|Private IP| WS2[Web Server 2]
    WS1 --> DB[(Database)]
    WS2 --> DB
```

**Benefits:**
- If Server 1 goes down → traffic routed to Server 2
- If traffic grows → just add more servers
- Servers use **private IPs** (not reachable from internet directly)
- Load balancer has the **public IP**

**Failover:** With this setup, the web tier has **no single point of failure**.

---

## 5. Database Replication (Master-Slave)

```mermaid
graph TD
    WebServers[Web Servers] -->|Writes| Master[(Master DB)]
    Master -->|Replicate| Slave1[(Slave DB 1)]
    Master -->|Replicate| Slave2[(Slave DB 2)]
    Master -->|Replicate| Slave3[(Slave DB 3)]
    WebServers -->|Reads| Slave1
    WebServers -->|Reads| Slave2
    WebServers -->|Reads| Slave3
```

| Master | Slave |
|---|---|
| Supports **write** operations (INSERT, UPDATE, DELETE) | Supports **read** operations only |
| Usually 1 master | Multiple slaves |

**Advantages:**
- **Better performance** — reads distributed across slaves (most apps have way more reads than writes)
- **Reliability** — data replicated across locations; no data loss if one DB destroyed
- **High availability** — site still operational if one DB goes offline

**Failover scenarios:**
- If **only one slave** & it goes offline → reads directed to master temporarily
- If **master** goes offline → a slave is promoted to master

---

## 6. Cache

A temporary storage layer that stores frequently accessed data in memory (much faster than DB).

```mermaid
graph LR
    WebServer -->|1: Check Cache| Cache[(Cache<br/>e.g. Memcached/Redis)]
    Cache -->|2a: Cache Hit| WebServer
    WebServer -->|2b: Cache Miss| DB[(Database)]
    DB -->|3: Return Data| WebServer
    WebServer -->|4: Store in Cache| Cache
```

### Read-Through Cache Strategy
1. Web server checks cache first
2. **Cache hit** → return data from cache
3. **Cache miss** → query DB, store result in cache, return to client

### Cache Considerations

| Consideration | Guideline |
|---|---|
| **When to use** | Frequent reads, infrequent writes |
| **Expiration** | Too short = too many DB hits; Too long = stale data |
| **Consistency** | Keep data store & cache in sync (hard at scale) |
| **Eviction policy** | LRU (Least Recently Used) is most popular; also LFU, FIFO |
| **Single point of failure** | Use multiple cache servers across data centers |
| **Memory** | Don't overload; evict old data when full |

---

## 7. Content Delivery Network (CDN)

A geographically distributed network of servers that cache **static content** (images, CSS, JS, videos).

```mermaid
graph LR
    User -->|1: Request image.png| CDN[CDN Edge Server<br/>closest to user]
    CDN -->|2a: Cache Hit| User
    CDN -->|2b: Cache Miss| Origin[Origin Server]
    Origin -->|3: Return + TTL header| CDN
    CDN -->|4: Cache and Return| User
```

**How it works:**
1. User requests a static asset
2. CDN edge server (nearest) checks if it has the asset
3. If not → fetches from origin, caches it with TTL
4. Subsequent requests served from CDN directly

### CDN Considerations
- **Cost** — CDNs are third-party; charged for data transfer. Don't cache infrequently used assets
- **Expiry (TTL)** — Too long = stale; Too short = too many origin hits
- **CDN fallback** — If CDN is down, client should be able to hit origin directly
- **Invalidation** — Use APIs provided by CDN vendors, or use object versioning (e.g., `image_v2.png`)

---

## 8. Stateless Web Tier

To scale horizontally, move **state** (e.g., session data) out of web servers.

### Stateful Architecture (❌ Problem)

```mermaid
graph TD
    UserA -->|Session stored here| Server1[Server 1]
    UserB -->|Session stored here| Server2[Server 2]
    UserC -->|Session stored here| Server3[Server 3]
```

- Each user's session is tied to a specific server
- Load balancer must use **sticky sessions** → harder to scale, harder to handle failures

### Stateless Architecture (✅ Solution)

```mermaid
graph TD
    Users --> LB[Load Balancer]
    LB --> Server1[Server 1]
    LB --> Server2[Server 2]
    LB --> Server3[Server 3]
    Server1 --> SharedStore[(Shared Data Store<br/>Redis / Memcached / NoSQL)]
    Server2 --> SharedStore
    Server3 --> SharedStore
```

- Session data stored in a **shared data store** (Redis, Memcached, or NoSQL DB)
- Any server can handle any request
- **Simpler, more robust, scalable**
- Auto-scaling becomes easy — just add/remove servers

---

## 9. Data Centers (Geo-Routing)

For high availability and better user experience across geographies.

```mermaid
graph TD
    Users -->|GeoDNS| GeoDNS{GeoDNS}
    GeoDNS -->|US users| DC1[Data Center 1 - US East]
    GeoDNS -->|EU users| DC2[Data Center 2 - EU West]
    DC1 --> DB1[(DB)]
    DC2 --> DB2[(DB)]
    DB1 <-->|Replication| DB2
```

**GeoDNS** routes users to the nearest data center based on IP location.

### Multi-Data Center Challenges:
- **Traffic redirection** — GeoDNS to direct to correct DC
- **Data synchronization** — replicate data across DCs
- **Test & deployment** — test at different locations; automated deployments across DCs

**Failover:** If DC1 goes offline → all traffic redirected to DC2.

---

## 10. Message Queue (Decoupling)

A durable component that supports **asynchronous communication**.

```mermaid
graph LR
    Producers[Producers<br/>Web Servers] -->|Publish| MQ[Message Queue<br/>RabbitMQ / Kafka / SQS]
    MQ -->|Subscribe/Consume| Workers[Workers/Consumers<br/>Photo Processing, etc.]
```

**How it works:**
- **Producers** publish messages to the queue
- **Consumers/Workers** pick up and process messages
- Producer and consumer can scale **independently**

**Example:** Photo customization (crop, sharpen, blur)
- Web server publishes photo processing tasks to queue
- Workers pick up tasks asynchronously
- If queue grows → add more workers

**Benefits:**
- Decouples components → each can fail independently
- Producer doesn't wait for consumer
- Can buffer spikes in traffic

---

## 11. Logging, Metrics & Automation

Essential at large scale:

| Tool | Purpose |
|---|---|
| **Logging** | Monitor errors, aggregate logs (ELK stack, Splunk) |
| **Metrics** | Host-level (CPU, Memory, Disk), Aggregated (DB tier perf), Business (DAU, retention) |
| **Automation** | CI/CD, automated testing, build automation |

---

## 12. Database Scaling

### Vertical Scaling (Scale Up)
- Add more CPU, RAM, disk to existing DB
- e.g., Amazon RDS: up to 24 TB RAM
- ❌ Single point of failure, hardware limits, expensive

### Horizontal Scaling (Sharding)

Split data across multiple databases (shards).

```mermaid
graph TD
    App[Application] -->|user_id % 4| Router{Shard Router}
    Router -->|0| Shard0[(Shard 0<br/>user_id 0,4,8...)]
    Router -->|1| Shard1[(Shard 1<br/>user_id 1,5,9...)]
    Router -->|2| Shard2[(Shard 2<br/>user_id 2,6,10...)]
    Router -->|3| Shard3[(Shard 3<br/>user_id 3,7,11...)]
```

**Sharding Key (Partition Key):** Determines how data is distributed. e.g., `user_id % num_shards`

### Sharding Challenges

| Challenge | Description |
|---|---|
| **Resharding** | Needed when a shard is full or uneven distribution. Use consistent hashing |
| **Celebrity problem (Hotspot)** | Excessive reads on one shard (e.g., celebrity user). May need further partition or dedicated shard |
| **Joins & Denormalization** | Cross-shard joins are hard. Solution: denormalize data |

---

## 13. Final Architecture — Millions of Users

```mermaid
graph TD
    Users -->|GeoDNS| CDN[CDN]
    Users -->|GeoDNS| LB[Load Balancer]
    
    LB --> WS1[Web Server 1]
    LB --> WS2[Web Server 2]
    LB --> WSn[Web Server N]
    
    WS1 --> Cache[(Cache Cluster<br/>Redis)]
    WS2 --> Cache
    WSn --> Cache
    
    WS1 --> MQ[Message Queue]
    MQ --> Workers[Workers]
    
    Cache --> DBShards[(DB Shards)]
    WS1 --> DBShards
    
    subgraph Data Center 1
        LB
        WS1
        WS2
        WSn
        Cache
        MQ
        Workers
        DBShards
    end
```

---

## 📋 Summary — Scale Step by Step

| Users | What to Add |
|---|---|
| **1** | Single server (web + DB on same machine) |
| **~100** | Separate web server & database |
| **~1,000** | Load balancer + multiple web servers |
| **~10,000** | Database replication (master/slave) |
| **~100,000** | Cache layer (Redis/Memcached) |
| **~500,000** | CDN for static assets |
| **~1,000,000** | Stateless web tier + session store |
| **~5,000,000** | Multiple data centers + GeoDNS |
| **~10,000,000+** | Message queues + sharding + logging/metrics/automation |

---

## 🔑 Key Takeaways

- Keep web tier **stateless**
- Build **redundancy** at every tier
- **Cache** as much data as possible
- Support **multiple data centers**
- Host **static assets** on CDN
- Scale data tier by **sharding**
- Split tiers into **individual services** (microservices)
- **Monitor** the system and use **automation** tools