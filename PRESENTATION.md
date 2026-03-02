# 📊 Presentation Outline — Software Architecture for IoT

> Slide-by-slide outline for a ~90-minute seminar

---

## Part 1: Introduction & IoT Landscape (10 min)

### Slide 1 — Title
- **Software Architecture for IoT**
- Subtitle: *Designing Scalable, Secure, and Resilient Connected Systems*
- Speaker name, date, organization

### Slide 2 — What is IoT?
- Definition: Network of physical devices embedded with sensors, software, and connectivity to exchange data
- Scale: **75+ billion connected devices projected by 2030**
- Key stat: IoT market expected to reach **$1.5 trillion by 2030**
- Visual: Diagram showing device diversity (industrial sensors, wearables, vehicles, smart home)

### Slide 3 — Why Does Architecture Matter?
- IoT is not just "connecting things" — it's about managing **complexity at scale**
- Unique challenges:
  - Heterogeneous devices with varying capabilities
  - Unreliable network connectivity
  - Massive data volumes (billions of events/day)
  - Strict latency requirements (milliseconds in industrial)
  - Security surface area is enormous
- **Bad architecture = technical debt × number of devices**

### Slide 4 — Seminar Roadmap
- Visual agenda overview (7 sections):
  1. Introduction & IoT Landscape (10 min)
  2. IoT Reference Architecture (15 min)
  3. **Architecture Characteristics** (10 min) ← new
  4. Architecture Patterns (15 min)
  5. Communication Protocols (10 min)
  6. Security & Privacy (10 min)
  7. Case Studies & Trends (15 min)
- What participants will take away

---

## Part 2: IoT Reference Architecture (15 min)

### Slide 5 — The 4-Layer IoT Reference Architecture
- **Perception Layer** (Devices & Sensors)
- **Network Layer** (Connectivity & Transport)
- **Edge/Processing Layer** (Pre-processing & Local Logic)
- **Application Layer** (Business Logic, Analytics, UI)
- Diagram: Layered stack with example components at each layer

### Slide 6 — Perception Layer Deep-Dive
- Sensors: temperature, humidity, motion, GPS, camera
- Actuators: motors, valves, switches, displays
- Embedded firmware: FreeRTOS, Zephyr, Arduino
- Key constraints: power, memory, compute, cost
- Example: Industrial temperature sensor (ESP32 + DS18B20)

### Slide 7 — Network Layer Deep-Dive
- Short-range: BLE, Zigbee, Z-Wave, Thread
- Long-range: LoRaWAN, NB-IoT, LTE-M, 5G
- Internet protocols: WiFi, Ethernet
- Satellite IoT for remote assets
- **Protocol selection matrix** (range vs. power vs. bandwidth vs. cost)

### Slide 8 — Edge/Processing Layer Deep-Dive
- IoT Gateway: protocol translation, data aggregation, local rules
- Edge computing: latency reduction, bandwidth optimization, offline capability
- Technologies: AWS Greengrass, Azure IoT Edge, K3s, KubeEdge
- Key benefit: **Process 80% of data locally, send 20% to cloud**

### Slide 9 — Application Layer Deep-Dive
- Cloud platforms: AWS IoT Core, Azure IoT Hub, Google Cloud IoT
- Data pipeline: Ingestion → Processing → Storage → Analytics → Visualization
- Databases: Time-series (InfluxDB, TimescaleDB), Document (MongoDB), Relational
- Dashboards & APIs for end users
- AI/ML for predictive analytics

### Slide 10 — Layer Interactions & Data Flow
- Full data flow diagram from sensor to dashboard
- Bidirectional communication: commands flow downstream, telemetry flows upstream
- Highlight: Where data transformation happens at each layer

---

## Part 3: Architecture Characteristics (10 min)

### Slide 11 — What Are Architecture Characteristics?
- Before choosing a pattern, define **what the system must be good at**
- Two distinct question types:
  - **Functional requirements:** *What does the system do?* → "Collect temperature every 30 seconds"
  - **Architecture characteristics:** *What must the system be good at?* → "Handle 1M simultaneous connections"
