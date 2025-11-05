# Scalable Inventory System Architecture
### Principal Engineer Case Study – Explanation

## Introduction
This system is engineered to support high-volume, real-time inventory operations across distributed locations. The design ensures inventory accuracy, low latency, and operational resilience under unpredictable load.

To accomplish this, the platform uses a serverless, event-driven architecture paired with NoSQL for real-time stock control and RDS for analytical and reporting workloads, ensuring both high-speed transactional updates and robust query capabilities for business insights.

---

## Architecture Overview
This architecture blends:

- Serverless compute
- DynamoDB with atomic conditional writes for real-time inventory control
- Redis for low-latency hot-SKU reads
- Event streaming for system-wide propagation and analytics feeds
- SQS for asynchronous work buffering
- **RDS for analytical/reporting queries separated from transactional paths**

This combination delivers consistent inventory correctness at scale without sacrificing reporting capabilities.

---

## Component Breakdown & Justification

### Client Applications (React, React Native)

**Why**

- Cross-platform UI
- Optimistic UX pattern to reduce perceived latency

**Role**

- Real-time WebSocket subscription for stock changes

---

### CloudFront (Edge Caching)

**Why**

- Global acceleration for product browsing

**Role**

- Reduces load on API tier
- Ensures rapid asset delivery at scale

---

### API Gateway (REST + WebSockets)

**Why**

- Managed ingress + throttling

**Role**

- REST for inventory actions
- WebSocket for instant stock updates and cart events

---

### AWS Lambda

**Why**

- Dynamic scaling to match traffic bursts
- No idle cost

**Role**

- Executes business logic, idempotency, request validation
- Protects DB from malformed traffic

---

### DynamoDB (Primary Real-Time Store)

**Why**

- Millisecond writes under massive concurrency
- Atomic conditional updates prevent oversells

**Role**

- Source of truth for stock counts
- Version-based concurrency enforcement

---

### Redis / ElastiCache

**Why**

- Sub-millisecond reads for hot products

**Role**

- Caches popular SKUs
- Optional short-lived locks or rate limiting during flash events

---

### SQS + Lambda Workers

**Why**

- Decouples slow tasks from API path

**Role**

- Inventory reservation expiry
- Email/notification delivery
- Integration tasks

---

### DynamoDB Streams → Kinesis / EventBridge

**Why**

- Change Data Capture for system-wide sync

**Role**

- Streams real-time updates to clients, search index, analytics, and **RDS ETL pipeline**

---

### RDS (PostgreSQL / MySQL) for Reporting

**Why**

- Operational reporting and analytical queries often require joins, filtering, aggregates
- Avoids burdening DynamoDB with analytical workloads

**Role**

- Houses historical inventory views, audit trails, and business reports
- Populated via streaming ETL from DynamoDB events
- Enables BI dashboards and SQL-driven insights without impacting real-time operations

> The separation of OLTP (DynamoDB) and OLAP/reporting workloads (RDS) ensures that real-time inventory performance remains unaffected by heavy reporting queries.

---

### Observability: CloudWatch, X-Ray, Structured Logging

**Why**

- Distributed tracing and SLO-driven alerting

**Role**

- End-to-end visibility
- Fast failure identification and response

---

### Security Layer: WAF, IAM, Secrets Manager

**Why**

- Inventory systems are high-value targets for bot manipulation

**Role**

- API protection, least-privilege access, secret rotation

---

## Real-Time Inventory Consistency Strategy

DynamoDB handles real-time inventory updates exceptionally well, but analytical queries and reporting require relational capabilities and efficient aggregation. By streaming changes into RDS, we isolate analytical workloads from the transactional path, ensuring that reporting does not impact real-time performance and scale. This also supports SQL-based BI dashboards, audit views, and historical metrics, which are essential for operational visibility and business planning.

