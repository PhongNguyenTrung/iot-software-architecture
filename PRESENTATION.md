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
- Visual agenda overview (6 sections)
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

## Part 3: Architecture Patterns (15 min)

### Slide 11 — Pattern Overview
- Why patterns matter: proven solutions to recurring problems
- Categories we'll cover:
  - Layered Architecture
  - Microservices
  - Event-Driven Architecture (EDA)
  - Serverless
  - Edge-Cloud Hybrid

### Slide 12 — Microservices for IoT
- Break IoT backend into independent services:
  - Device Management Service
  - Telemetry Ingestion Service
  - Alert/Notification Service
  - Analytics Service
  - OTA Update Service
- Benefits: independent scaling, tech diversity, fault isolation
- Challenges: operational complexity, network latency, data consistency
- **Diagram: Microservices topology for smart factory**

### Slide 13 — Event-Driven Architecture (EDA)
- IoT is inherently event-driven: sensors produce events continuously
- Publish/Subscribe model fits naturally
- Event broker: Apache Kafka, RabbitMQ, MQTT broker
- Event sourcing for audit trails and replay
- CQRS for separating read/write workloads
- **Diagram: Event flow from sensor → broker → consumers**

### Slide 14 — Serverless IoT
- AWS Lambda / Azure Functions triggered by IoT events
- Pay-per-invocation: cost-efficient for sporadic workloads
- Great for: alerts, data transformations, webhook integrations
- Limitations: cold starts, execution time limits, vendor lock-in
- **When to use**: Unpredictable traffic, simple transformations

### Slide 15 — Edge-Cloud Hybrid Architecture
- The dominant pattern for modern IoT (2025+)
- Edge handles: real-time processing, local decisions, offline operation
- Cloud handles: long-term storage, ML training, fleet management, analytics
- Synchronization patterns: store-and-forward, eventual consistency
- **Diagram: Hybrid architecture with data flow**

### Slide 16 — Pattern Selection Guide
| Criteria | Microservices | EDA | Serverless | Hybrid |
|---|---|---|---|---|
| Latency | Medium | Low | Variable | Very Low |
| Scale | High | Very High | Auto | High |
| Complexity | High | Medium | Low | Very High |
| Cost | Medium | Medium | Low (sporadic) | High |
| Offline support | No | No | No | Yes |
| Best for | Complex backends | Real-time streams | Simple triggers | Critical IoT |

---

## Part 4: Communication Protocols (15 min)

### Slide 17 — Protocol Landscape
- IoT has many protocol options — choosing the right one is critical
- Categories: Application layer, Transport layer, Network layer
- Focus on application-layer protocols: MQTT, CoAP, AMQP, HTTP/REST

### Slide 18 — MQTT Deep-Dive
- **Message Queuing Telemetry Transport**
- Publish/Subscribe model with a central broker
- Designed for: low bandwidth, unreliable networks, constrained devices
- QoS levels: 0 (at most once), 1 (at least once), 2 (exactly once)
- Topic hierarchy: `factory/floor1/machine3/temperature`
- Retained messages, Last Will and Testament (LWT)
- **De facto standard for IoT messaging**

### Slide 19 — MQTT Architecture Diagram
- Visual: Devices → MQTT Broker → Subscribers (services, dashboards, rules engine)
- Show topic structure and QoS flow
- Clustered broker for high availability

### Slide 20 — CoAP, AMQP, HTTP Comparison
| Protocol | Transport | Model | Overhead | Best For |
|---|---|---|---|---|
| MQTT | TCP | Pub/Sub | Very Low | Telemetry, events |
| CoAP | UDP | Req/Res | Very Low | Constrained devices |
| AMQP | TCP | Pub/Sub + Queue | Medium | Enterprise integration |
| HTTP/REST | TCP | Req/Res | High | Web APIs, configuration |
| WebSocket | TCP | Bidirectional | Medium | Real-time dashboards |

### Slide 21 — Protocol Selection Decision Tree
- Flowchart: 
  - Is device severely constrained? → CoAP
  - Need pub/sub with reliable delivery? → MQTT
  - Enterprise messaging with routing? → AMQP
  - Simple request/response with web compatibility? → HTTP
  - Real-time bidirectional in browser? → WebSocket

---

## Part 5: Security & Privacy (10 min)

### Slide 22 — IoT Security Landscape
- IoT expands the attack surface dramatically
- Common vulnerabilities: default credentials, unencrypted communication, no OTA updates
- **Real incidents**: Mirai botnet (2016), casino fish tank hack, industrial SCADA attacks
- Security must be "by design", not an afterthought