- Characteristics are **measurable quality attributes** that constrain structural decisions
- **The correct design flow:**
  ```
  IoT Requirements → [Architecture Characteristics] → Pattern Selection
  ```
- Common mistake: jumping straight to technology choices before defining what the system needs to be

> *Ask audience: Before today, how did you decide which architecture pattern to use?*

### Slide 12 — The 8 IoT Architecture Characteristics

| # | Characteristic | IoT-Specific Definition | Example Metric |
|---|---|---|---|
| 1 | **Scalability** | Handle connections AND data volume growth independently | 100 → 10M devices without re-architecture |
| 2 | **Reliability** | Device autonomy when cloud is unreachable | 99.99% uptime; offline operation continues |
| 3 | **Performance** | Hard physical-world deadlines, not just user perception | < 10ms edge response for safety systems |
| 4 | **Security** | Physical attack surface + structural requirement | Zero Trust; cannot be bolted on later |
| 5 | **Maintainability** | OTA updates for devices you cannot physically reach | 10-year device lifespan with remote patches |
| 6 | **Interoperability** | Speak MQTT + BLE + LoRaWAN + Modbus simultaneously | Multi-protocol gateway bridging |
| 7 | **Observability** | Infer device state without physical access | Monitor millions of sensors you can't SSH into |
| 8 | **Energy Efficiency** | Battery-powered devices lasting years | AA battery → 5 years in the field |

### Slide 13 — Characteristics Tradeoffs: You Can't Have Everything

Four key tensions in IoT systems:

| Characteristic A | ↔ | Characteristic B | Tension | Resolution |
|---|---|---|---|---|
| **Security** (TLS) | | **Performance** (latency) | TLS adds 50–150ms handshake | Hardware crypto accelerators; session resumption |
| **Reliability** (QoS 2) | | **Performance** (throughput) | 4-message handshake per delivery | QoS 1 for telemetry; QoS 2 for commands only |
| **Observability** (logging) | | **Energy Efficiency** | Logging requires radio TX | Batch logs; transmit on scheduled windows |
| **Scalability** (microservices) | | **Maintainability** | More services = more complexity | Invest in Kubernetes + GitOps from day one |

> *"Good architecture is not maximizing all characteristics — it's making explicit, deliberate tradeoffs aligned with business priorities."*

### Slide 14 — Characteristics → Pattern Selection

Your characteristics profile determines your starting pattern:

| If your top priority is... | Start with this pattern | Because... |
|---|---|---|
| Latency < 100ms | **Edge-Cloud Hybrid** | Cloud cannot be in the critical path |
| Scale > 100K devices | **Microservices + EDA** | Independent horizontal scaling |
| Offline reliability | **Edge-Cloud Hybrid** | Local rules engine without cloud |
| Cost efficiency / sporadic | **Serverless** | Pay-per-invocation, auto-scaling |
| Regulated security | Any pattern + **Zero Trust** | Security is cross-cutting, not pattern-specific |

**Coming up next:** Each pattern addresses specific characteristics — we'll see exactly which ones.

---

## Part 4: Architecture Patterns (15 min)

### Slide 15 — Pattern Overview
- Why patterns matter: proven solutions to recurring problems
- Categories we'll cover:
  - Layered Architecture
  - Microservices
  - Event-Driven Architecture (EDA)
  - Serverless
  - Edge-Cloud Hybrid

### Slide 16 — Microservices for IoT
- *Primary characteristics addressed: Scalability, Maintainability (OTA Update Service)*
- Break IoT backend into independent services:
  - Device Management Service
  - Telemetry Ingestion Service
  - Alert/Notification Service
  - Analytics Service
  - OTA Update Service
- Benefits: independent scaling, tech diversity, fault isolation
- Challenges: operational complexity, network latency, data consistency
- **Diagram: Microservices topology for smart factory**

