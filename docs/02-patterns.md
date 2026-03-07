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

### Trade-off Analysis

| Dimension | Pros | Cons | IoT-Specific Context |
|---|---|---|---|
| **Complexity** | Simple to understand and implement | Can become a "big ball of mud" at scale | ✅ Ideal for POC; ❌ breaks down when device count crosses ~1K |
| **Coupling** | Clear separation of concerns within layers | Tight coupling *between* layers — cascading changes | ❌ Adding a new sensor type requires touching Perception, Network, and Application layers simultaneously |
| **Scalability** | Easy to reason about | Cannot scale layers independently | ❌ If telemetry ingestion is the bottleneck, you must scale the *entire* application, not just the ingestion component |
| **Testing** | Easy to unit-test individual layers | Integration tests must span all layers | ✅ Good for early validation; becomes slow as system grows |
| **Team fit** | Low barrier to entry | Not suited for multi-team development | ✅ Right for 1–3 person teams; ❌ creates merge conflicts in larger teams |

**When Layered Architecture breaks down in IoT:**
- Device count grows past ~1,000: the monolith cannot handle the telemetry volume without scaling everything
- Real-time requirements tighten below 100ms: a single application process cannot prioritize critical events
- Multiple teams need to work in parallel: shared codebase creates delivery bottlenecks

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

### Trade-off Analysis

| Dimension | Pros | Cons | IoT-Specific Context |
|---|---|---|---|
| **Scalability** | Scale each service independently | More infrastructure to manage | ✅ Telemetry Ingestion can scale to 10M msg/sec while OTA Service stays at minimal replicas |
| **Fault isolation** | One service failure doesn't cascade | Requires circuit breakers and timeouts between services | ✅ If Analytics Service crashes, device connectivity and alerting continue unaffected |
| **Technology diversity** | Each service can use the best tool for its job | Operational complexity of maintaining multiple stacks | ✅ Time-series DB for telemetry, relational DB for device registry, graph DB for device hierarchy |
| **Team velocity** | Multiple teams deliver independently | Requires API contract discipline between teams | ✅ Firmware team, ingestion team, and analytics team work without blocking each other |
| **Operational cost** | Right-sized resources per service | Kubernetes + service mesh + observability stack adds cost | ❌ Small teams spend 40-60% of effort on operations instead of features |
| **Latency** | Services are purpose-optimized | Network hops between services add latency | ❌ A command path across 3 services adds 10-30ms per hop — design the critical path carefully |

**When Microservices break down in IoT:**
- Small teams (< 5 engineers): the operational overhead exceeds the benefits
- Early-stage product: service boundaries drawn incorrectly require expensive refactoring
- Latency-critical paths: if < 10ms response is required, inter-service calls must be minimized or eliminated via event-driven communication

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

### Trade-off Analysis

| Dimension | Pros | Cons | IoT-Specific Context |
|---|---|---|---|
| **Coupling** | Producers and consumers are fully decoupled | Implicit contracts via event schema — schema changes break consumers silently | ✅ Add a new analytics consumer without touching the sensor firmware or ingestion code |
| **Scalability** | Brokers like Kafka handle millions of events/sec | Broker is a single point of failure if not clustered | ✅ Kafka handles 1M+ msg/sec; IoT telemetry peaks are absorbed naturally |
| **Extensibility** | Add consumers without modifying producers | Multiple consumers can process the same event — difficult to control side effects | ✅ Add an ML anomaly detector as a new consumer without any producer change |
| **Consistency** | Events are the source of truth; replay enables recovery | Eventual consistency — consumers may lag behind producers | ❌ A dashboard showing "current" state may be 50-500ms behind — acceptable for monitoring, not for control |
| **Debugging** | Event log enables full replay and forensics | Tracing an issue across async event chains is complex | ❌ A missed alert may be caused by a producer, broker, or consumer issue — requires distributed tracing |
| **Ordering** | Kafka preserves order within a partition | Guaranteeing global event ordering across partitions is hard | ❌ Two sensors in the same machine publishing to different partitions may arrive out of order at the consumer |

**When EDA breaks down in IoT:**
- Command-and-control paths: "turn off the pump NOW" cannot tolerate the latency/uncertainty of async messaging — use synchronous REST or gRPC for commands
- Small projects: a message broker adds infrastructure overhead that outweighs the benefits for < 10K events/day
- Strict read-your-own-writes consistency: if a device writes a configuration and immediately reads it back, eventual consistency can return stale data

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

### Trade-off Analysis

