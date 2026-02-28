# 🔧 Architecture Patterns for IoT

> Selecting the right architecture pattern is crucial for building IoT systems that scale, perform, and evolve with changing requirements.

---

## Overview

No single architecture pattern fits all IoT scenarios. The best approach depends on factors like latency requirements, scale, team expertise, and operational constraints. This document explores the most relevant patterns for IoT systems.

---

## 1. Layered Architecture

### Description
Organizes the system into horizontal layers, each with a specific responsibility. Communication flows vertically between adjacent layers.

### Structure
```
┌─────────────────────┐
│   Presentation      │  UI, dashboards, mobile apps
├─────────────────────┤
│   Business Logic    │  Rules, workflows, domain logic
├─────────────────────┤
│   Data Access       │  Repositories, queries, caching
├─────────────────────┤
│   Infrastructure    │  Databases, message brokers, cloud services
└─────────────────────┘
```

### Pros & Cons

| Pros | Cons |
|---|---|
| Simple to understand and implement | Can become a "big ball of mud" at scale |
| Clear separation of concerns | Tight coupling between layers |
| Good for small-to-medium IoT projects | Difficult to scale independently |
| Easy to test individual layers | Changes often cascade across layers |

### Best For
- Small IoT projects with < 100 devices
- Proof of concept and prototyping
- Teams new to IoT development

---

## 2. Microservices Architecture

### Description
Decomposes the system into small, independently deployable services, each owning a specific domain capability.

### IoT Microservices Example
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Device     │  │  Telemetry   │  │   Alert      │
│   Registry   │  │  Ingestion   │  │   Service    │
│              │  │              │  │              │
│ CRUD devices │  │ MQTT → DB    │  │ Rules → Push │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └────────────┬────┴─────────────────┘
                    │
            ┌───────┴───────┐
            │ Message Broker │  (Kafka / RabbitMQ)
            └───────┬───────┘
                    │
┌──────────────┐  ┌─┴────────────┐  ┌──────────────┐
│   OTA Update │  │  Analytics   │  │  User        │
│   Service    │  │  Service     │  │  Service     │
│              │  │              │  │              │
│ Firmware mgmt│  │ ML pipeline  │  │ Auth, prefs  │
└──────────────┘  └──────────────┘  └──────────────┘
```

### Key Design Decisions

1. **Service boundaries**: Align with IoT domain capabilities (device management, telemetry, analytics, OTA)
2. **Communication**: Asynchronous messaging (Kafka, MQTT) for telemetry; synchronous (REST, gRPC) for queries
3. **Data ownership**: Each service owns its database — no shared databases
4. **Service discovery**: Use Kubernetes service mesh, Consul, or Eureka
5. **Containerization**: Docker + Kubernetes for deployment and orchestration

### Pros & Cons

| Pros | Cons |
|---|---|
| Independent scaling per service | Operational complexity (many moving parts) |
| Technology diversity per service | Distributed system challenges (network, consistency) |
| Fault isolation | Requires DevOps maturity (CI/CD, monitoring) |
| Team autonomy and parallel development | Inter-service communication overhead |

### Best For
- Large-scale IoT platforms (10K+ devices)
- Multi-team organizations
- Systems requiring independent scaling of components

---

## 3. Event-Driven Architecture (EDA)

### Description
Components communicate by producing and consuming events. An event represents a significant state change in the system.

### Structure
```
  Sensors/Devices          Event Broker              Event Consumers
  ┌────────────┐      ┌───────────────────┐      ┌─────────────────┐
  │ Temperature │─────▶│                   │─────▶│ Real-time Alert │
  │ Sensor      │      │                   │      │ Service         │
  └────────────┘      │   Apache Kafka    │      └─────────────────┘
                      │       or          │
  ┌────────────┐      │   MQTT Broker     │      ┌─────────────────┐
  │ Motion     │─────▶│       or          │─────▶│ Time-Series     │
  │ Detector   │      │   RabbitMQ        │      │ Storage         │
  └────────────┘      │                   │      └─────────────────┘
                      │                   │
  ┌────────────┐      │                   │      ┌─────────────────┐
  │ Door       │─────▶│                   │─────▶│ Analytics       │
  │ Sensor     │      │                   │      │ Pipeline        │
  └────────────┘      └───────────────────┘      └─────────────────┘