### Slide 17 — Event-Driven Architecture (EDA)
- *Primary characteristics addressed: Performance (throughput), Scalability, Interoperability, Observability*
- IoT is inherently event-driven: sensors produce events continuously
- Publish/Subscribe model fits naturally
- Event broker: Apache Kafka, RabbitMQ, MQTT broker
- Event sourcing for audit trails and replay
- CQRS for separating read/write workloads
- **Diagram: Event flow from sensor → broker → consumers**

### Slide 18 — Serverless IoT
- *Primary characteristics addressed: Scalability (auto), cost efficiency*
- AWS Lambda / Azure Functions triggered by IoT events
- Pay-per-invocation: cost-efficient for sporadic workloads
- Great for: alerts, data transformations, webhook integrations
- Limitations: cold starts, execution time limits, vendor lock-in
- **When to use**: Unpredictable traffic, simple transformations

### Slide 19 — Edge-Cloud Hybrid Architecture
- *Primary characteristics addressed: Performance (latency), Reliability (offline), Energy Efficiency*
- The dominant pattern for modern IoT (2025+)
- Edge handles: real-time processing, local decisions, offline operation
- Cloud handles: long-term storage, ML training, fleet management, analytics
- Synchronization patterns: store-and-forward, eventual consistency
- **Diagram: Hybrid architecture with data flow**

### Slide 20 — Trade-off Comparison & Pattern Selection

**Cross-pattern trade-off matrix:**

| Dimension | Layered | Microservices | EDA | Serverless | Edge-Cloud Hybrid |
|---|---|---|---|---|---|
| Initial complexity | Low | High | Medium | Low | Very High |
| Scalability | Low | High | Very High | Very High | High |
| Latency | Medium | Medium | Low | Variable* | Very Low |
| Offline resilience | ❌ | ❌ | ❌ | ❌ | ✅ |
| Operational burden | Low | Very High | High | Low | Very High |
| Team size needed | 1–3 | 5+ | 3–5 | 1–3 | 5+ |

\* Serverless cold start 100ms–2s; warm < 10ms

**Pattern evolution path — start left, migrate right when current pattern hurts:**
```
POC → Layered → Microservices + Serverless → + EDA → + Edge-Cloud Hybrid
       < 1K         1K–100K devices            100K+     Enterprise/IIoT
```

**When each pattern breaks down:**
- **Layered:** device count > 1K; latency < 100ms; multiple teams
- **Microservices:** team < 5; early-stage (wrong service boundaries); latency < 10ms across services
- **EDA:** command-and-control paths; strict consistency required; small projects
- **Serverless:** continuous high-volume ingestion; latency < 200ms paths
- **Edge-Cloud Hybrid:** team lacks distributed systems expertise; CapEx-constrained

---

## Part 5: Communication Protocols (10 min)

### Slide 21 — Protocol Landscape
- IoT has many protocol options — choosing the right one is critical
- Categories: Application layer, Transport layer, Network layer
- Focus on application-layer protocols: MQTT, CoAP, AMQP, HTTP/REST

### Slide 22 — MQTT Deep-Dive
- **Message Queuing Telemetry Transport**
- Publish/Subscribe model with a central broker
- Designed for: low bandwidth, unreliable networks, constrained devices
- QoS levels: 0 (at most once), 1 (at least once), 2 (exactly once)
- Topic hierarchy: `factory/floor1/machine3/temperature`
- Retained messages, Last Will and Testament (LWT)
- **De facto standard for IoT messaging**

### Slide 23 — MQTT Architecture Diagram
- Visual: Devices → MQTT Broker → Subscribers (services, dashboards, rules engine)
- Show topic structure and QoS flow
- Clustered broker for high availability

### Slide 24 — CoAP, AMQP, HTTP Comparison
| Protocol | Transport | Model | Overhead | Best For |
|---|---|---|---|---|
| MQTT | TCP | Pub/Sub | Very Low | Telemetry, events |
| CoAP | UDP | Req/Res | Very Low | Constrained devices |
| AMQP | TCP | Pub/Sub + Queue | Medium | Enterprise integration |
| HTTP/REST | TCP | Req/Res | High | Web APIs, configuration |
| WebSocket | TCP | Bidirectional | Medium | Real-time dashboards |