| Dimension | Pros | Cons | IoT-Specific Context |
|---|---|---|---|
| **Operational overhead** | Zero server management — no patching, capacity planning, or cluster ops | Vendor lock-in; migrating off requires rewriting function code and IAM | ✅ A 2-person startup can run alerting for 10K devices without a DevOps engineer |
| **Cost** | Pay-per-invocation — zero cost at zero traffic | At continuous high volume, per-invocation cost exceeds reserved capacity | ✅ Sporadic alert processing: cost is near-zero when no alerts fire. ❌ Continuous telemetry ingestion: 1M events/day at Lambda pricing can cost 5–10× more than a dedicated container |
| **Latency** | No idle server waste | Cold start: 100ms–2s when function hasn't been invoked recently | ❌ Unacceptable for < 100ms latency requirements. ✅ Acceptable for alert notifications where 1–2s delay is tolerable |
| **Scalability** | Scales to zero and to millions automatically | Concurrency limits (1,000 concurrent Lambda by default in AWS) may throttle under sudden spikes | ✅ Normal IoT workloads. ❌ Massive telemetry flood (DDoS scenario or large fleet reconnection) |
| **State** | Forces stateless design — easier to reason about | Stateless means each invocation re-fetches context from DB — adds latency | ❌ Session state (e.g., tracking whether a device alert was already sent) requires an external state store (DynamoDB, ElastiCache) |
| **Execution limits** | AWS Lambda: 15 min. Azure Functions: 10 min | Long-running processing (ML batch, video transcoding) cannot use serverless | ❌ ML inference on large sensor datasets; ✅ lightweight data enrichment and routing |

**When Serverless breaks down in IoT:**
- High-frequency continuous telemetry: Lambda cost is 3–10× higher than ECS/Kubernetes at sustained load
- Latency-sensitive paths: cold starts make serverless unsuitable for any path requiring < 200ms
- Complex stateful workflows: orchestrating multi-step device provisioning flows requires Step Functions or Durable Functions (workflow complexity, not simplicity)

### Best For
- Notification and alerting pipelines
- Data transformation and enrichment
- Sporadic or unpredictable workloads
- Small teams with limited DevOps capacity

> **Deep dive:** For cost analysis, cold start mitigation, cloud provider comparison (AWS/Azure/GCP), four architecture patterns with ASCII diagrams, security, observability, and a decision framework with real cost numbers, see [07-serverless-iot.md](07-serverless-iot.md).

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

### Trade-off Analysis

| Dimension | Pros | Cons | IoT-Specific Context |
|---|---|---|---|
| **Latency** | Edge processes locally — < 1ms for on-device, < 10ms for edge gateway | Adds an architectural tier that must be managed | ✅ Safety shutoff in a factory cannot wait for a cloud round-trip (50–200ms); edge reacts in < 10ms |
| **Reliability** | Continues operating when cloud is unreachable | Synchronization after reconnection is complex (conflict resolution, ordering, deduplication) | ✅ A smart grid node keeps managing load during WAN outage. ❌ Reconciling 4 hours of edge events with cloud state requires careful design |
| **Bandwidth cost** | Aggregate and filter at the edge — send 1–5% of raw data to cloud | Edge hardware (NVIDIA Jetson, industrial PC) costs $500–$5,000+ per site | ✅ Factory with 500 cameras: processing video at edge saves 95% of cloud bandwidth costs |
| **Data sovereignty** | Sensitive data never leaves the facility | Data residency is achieved by configuration, not architecture — requires operational discipline | ✅ GDPR, HIPAA scenarios where data must stay on-premises or in-country |
| **Security** | Network segmentation limits blast radius of a breach | Physical security of edge hardware is the operator's responsibility — devices can be stolen | ❌ An attacker with physical access to an edge gateway can extract credentials for the cloud backend |
| **Complexity** | Right tool for complex IoT requirements | Most complex pattern — requires expertise in distributed systems, edge runtime (K3s, Greengrass), and sync protocols | ❌ A team new to IoT should not start here — the learning curve delays initial delivery by months |
| **AI at the edge** | On-device ML inference eliminates cloud round-trip for AI decisions | Model updates require OTA delivery to potentially thousands of edge nodes | ✅ YOLO object detection on Jetson: 30fps locally, no cloud needed. ❌ Retraining and deploying a new model to 1,000 edge nodes takes hours |

**When Edge-Cloud Hybrid breaks down in IoT:**
- Teams without distributed systems experience: synchronization bugs are subtle and catastrophic (double-execution of commands, lost updates)
- CapEx-constrained projects: edge hardware per site is a significant upfront cost that is hard to justify for non-critical monitoring
- Low-value data with high volume: if the data is non-critical and cloud can handle the volume at reasonable cost, adding edge complexity provides no benefit

### Best For
- Industrial IoT (IIoT) with safety requirements
- Systems requiring offline operation
- Video/image processing workloads
- Latency-sensitive applications (< 10ms response)

