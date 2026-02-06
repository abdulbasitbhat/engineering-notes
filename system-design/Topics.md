# 📘 System Design — Topics To Cover

# 🧱 00 — Fundamentals (Core Theory)

- **Scalability** — Ability of a system to handle increasing users/data without performance loss.  
- **Availability** — Ensuring system remains operational and accessible most of the time.  
- **Latency vs Throughput** — Response time of a request vs number of requests handled per second.  
- **Consistency Models** — Rules defining when data updates become visible across systems.  
- **CAP Theorem** — Tradeoff between Consistency, Availability, and Partition tolerance.  
- **PACELC Theorem** — Extends CAP showing latency vs consistency tradeoffs even without failures.  
- **Load Balancing** — Distributing incoming traffic across multiple servers.  
- **Caching Basics** — Storing frequently accessed data for faster retrieval.  
- **Cache Invalidation** — Techniques to update/remove stale cache data correctly.  
- **Database Indexing** — Optimizing database queries using indexed data structures.  
- **Replication** — Copying data across multiple DB nodes for reliability and scaling reads.  
- **Sharding / Partitioning** — Splitting data across multiple databases for scale.  
- **Messaging Queues Basics** — Asynchronous communication using queues between services.  
- **Horizontal vs Vertical Scaling** — Adding more machines vs upgrading existing machines.  
- **Stateless vs Stateful Services** — Whether server stores user/session state or not.  

---

# 🏗️ 01 — Core System Components

- **API Gateway** — Central entry point managing routing, auth, and rate limits.  
- **Reverse Proxy** — Intermediary server routing client requests to backend services.  
- **CDN** — Global cache servers delivering content faster to users.  
- **Rate Limiting** — Restricting number of requests to prevent abuse or overload.  
- **Authentication vs Authorization** — Identity verification vs permission checking.  
- **Session Management** — Tracking logged-in user state across requests.  
- **Service Discovery** — Mechanism for services to find each other dynamically.  
- **Configuration Management** — Centralized handling of environment configs and secrets.  
- **Feature Flags** — Enabling/disabling features without redeploying code.  
- **Monolith Architecture** — Single unified codebase handling all features.  
- **Microservices Architecture** — System split into independently deployable services.  
- **Event Driven Architecture** — Services communicate through events asynchronously.  
- **Serverless Architecture** — Running code without managing servers.  
- **Webhooks** — Event-based HTTP callbacks between systems.  
- **Background Workers** — Async workers processing tasks outside request cycle.  

---

# 🗄️ 02 — Databases & Storage

## Database Basics
- **SQL vs NoSQL** — Structured relational DBs vs flexible distributed DBs.  
- **When to choose SQL** — Use when consistency and relations are critical.  
- **When to choose NoSQL** — Use when scalability and flexible schema needed.  
- **ACID vs BASE** — Strong consistency guarantees vs eventual consistency tradeoffs.  
- **Transactions** — Ensuring multiple DB operations succeed or fail together.  
- **Isolation Levels** — Control visibility of concurrent transactions.  

## Scaling Databases
- **Read Replicas** — Copies of DB used to scale read traffic.  
- **Write Scaling** — Techniques to handle heavy write workloads.  
- **Sharding Strategies** — Splitting data across DBs using shard keys.  
- **Partitioning Strategies** — Dividing tables for performance and management.  
- **Database Federation** — Splitting DB by function/service boundaries.  
- **Multi-region Databases** — DB deployments across geographic regions.  

## Data Modeling
- **Normalization vs Denormalization** — Structured relations vs optimized reads.  
- **Indexing Strategies** — Choosing indexes for performance-critical queries.  
- **Composite Indexes** — Multi-column indexes for complex queries.  
- **Query Optimization** — Improving query speed via schema/index tuning.  
- **Schema Design for Scale** — Designing data structures for growth.  

---

# 🔌 03 — Communication & Messaging

- **REST vs gRPC** — HTTP JSON APIs vs high-performance binary RPC.  
- **GraphQL** — Flexible query language for fetching specific data.  
- **WebSockets** — Persistent real-time bidirectional communication.  
- **Polling vs Long Polling** — Repeated requests vs held request for updates.  
- **Server Sent Events** — One-way real-time server updates to client.  
- **Message Queues** — Buffer for async task processing.  
- **Pub-Sub Architecture** — Producers publish events, consumers subscribe.  
- **Event Streaming** — Continuous stream processing of events.  
- **Kafka** — Distributed event streaming platform.  
- **RabbitMQ** — Reliable message broker for async communication.  
- **Exactly once vs At least once** — Message delivery guarantees.  
- **Idempotency** — Ensuring repeated requests produce same result.  

---