### Slide 25 — Protocol Selection Decision Tree
- Flowchart: 
  - Is device severely constrained? → CoAP
  - Need pub/sub with reliable delivery? → MQTT
  - Enterprise messaging with routing? → AMQP
  - Simple request/response with web compatibility? → HTTP
  - Real-time bidirectional in browser? → WebSocket

---

## Part 6: Security & Privacy (10 min)

### Slide 26 — IoT Security Landscape
- IoT expands the attack surface dramatically
- Common vulnerabilities: default credentials, unencrypted communication, no OTA updates
- **Real incidents**: Mirai botnet (2016), casino fish tank hack, industrial SCADA attacks
- Security must be "by design", not an afterthought

### Slide 27 — Security at Every Layer
| Layer | Threats | Mitigations |
|---|---|---|
| Perception | Physical tampering, cloning | Secure boot, hardware security modules (HSM) |
| Network | Eavesdropping, MITM, replay | TLS/DTLS, certificate pinning, VPN |
| Edge | Unauthorized access, malware | Firewalls, container isolation, signed firmware |
| Application | Data breach, injection, DDoS | OAuth 2.0, rate limiting, WAF, encryption at rest |

### Slide 28 — Zero Trust for IoT
- Never trust, always verify — even internal devices
- Unique device identity (X.509 certificates, device tokens)
- Mutual TLS (mTLS) for device-to-cloud authentication
- Role-Based Access Control (RBAC) for topic/resource permissions
- Continuous monitoring and anomaly detection
- Secure OTA updates with signed firmware images

### Slide 29 — Data Privacy
- GDPR, CCPA, and IoT-specific regulations
- Data minimization: collect only what you need
- Edge processing to keep sensitive data local
- Anonymization and pseudonymization techniques
- Data lifecycle management: retention policies, right to deletion

---

## Part 7: Case Studies & Emerging Trends (15 min)

### Slide 30 — Case Study: Rolls-Royce IntelligentEngine
- **Company:** Rolls-Royce | **Domain:** Civil aviation jet engine monitoring
- **Scale:** 4,500+ engines continuously monitored; thousands of sensors per engine
- **Architecture:** Sensors → ECU (edge preprocessing) → MQTT → Azure IoT Backend → Databricks ML → 24/7 Monitoring Centers
- **Patterns:** Edge-Cloud Hybrid + Digital Twin
- **Key decisions:**
  - MQTT over satellite/ground links for reliable delivery
  - Edge preprocessing at ECU reduces transmission volume
  - Data normalization layer standardizes diverse engine variants
  - Azure Databricks for self-learning predictive maintenance models