### Slide 23 — Security at Every Layer
| Layer | Threats | Mitigations |
|---|---|---|
| Perception | Physical tampering, cloning | Secure boot, hardware security modules (HSM) |
| Network | Eavesdropping, MITM, replay | TLS/DTLS, certificate pinning, VPN |
| Edge | Unauthorized access, malware | Firewalls, container isolation, signed firmware |
| Application | Data breach, injection, DDoS | OAuth 2.0, rate limiting, WAF, encryption at rest |

### Slide 24 — Zero Trust for IoT
- Never trust, always verify — even internal devices
- Unique device identity (X.509 certificates, device tokens)
- Mutual TLS (mTLS) for device-to-cloud authentication
- Role-Based Access Control (RBAC) for topic/resource permissions
- Continuous monitoring and anomaly detection
- Secure OTA updates with signed firmware images

### Slide 25 — Data Privacy
- GDPR, CCPA, and IoT-specific regulations
- Data minimization: collect only what you need
- Edge processing to keep sensitive data local
- Anonymization and pseudonymization techniques
- Data lifecycle management: retention policies, right to deletion

---

## Part 6: Case Studies & Emerging Trends (15 min)

### Slide 26 — Case Study: Smart Factory (Industry 4.0)
- Architecture: 500+ sensors → Edge gateways → MQTT broker → Microservices backend → Dashboard
- Patterns used: Edge-Cloud Hybrid + EDA + Microservices
- Results: 30% reduction in downtime, predictive maintenance saving $2M/year
- Key decisions: MQTT for telemetry, InfluxDB for time-series, Grafana for visualization

### Slide 27 — Case Study: Smart City Traffic Management
- Architecture: Traffic cameras + sensors → Edge AI → Central cloud → Real-time optimization
- Patterns used: Edge computing + AI inference + Event-driven
- Results: 20% improvement in traffic flow, 15% reduction in emissions
- Key insight: Processing video at the edge (privacy + bandwidth)

### Slide 28 — Case Study: Connected Healthcare
- Architecture: Wearable devices → Mobile gateway → HIPAA-compliant cloud → Clinical dashboard
- Patterns used: Serverless + Event-driven + Strict security
- Results: Continuous patient monitoring, 40% faster emergency response
- Key challenge: Regulatory compliance (HIPAA, HL7 FHIR)

### Slide 29 — Emerging Trends (2025–2026)
- **AIoT (AI + IoT)**: AI models running on edge devices for autonomous decisions
- **Digital Twins**: Real-time virtual replicas of physical assets for simulation and optimization
- **5G + IoT**: Ultra-low latency enabling new use cases (remote surgery, autonomous vehicles)
- **Sustainability**: Energy-efficient architectures, green IoT design
- **Satellite IoT**: Connectivity for truly remote assets (agriculture, maritime, mining)
- **Matter Protocol**: Unified smart home standard

### Slide 30 — Key Takeaways
1. IoT architecture is **layered** — understand each layer's role
2. Choose patterns based on **requirements**, not hype
3. MQTT is the **de facto standard** for IoT messaging
4. Edge computing is **essential**, not optional
5. Security must be **baked in from day one**
6. AIoT and Digital Twins are **reshaping** the landscape

### Slide 31 — Resources & Further Reading
- Books: "Designing IoT Solutions with Microsoft Azure" (O'Reilly)
- Online: AWS IoT Lens (Well-Architected Framework)
- Communities: Eclipse IoT, MQTT.org, IoT For All
- This repository: all materials and diagrams

### Slide 32 — Q&A
- Open discussion
- Discussion prompts:
  - *What IoT architecture challenges does your team face?*
  - *How do you handle device management at scale?*
  - *Edge vs. Cloud — where do you draw the line?*

---

## Speaker Notes

### Timing Guide
| Section | Target Time | Cumulative |
|---|---|---|
| Introduction | 10 min | 10 min |
| Reference Architecture | 15 min | 25 min |
| Patterns | 15 min | 40 min |
| Protocols | 15 min | 55 min |
| Security | 10 min | 65 min |
| Case Studies & Trends | 15 min | 80 min |
| Q&A | 10 min | 90 min |

### Tips for Delivery
- Use the architecture diagrams frequently — IoT is visual
- Ask the audience about their IoT experience early on
- Run a live MQTT demo if time permits (use `mosquitto_pub`/`mosquitto_sub`)
- For the pattern section, use poll: "Which pattern does your team use?"