# 🚀 04 — Scalability & Reliability Patterns

## Caching
- **Client Caching** — Browser/device-side caching.  
- **Server Caching** — Backend caching layer.  
- **CDN Caching** — Edge caching near users.  
- **Redis Caching** — In-memory distributed caching.  
- **Cache Invalidation Strategies** — Keeping cache consistent with DB.  
- **Write-through vs Write-back** — Cache update timing strategies.  
- **Cache-aside** — App manages cache reads/writes manually.  

## Reliability Patterns
- **Circuit Breaker** — Stops calls to failing service temporarily.  
- **Retry Pattern** — Reattempt failed operations safely.  
- **Exponential Backoff** — Increasing delay between retries.  
- **Dead Letter Queue** — Stores failed messages for inspection.  
- **Bulkhead Pattern** — Isolating failures to prevent system-wide crash.  
- **Graceful Degradation** — Reducing features instead of full failure.  
- **Failover Strategies** — Switching to backup systems on failure.  
- **Health Checks** — Monitoring service health automatically.  

## Performance
- **Load Shedding** — Dropping excess traffic to protect system.  
- **Backpressure** — Slowing producers when consumers overloaded.  
- **Connection Pooling** — Reusing DB/network connections.  
- **Async Processing** — Handling tasks asynchronously.  
- **Batching** — Processing multiple items together.  
- **Rate Limiting Design** — Controlling traffic per user/IP/service.  

---

# 🧠 05 — Distributed Systems Concepts

- **Distributed Locks** — Coordinating access across services.  
- **Leader Election** — Selecting a master node among many.  
- **Consensus (Raft/Paxos)** — Agreement among distributed nodes.  
- **Clock Synchronization** — Keeping time consistent across systems.  
- **Eventual Consistency** — Data becomes consistent over time.  
- **Split Brain Problem** — Cluster nodes diverging due to network partition.  
- **Quorum** — Majority agreement for distributed decisions.  
- **Gossip Protocols** — Nodes sharing state via peer communication.  
- **UUID** — Globally unique identifiers without coordination.  
- **Snowflake IDs** — Time-based distributed unique IDs.  
- **Auto Increment Issues** — Limits of sequential IDs in distributed DBs.  

---

# 🏢 06 — Real World System Design Case Studies

## Beginner
- **URL Shortener** — Map long URLs to short scalable links.  
- **Rate Limiter** — Control API usage per client.  
- **File Upload System** — Reliable storage for user uploads.  
- **Logging System** — Collect and analyze system logs.  
- **Notification System** — Send emails/SMS/push reliably.  
- **API Rate Limiter Design** — Scalable distributed throttling.  

## Intermediate
- **WhatsApp** — Real-time messaging at global scale.  
- **Twitter/X** — Feed generation and high write throughput.  
- **Instagram Feed** — Media-heavy personalized feed.  
- **YouTube** — Video upload, processing, and streaming.  
- **Dropbox** — Distributed file storage and sync.  
- **Google Drive** — Cloud storage and collaboration.  
- **Email System** — Reliable asynchronous messaging system.  
- **Search Autocomplete** — Real-time suggestion engine.  

## Advanced
- **Uber** — Real-time location and matching system.  
- **Netflix** — Global video streaming platform.  
- **Payment System** — Secure and consistent transaction processing.  
- **Trading System** — Low-latency financial order matching.  
- **Distributed Cron/Job Scheduler** — Reliable scheduled task execution.  
- **Real-time Chat System** — WebSocket-based messaging at scale.  
- **Google Docs** — Real-time collaborative editing.  
- **Multi-tenant SaaS** — Supporting many customers in one system.  

---

# 🛠️ 07 — Real World Engineering Learnings

- **Connector Architecture** — Designing pluggable integration systems.  
- **Job/Task Processing** — Async background processing systems.  
- **Retry & Failure Handling** — Making systems resilient to failures.  
- **Third-party API Integration** — Handling external dependencies.  
- **Webhook Processing** — Reliable event ingestion systems.  
- **Queue-based Processing** — Decoupled async processing pipelines.  
- **Monitoring & Alerting** — Observability of production systems.  
- **Logging Strategy** — Structured logging for debugging.  
- **Production Incidents** — Lessons from real outages.  
- **Scaling Learnings** — Real-world performance bottlenecks.  

---

# 📊 08 — Estimation Practice

- **Requests per Second** — Calculate system traffic load.  
- **Storage Estimation** — Predict storage growth.  
- **Bandwidth Estimation** — Network usage calculations.  
- **Memory Estimation** — RAM requirements planning.  
- **DAU Estimation** — Daily active users impact on scale.  
- **Peak Traffic Estimation** — Handling traffic spikes.  

---