- **Results:** Millions in cost avoidance; enables TotalCare "Power by the Hour" commercial model
- **Architecture lesson:** *Business model and architecture are inseparable — the IoT system IS the product*
- **Source:** [Microsoft Customer Story](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks) | [RTInsights](https://www.rtinsights.com/rolls-royce-jet-engine-maintenance-iot/)

### Slide 31 — Case Study: John Deere Precision Agriculture
- **Company:** Deere & Company (John Deere) | **Domain:** Connected agricultural equipment
- **Scale:** 1.5M+ machines connected; 500M acres managed; telemetry every 5 seconds; data doubles annually
- **Architecture:** Machines (sensors + GPS + computer vision) → JDLink (cellular) → Multi-Cloud (AWS + Azure Arc + Private DC) → AI/ML Pipeline → Operations Center
- **Patterns:** Multi-Cloud Hybrid + Edge + Microservices
- **Key AWS services:** DynamoDB, Kinesis (streaming), OpenSearch (analytics), S3
- **Key decisions:**
  - Multi-cloud by design: AWS for IoT/analytics, Azure for manufacturing operations
  - Databricks for individual-plant-level AI (thousands of plants/second/machine)
  - 5-second telemetry balances real-time value vs. cellular bandwidth
- **Results:** Individual plant precision agriculture; variable-rate prescriptions reduce input waste
- **Architecture lesson:** *Design for 10× growth from day one; data volume is exponential, not linear*
- **Source:** [Databricks Blog](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html) | [Harvard D3](https://d3.harvard.edu/platform-digit/submission/farm-to-data-table-john-deere-and-data-in-precision-agriculture/)

### Slide 32 — Case Study: Siemens MindSphere / Insights Hub
- **Company:** Siemens AG | **Domain:** Industrial IoT Platform-as-a-Service
- **Scale:** Multi-tenant global platform; 1,700+ partner ecosystem; supports 200+ industrial protocols
- **Architecture:** Industrial equipment → MindConnect adapters (OPC UA, Modbus, S7, MQTT) → AWS-hosted microservices → Kinesis/Kafka streaming → Customer analytics apps
- **Patterns:** Microservices + EDA + Hybrid Cloud (Public / Private / Cloud-Dedicated)
- **Key decisions:**
  - Use AWS managed services rather than self-managing infrastructure
  - Offer both Kinesis (managed) and Kafka (self-managed) — customer governance choice
  - Loosely-coupled containerized microservices for independent scaling
  - MindConnect Elements: hardware-agnostic protocol bridge
- **Results (Rittal Blue e+ case):** 75% energy reduction; 75% carbon reduction; sub-3-month ROI
- **Architecture lesson:** *Protocol interoperability is the real competitive moat — harder to replicate than the cloud analytics stack*
- **Source:** [AWS re:Invent 2019 MFG202](https://d1.awsstatic.com/events/reinvent/2019/Building_on_AWS_The_architecture_of_the_Siemens_MindSphere_platform_MFG202.pdf) | [AWS Case Study](https://aws.amazon.com/solutions/case-studies/siemens-mindsphere/)

### Slide 33 — Emerging Trends (2025–2026)
- **AIoT (AI + IoT)**: AI models running on edge devices for autonomous decisions
- **Digital Twins**: Real-time virtual replicas of physical assets for simulation and optimization
- **5G + IoT**: Ultra-low latency enabling new use cases (remote surgery, autonomous vehicles)
- **Sustainability**: Energy-efficient architectures, green IoT design
- **Satellite IoT**: Connectivity for truly remote assets (agriculture, maritime, mining)
- **Matter Protocol**: Unified smart home standard

### Slide 34 — Key Takeaways
1. IoT architecture is **layered** — understand each layer's role
2. Identify **architecture characteristics** (quality attributes) before selecting patterns — they are the bridge between requirements and structure
3. Choose patterns based on **requirements**, not hype
4. MQTT is the **de facto standard** for IoT messaging
5. Edge computing is **essential**, not optional
6. Security must be **baked in from day one**
7. AIoT and Digital Twins are **reshaping** the landscape

### Slide 35 — Resources & Further Reading
- Books: "Designing IoT Solutions with Microsoft Azure" (O'Reilly)
- Online: AWS IoT Lens (Well-Architected Framework)
- Communities: Eclipse IoT, MQTT.org, IoT For All
- This repository: all materials and diagrams

### Slide 36 — Q&A
- Open discussion
- Discussion prompts:
  - *What IoT architecture challenges does your team face?*
  - *How do you handle device management at scale?*
  - *Edge vs. Cloud — where do you draw the line?*

---

## Speaker Notes

### Timing Guide
| Section | Target Time | Cumulative | Slides |
|---|---|---|---|
| Introduction | 10 min | 10 min | 1–4 |
| Reference Architecture | 15 min | 25 min | 5–10 |
| **Architecture Characteristics** | **10 min** | **35 min** | **11–14** |
| Architecture Patterns | 15 min | 50 min | 15–20 |
| Communication Protocols | 10 min | 60 min | 21–25 |
| Security & Privacy | 10 min | 70 min | 26–29 |
| Case Studies & Trends | 15 min | 85 min | 30–33 |
| Q&A | 5 min | 90 min | 34–36 |

### Tips for Delivery
- Use the architecture diagrams frequently — IoT is visual
- Ask the audience about their IoT experience early on
- Run a live MQTT demo if time permits (use `mosquitto_pub`/`mosquitto_sub`)
- For the pattern section, use poll: "Which pattern does your team use?"