---

## Cross-Pattern Trade-off Comparison

### Comprehensive Trade-off Matrix

The table below compares all five patterns across 10 dimensions critical to IoT systems. Ratings are relative (Low / Medium / High / Very High) and assume a production IoT deployment.

| Dimension | Layered | Microservices | EDA | Serverless | Edge-Cloud Hybrid |
|---|---|---|---|---|---|
| **Initial complexity** | Low | High | Medium | Low | Very High |
| **Operational complexity** | Low | Very High | High | Low | Very High |
| **Scalability** | Low | High | Very High | Very High | High |
| **Latency** | Medium | Medium | Low | Variable* | Very Low |
| **Offline resilience** | None | None | None | None | Full |
| **Bandwidth efficiency** | None | None | None | None | Very High |
| **Team size needed** | 1–3 | 5+ | 3–5 | 1–3 | 5+ |
| **Upfront cost** | Low | Medium | Medium | Very Low | High (edge HW) |
| **Ongoing cost (scale)** | High (vertical) | Medium | Medium | High** | Medium |
| **Debug/observability** | Easy | Hard | Very Hard | Hard | Very Hard |

\* Serverless latency: low for warm functions (< 10ms), high for cold starts (100ms–2s)
\** Serverless ongoing cost exceeds containers above ~1M invocations/day at sustained load

### Trade-off Summary by Scenario

**Scenario 1: Startup building first IoT product**
- ✅ Start with **Layered** or **Serverless** — minimize operational burden, ship fast
- ❌ Avoid Microservices and Edge-Cloud Hybrid — premature complexity kills velocity
- Migrate path: Layered → Microservices → EDA as scale demands

**Scenario 2: Industrial safety system (factory floor)**
- ✅ **Edge-Cloud Hybrid** is mandatory — cloud latency is unacceptable for safety shutoffs
- ✅ Combine with **EDA** for the cloud backend — high-throughput event processing
- ❌ Serverless is unsuitable — cold starts are incompatible with < 100ms safety requirements

**Scenario 3: Consumer IoT platform (smart home, wearables)**
- ✅ **Microservices + EDA** — diverse feature set needs independent scaling and deployment
- ✅ **Serverless** for alerting and notifications — sporadic volume, cost-sensitive
- ❌ Edge-Cloud Hybrid adds cost without proportional benefit unless offline operation is required

**Scenario 4: Smart city / large-scale monitoring**
- ✅ **Edge-Cloud Hybrid** for latency-sensitive subsystems (traffic signals, emergency response)
- ✅ **EDA** as the city-wide backbone — millions of heterogeneous event sources
- ✅ **Microservices** for the cloud platform — independent teams, independent scaling
- This is the "combine all patterns" scenario — justified only at city scale

**Scenario 5: Healthcare IoT (wearables, remote monitoring)**
- ✅ **EDA** for event-driven vital sign alerts
- ✅ **Serverless** for alert delivery (sporadic, must not fail)
- ✅ **Edge-Cloud Hybrid** if offline operation is required (remote patient programs)
- Security and compliance characteristics override pure performance optimization

### The Pattern Evolution Path

IoT systems typically evolve through patterns as they grow. **Do not start at the right — start at the left and migrate rightward when the pain of the current pattern exceeds the cost of migration:**

```
Stage 1: POC/MVP        Stage 2: Growth         Stage 3: Scale          Stage 4: Enterprise
┌──────────────┐        ┌──────────────┐         ┌──────────────┐        ┌──────────────┐
│   Layered    │ ──→    │ Microservices│ ──→      │ Microservices│ ──→    │ Edge-Cloud   │
│  Monolith    │ scale  │     +        │ latency  │    + EDA     │ latency│  Hybrid +    │
│              │        │  Serverless  │          │              │        │  Microservices│
│ < 1K devices │        │ 1K-100K devs │          │ 100K+ devs   │        │ + EDA        │
└──────────────┘        └──────────────┘         └──────────────┘        └──────────────┘
```

**Migration triggers:**
- Layered → Microservices: service deployment conflicts, independent scaling needed, > 5 engineers
- Microservices → +EDA: inter-service REST calls causing latency cascades, throughput > 100K events/sec
- +EDA → +Edge: latency requirements drop below 100ms, offline operation required, bandwidth costs significant

---

## Pattern Selection Framework

> **Start with your characteristics profile.** Before using this matrix, identify your top 2–3 architecture characteristics (quality attributes) from [06-architecture-characteristics.md](06-architecture-characteristics.md). The two axes of this matrix — scale and latency — correspond to the **Scalability** and **Performance** characteristics respectively. Other characteristics (Reliability, Security, Energy Efficiency) further refine the choice within each quadrant.

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