```

### Event-Driven Patterns for IoT

#### Event Notification
- Lightweight events announcing that something happened
- Consumer decides whether to react
- Example: `{event: "temperature_exceeded", deviceId: "sensor-42", threshold: 90}`

#### Event-Carried State Transfer
- Events carry full state, reducing need for callbacks
- Example: `{deviceId: "sensor-42", temperature: 95.2, humidity: 45, battery: 78, timestamp: "..."}`

#### Event Sourcing
- Store all events as an immutable log
- Reconstruct current state by replaying events
- Perfect for: audit trails, debugging, regulatory compliance
- Example: Full history of a valve's open/close events over 2 years

#### CQRS (Command Query Responsibility Segregation)
- Separate write path (commands from devices) from read path (queries from dashboards)
- Write path optimized for high-throughput ingestion
- Read path optimized for complex queries and aggregations

### Pros & Cons

| Pros | Cons |
|---|---|
| Natural fit for IoT (sensors produce events) | Eventual consistency (not immediate) |
| Loose coupling between producers and consumers | Debugging is harder (asynchronous flow) |
| Easy to add new consumers without changing producers | Message ordering can be complex |
| Excellent scalability for high-throughput data | Requires robust broker infrastructure |

### Best For
- Real-time monitoring and alerting systems
- Systems with high event throughput (millions/second)
- Systems needing audit trails and replay

---

## 4. Serverless Architecture

### Description
Application logic runs in stateless, event-triggered functions managed by a cloud provider. No server management required.

### IoT Serverless Example
```
  IoT Device                Cloud Provider                  Outputs
  ┌─────────┐   MQTT   ┌────────────────────┐        ┌──────────────┐
  │ Sensor  │────────▶ │ IoT Hub / IoT Core │        │ DynamoDB     │
  └─────────┘          └────────┬───────────┘        │ (telemetry)  │
                                │                     └──────────────┘
                                │ trigger                    ▲
                                ▼                            │
                       ┌────────────────┐                    │
                       │ Lambda/Function │ ──── transform ───┘
                       │ (process event) │
                       └────────┬───────┘
                                │ if anomaly detected
                                ▼
                       ┌────────────────┐        ┌──────────────┐
                       │ Lambda/Function │───────▶│ SNS / Email  │
                       │ (send alert)    │        │ (notification)│
                       └────────────────┘        └──────────────┘
```

### Pros & Cons

| Pros | Cons |
|---|---|
| Zero server management | Cold start latency (100ms–2s) |
| Automatic scaling to zero and to millions | Execution time limits (15 min for Lambda) |
| Pay only for actual invocations | Vendor lock-in |
| Fast development cycle | Limited local state |
| Built-in HA and fault tolerance | Debugging and testing challenges |

### Best For
- Notification and alerting pipelines
- Data transformation and enrichment
- Sporadic or unpredictable workloads
- Small teams with limited DevOps capacity

---

## 5. Edge-Cloud Hybrid Architecture

### Description
Distributes processing between edge nodes (close to devices) and cloud services. This is the **dominant pattern for enterprise IoT** in 2025+.

### Structure
```
┌─────────────────────────────────────────────────────────────┐
│                        CLOUD TIER                           │
│                                                             │
│  ┌───────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ ML Model  │  │ Long-term│  │ Fleet    │  │ Dashboard │  │
│  │ Training  │  │ Storage  │  │ Mgmt     │  │ & APIs    │  │
│  └───────────┘  └──────────┘  └──────────┘  └───────────┘  │
│                                                             │
└──────────────────────────┬──────────────────────────────────┘
                           │  Sync: aggregated data,
                           │  model updates, commands
                           │
┌──────────────────────────┼──────────────────────────────────┐
│                    EDGE TIER                                 │
│                          │                                   │
│  ┌───────────┐  ┌───────┴──┐  ┌──────────┐  ┌───────────┐  │
│  │ Real-time │  │ Local    │  │ AI       │  │ Gateway   │  │
│  │ Rules     │  │ Storage  │  │ Inference│  │ & Protocol│  │
│  │ Engine    │  │ (buffer) │  │ Model    │  │ Bridge    │  │
│  └───────────┘  └──────────┘  └──────────┘  └───────────┘  │
│                                                              │
└──────────────────────────┬──────────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
         ┌────┴────┐  ┌───┴────┐  ┌───┴────┐
         │ Sensor  │  │ Camera │  │ Actuator│
         └─────────┘  └────────┘  └────────┘
              DEVICE TIER
```

### Synchronization Patterns

| Pattern | Description | Use Case |
|---|---|---|
| **Store-and-Forward** | Buffer data locally, send when connected | Unreliable connectivity |
| **Eventual Consistency** | Edge and cloud converge over time | Non-critical telemetry |
| **Edge-First** | Edge makes decisions, cloud learns from them | Real-time safety systems |
| **Cloud-First** | Cloud makes decisions, edge executes | Fleet-wide coordination |
| **Bi-directional Sync** | Both edge and cloud can modify state | Device twin / shadow |

### Pros & Cons

| Pros | Cons |
|---|---|
| Ultra-low latency for critical decisions | Most complex architecture to implement |
| Works offline (resilience) | Synchronization challenges |
| Bandwidth cost reduction | Requires edge hardware investment |
| Data sovereignty compliance | Harder to debug (distributed systems) |
| Scalable (process data locally) | Security surface area at the edge |

### Best For
- Industrial IoT (IIoT) with safety requirements
- Systems requiring offline operation
- Video/image processing workloads
- Latency-sensitive applications (< 10ms response)

---

## Pattern Selection Framework

### Decision Matrix

Use this matrix to guide your pattern selection:

```
                        Low Scale              High Scale
                    ┌───────────────────┬────────────────────┐
  Low Latency      │   Layered +       │   Edge-Cloud       │
  Required         │   Edge Processing │   Hybrid           │
                   │                   │   + Microservices   │
                   ├───────────────────┼────────────────────┤
  Latency          │   Layered         │   Microservices    │
  Tolerant         │   or Serverless   │   + EDA            │
                    └───────────────────┴────────────────────┘
```

### Questions to Ask

1. **How many devices?** < 100 → Layered; 100–10K → Microservices; > 10K → Microservices + EDA
2. **Latency requirements?** < 100ms → Edge required; > 1s → Cloud-only viable
3. **Connectivity reliability?** Unreliable → Edge-Cloud Hybrid with store-and-forward
4. **Team size and expertise?** Small → Serverless or Layered; Large → Microservices
5. **Budget?** Limited → Serverless; Enterprise → Hybrid with dedicated edge hardware
6. **Regulatory requirements?** Data residency → Edge processing required

---

## Combining Patterns

In practice, IoT systems often **combine multiple patterns**:

| Combination | Example |
|---|---|
| Microservices + EDA | Cloud backend with Kafka for inter-service events |
| Serverless + EDA | Lambda functions triggered by IoT events |
| Edge-Cloud + Microservices | Edge gateways + cloud microservices |
| Edge-Cloud + EDA + Microservices | Full enterprise IoT platform |

### Example: Smart Factory (Combined)
```
Devices → [Edge: Rules + AI] → [MQTT Broker]
              ↕                       ↓
         Local actuators    ┌─────────┴─────────┐
                            │                   │
                    ┌───────┴───────┐   ┌───────┴───────┐
                    │ Ingestion     │   │ Alert         │
                    │ Service       │   │ Service       │
                    │ (Microservice)│   │ (Serverless)  │
                    └───────┬───────┘   └───────────────┘
                            │
                    ┌───────┴───────┐
                    │ Analytics     │
                    │ Service       │
                    │ (Microservice)│
                    └───────────────┘
```

---

## Next Steps

- [Communication Protocols](03-protocols.md) — Choose the right protocol for each pattern
- [Security](04-security.md) — Secure your architecture at every layer
