# 📊 Presentation Outline (Short Version) — Software Architecture for IoT

> Slide-by-slide outline for a **~50-minute seminar**
> Covers 4 core topics: Introduction · Architecture Characteristics · Architecture Styles & Tradeoffs · Case Studies
> Full 90-minute version: see [PRESENTATION.md](PRESENTATION.md)

---

## Part 1: Introduction to IoT (8 min)

### Slide 1 — Title
- **Software Architecture for IoT**
- Subtitle: *Designing Scalable, Secure, and Resilient Connected Systems*
- Speaker name, date, organization

### Slide 2 — What is IoT?
- Definition: Network of physical devices embedded with sensors, software, and connectivity to exchange data — autonomously, without direct human intervention
- Scale:
  - **~20 billion** connected devices today (2026)
  - **75+ billion** devices projected by 2030
  - **$1.5 trillion** global IoT market by 2030 (25% CAGR)
- Key application domains:
  | Domain | Examples |
  |---|---|
  | **Industrial (IIoT)** | Predictive maintenance, factory automation |
  | **Smart Home** | Lighting, HVAC, security systems |
  | **Smart City** | Traffic management, energy grids |
  | **Healthcare** | Remote patient monitoring, wearables |
  | **Agriculture** | Precision irrigation, livestock tracking |
  | **Logistics** | Asset tracking, cold-chain management |

### Slide 3 — Why Architecture Matters in IoT
- IoT is not just "connecting things" — it's managing **complexity at scale** across devices, networks, and cloud
- Unique challenges that don't exist in traditional web software:

  | Challenge | Description |
  |---|---|
  | **Heterogeneity** | Hundreds of device types — different hardware, OS, protocols |
  | **Massive scale** | Millions of devices sending data continuously |
  | **Unreliable connectivity** | Wireless networks disconnect; high latency |
  | **Real-time requirements** | Physical processes demand millisecond responses |
  | **Resource constraints** | Weak CPU, KB-scale RAM, limited battery |
  | **Large attack surface** | Every device is a potential entry point |
  | **Long lifecycle** | Devices in the field for 10–15 years |

- **Key formula:** `Bad architecture × Number of devices = Exponential technical debt`
- This is why architecture characteristics must be identified **before** selecting patterns

---

## Part 2: Architecture Characteristics (10 min)

### Slide 4 — What Are Architecture Characteristics?
- Two types of requirements in any software system:
  - **Functional requirements**: What does the system *do*? — "Collect temperature every 30 seconds"
  - **Architecture characteristics**: What must the system be *good at*? — "Handle 1M concurrent connections"
- **Architecture characteristics** (= quality attributes / non-functional requirements) are measurable system properties that force structural decisions
- They are **not features** — they describe *fitness for purpose*
- The correct design flow:

  ```
  Business Requirements (IoT domain)
           ↓
  Architecture Characteristics  ← (this section)
           ↓
  Architecture Pattern Selection  ← (next section)
           ↓
  Technology Implementation
  ```

- Why IoT characteristics differ from enterprise software:
  - IoT operates simultaneously across 3 tiers: **physical device, edge, cloud**
  - Each tier creates unique characteristic pressure not found in web applications

### Slide 5 — The 8 IoT Architecture Characteristics
- Reference card: characteristics that drive structural decisions in IoT

  | # | Characteristic | IoT-Specific Angle | Key Metric |
  |---|---|---|---|
  | 1 | **Scalability** | Two-dimensional: connections AND data volume | Concurrent devices, events/sec |
  | 2 | **Reliability** | Device-level offline autonomy — not just cloud uptime | Uptime %, message delivery rate |
  | 3 | **Performance** | Hard physical-world deadlines — milliseconds matter | End-to-end latency, events/sec |
  | 4 | **Security** | Physical attack surface — cannot be added later | CVE response time, cert rotation |
  | 5 | **Maintainability** | OTA updates for 10-year devices in the field | OTA success rate, rollback rate |
  | 6 | **Interoperability** | MQTT + BLE + LoRaWAN + Modbus simultaneously | Protocols supported, integration time |
  | 7 | **Observability** | Cannot SSH into a sensor — must be designed in | MTTD, MTTR, false alert rate |
  | 8 | **Energy Efficiency** | AA battery must last 5 years | Battery lifetime, avg mW |

### Slide 6 — Key Architecture Characteristic Tradeoffs
- You cannot optimize all characteristics simultaneously — make **intentional tradeoffs**
- Four critical tensions in IoT:

  | Tension | Conflict | Resolution Strategy |
  |---|---|---|
  | **Security ↔ Performance** | TLS adds 50–150ms handshake overhead | Hardware crypto (AES-NI), TLS session resumption |
  | **Security ↔ Energy** | AES-256 increases CPU cycles and power draw | Crypto coprocessor (TPM, ATECC608A) |
  | **Reliability (QoS 2) ↔ Throughput** | QoS 2 requires 4-message handshake per delivery | Use QoS 1 for telemetry, QoS 2 for critical commands |
  | **Observability ↔ Energy** | Continuous logging requires radio transmission | Batch logs; send on scheduled TX windows |

- **Rule:** "Good architecture does not maximize all characteristics — it makes intentional tradeoffs aligned with business priorities."
- Identify your **top 2–3 characteristics** → they guide your pattern selection

---

## Part 3: Architecture Styles & Tradeoffs (15 min)

### Slide 7 — Five Architecture Patterns for IoT
- No single pattern fits all IoT scenarios
- Pattern selection flows from your characteristics profile:

  | Top Priority Characteristic | Recommended Starting Pattern |
  |---|---|
  | Performance < 100ms | Edge-Cloud Hybrid |
  | Scale > 100K devices | Microservices + EDA |
  | Reliability (offline autonomy) | Edge-Cloud Hybrid |
  | Energy (battery-powered devices) | EDA + MQTT low QoS |
  | Interoperability (multi-protocol) | EDA + gateway bridge |

- **"Choose based on requirements, not hype"**
- Five patterns we'll cover: **Layered → Microservices → EDA → Serverless → Edge-Cloud Hybrid**

### Slide 8 — Layered Architecture
- **What it is:** System organized into horizontal layers (Presentation / Business Logic / Data Access / Infrastructure), communicating vertically between adjacent layers
- **When to use:** POC, prototyping, small teams (1–3 people), < 100 devices

  | Dimension | Pros | Cons | IoT Context |
  |---|---|---|---|
  | **Complexity** | Simple — low barrier to entry | Becomes "big ball of mud" at scale | ✅ Ideal for POC; ❌ breaks at ~1K devices |
  | **Coupling** | Clear layer separation | Tight coupling *between* layers — cascading changes | ❌ New sensor type requires changes across all layers |
  | **Scalability** | Easy to reason about | Cannot scale layers independently | ❌ Must scale entire application to unblock one bottleneck |
  | **Team fit** | Any developer can contribute | Not suited for multi-team development | ✅ 1–3 people; ❌ creates merge conflicts at larger teams |

- **When it breaks down in IoT:** Device count > 1,000 · Real-time < 100ms · Multiple parallel teams

### Slide 9 — Microservices Architecture
- **What it is:** System decomposed into small, independently deployable services, each owning a specific domain capability and its own database
- **When to use:** Large-scale IoT platforms (10K+ devices), multi-team organizations

  | Dimension | Pros | Cons | IoT Context |
  |---|---|---|---|
  | **Scalability** | Scale each service independently | More infrastructure to manage | ✅ Telemetry Ingestion scales to 10M msg/sec; OTA stays minimal |
  | **Fault isolation** | One service crash doesn't cascade | Requires circuit breakers and timeouts | ✅ Analytics crash doesn't affect device connectivity |
  | **Operations** | Teams deploy independently | Kubernetes + mesh + observability stack is costly | ❌ Small teams spend 40–60% effort on operations |
  | **Latency** | Purpose-optimized services | Network hops add 10–30ms each | ❌ A 3-hop command path may miss a 100ms deadline |

- **When it breaks down:** Teams < 5 engineers · Early-stage product · Latency-critical paths < 10ms

### Slide 10 — Event-Driven Architecture (EDA)
- **What it is:** Components communicate by producing and consuming events via a central broker; IoT is naturally event-driven — sensors constantly generate events
- **When to use:** Real-time monitoring, high throughput (millions of events/sec), audit trails

  ```
  Sensors ──publish──▶ [Event Broker]  ──deliver──▶ Consumers
                       (Kafka / MQTT / RabbitMQ)    ├── Alert Service
                                                    ├── Time-series DB
                                                    └── ML Pipeline
  ```

  | Dimension | Pros | Cons | IoT Context |
  |---|---|---|---|
  | **Coupling** | Producers and consumers fully decoupled | Schema changes break consumers silently | ✅ Add ML consumer without touching sensor firmware |
  | **Scalability** | Kafka handles 1M+ events/sec | Broker is single point of failure if not clustered | ✅ IoT telemetry peaks naturally absorbed |
  | **Consistency** | Event log enables replay and recovery | Eventual consistency — consumers may lag 50–500ms | ❌ Dashboard may show stale state; ✅ OK for monitoring |
  | **Debugging** | Full forensic replay available | Async chain tracing is complex — needs distributed tracing | ❌ A missed alert could be producer, broker, or consumer |

- **When it breaks down:** Command-and-control paths needing instant response · Small projects · Read-your-own-writes consistency needed

### Slide 11 — Serverless Architecture
- **What it is:** Application logic runs in stateless, event-triggered functions managed by a cloud provider — no server management
- **When to use:** Alerting pipelines, data transformation, sporadic workloads, small teams

  ```
  [IoT Device] → [IoT Hub] → trigger → [Lambda/Function] → [Database]
                                  │
                           if anomaly detected
                                  ↓
                          [Lambda] → [SNS/Email Notification]
  ```

  | Dimension | Pros | Cons | IoT Context |
  |---|---|---|---|
  | **Operations** | Zero server management | Vendor lock-in; migrating requires rewrite | ✅ 2-person startup runs alerting for 10K devices without DevOps |
  | **Cost** | Near-zero at zero traffic | Continuous high volume: 5–10× more expensive than containers | ✅ Sporadic alerts ❌ Continuous 1M events/day ingestion |
  | **Latency** | Warm: < 10ms | Cold start: 100ms–2s | ❌ Unacceptable for < 100ms paths ✅ OK for alert notifications |
  | **State** | Forces clean stateless design | Must re-fetch context from DB each invocation | ❌ Device state tracking requires external store (DynamoDB, Redis) |

- **When it breaks down:** High-frequency continuous telemetry · Latency-sensitive paths < 200ms · Complex stateful workflows

### Slide 12 — Edge-Cloud Hybrid Architecture
- **What it is:** Processing distributed between edge nodes (close to devices) and cloud services — the **dominant pattern for enterprise IoT in 2025+**
- **When to use:** Industrial IoT with safety requirements, offline operation, video/image processing

  ```
  ┌──────────── CLOUD ─────────────┐
  │  ML Training │ Long-term Store  │
  │  Fleet Mgmt  │ Dashboard & APIs │
  └──────────────┬─────────────────┘
                 │ Sync
  ┌──────────────┼─────────────────┐
  │          EDGE TIER             │
  │  Rules Engine │ Local DB       │
  │  AI Inference │ Protocol Bridge│
  └──────────────┬─────────────────┘
                 │
       ┌─────────┼─────────┐
  [Sensor]   [Camera]  [Actuator]
  ```

  | Dimension | Pros | Cons | IoT Context |
  |---|---|---|---|
  | **Latency** | Edge processes locally — < 10ms | Adds an architectural tier to manage | ✅ Factory safety shutoff can't wait for cloud round-trip (50–200ms) |
  | **Reliability** | Operates when cloud is unreachable | Post-reconnection sync is complex | ✅ Smart grid node manages load during WAN outage |
  | **Bandwidth** | Send 1–5% of raw data to cloud | Edge hardware costs $500–$5,000+/site | ✅ 500-camera factory: saves 95% cloud bandwidth |
  | **Security** | Network segmentation limits blast radius | Physical security of edge hardware is operator's responsibility | ❌ Attacker with physical access can extract cloud credentials |
  | **Complexity** | Right tool for complex enterprise IoT | Most complex pattern — distributed systems expertise required | ❌ Teams new to IoT should not start here |

- **When it breaks down:** Teams without distributed systems experience · CapEx-constrained projects · Non-critical data cloud can handle at low cost

- **Cross-pattern comparison:**

  | Dimension | Layered | Microservices | EDA | Serverless | Edge-Cloud Hybrid |
  |---|---|---|---|---|---|
  | Initial complexity | Low | High | Medium | Low | Very high |
  | Scalability | Low | High | Very high | Very high | High |
  | Min. latency | Medium | Medium | Low | Variable* | Very low |
  | Offline capability | None | None | None | None | Full |
  | Team size needed | 1–3 | 5+ | 3–5 | 1–3 | 5+ |

  \* Serverless: low (< 10ms warm), high (100ms–2s cold start)

---

## Part 4: Real-World Case Studies (12 min)

### Slide 13 — Case Study: Rolls-Royce IntelligentEngine
- **Company:** Rolls-Royce Holdings plc | **Domain:** Civil aviation jet engine monitoring
- **Scale:** 4,500+ engines continuously monitored; thousands of sensors per engine
- **Business model:** TotalCare "Power by the Hour" — airlines pay per flying hour, not per maintenance event
- **Architecture:** Sensors → ECU (edge preprocessing) → MQTT → Azure IoT Backend → Databricks ML → 24/7 Monitoring Centers
- **Patterns:** Edge-Cloud Hybrid + Digital Twin
- **Dominant characteristics:** Reliability · Performance · Observability
- **Key decisions:**
  - MQTT over satellite/ground links for reliable delivery
  - Edge preprocessing at ECU reduces transmission volume; detects anomalies instantly
  - Data normalization layer standardizes hundreds of engine variants into one analytics format
  - Azure Databricks self-learning models predict component wear and remaining useful life
  - Real-time Digital Twin enables "what-if" simulation before maintenance intervention
- **Results:** Millions in cost avoidance; enables TotalCare commercial model
- **Architecture lesson:** *Business model and architecture are inseparable — the IoT system IS the product*
- **Source:** [Microsoft Customer Story](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks) | [RTInsights](https://www.rtinsights.com/rolls-royce-jet-engine-maintenance-iot/)

### Slide 14 — Case Study: John Deere Precision Agriculture
- **Company:** Deere & Company (John Deere) | **Domain:** Connected agricultural equipment
- **Scale:** 1.5M+ machines connected; 500M acres managed; telemetry every 5 seconds; data doubles annually
- **Architecture:** Machines (sensors + GPS + computer vision) → JDLink (cellular) → Multi-Cloud (AWS + Azure Arc + Private DC) → AI/ML Pipeline → Operations Center
- **Patterns:** Multi-Cloud Hybrid + Edge + Microservices
- **Dominant characteristics:** Scalability · Interoperability · Performance
- **Key AWS services:** DynamoDB, Kinesis (streaming), OpenSearch (analytics), S3
- **Key decisions:**
  - Multi-cloud by design: AWS for IoT/analytics, Azure for manufacturing operations
  - Databricks for individual-plant-level AI (thousands of plants/second/machine)
  - 5-second telemetry balances real-time value vs. cellular bandwidth cost
  - Architecture designed to handle data doubling annually without re-architecting
- **Results:** Individual-plant precision agriculture; variable-rate prescriptions reduce input waste
- **Architecture lesson:** *Design for 10× growth from day one — data volume is exponential, not linear*
- **Source:** [Databricks Blog](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html) | [Harvard D3](https://d3.harvard.edu/platform-digit/submission/farm-to-data-table-john-deere-and-data-in-precision-agriculture/)

### Slide 15 — Case Study: Siemens MindSphere / Insights Hub
- **Company:** Siemens AG | **Domain:** Industrial IoT Platform-as-a-Service
- **Scale:** Multi-tenant global platform; 1,700+ partner ecosystem; supports 200+ industrial protocols
- **Architecture:** Industrial equipment → MindConnect adapters (OPC UA, Modbus, S7, MQTT) → AWS-hosted microservices → Kinesis/Kafka streaming → Customer analytics apps
- **Patterns:** Microservices + EDA + Hybrid Cloud (Public / Private / Cloud-Dedicated)
- **Dominant characteristics:** Scalability · Interoperability · Maintainability
- **Key decisions:**
  - Use AWS managed services rather than self-managing infrastructure — "follow AWS innovation"
  - Offer both Kinesis (managed) and Kafka (self-managed) — customer governance choice
  - Loosely-coupled containerized microservices for independent scaling per capability
  - MindConnect Elements: hardware-agnostic protocol bridge — the real competitive moat
- **Results (Rittal Blue e+ deployment):** 75% energy reduction · 75% carbon reduction · sub-3-month ROI
- **Architecture lesson:** *Protocol interoperability is the real competitive moat — harder to replicate than the cloud analytics stack*
- **Source:** [AWS re:Invent 2019 MFG202](https://d1.awsstatic.com/events/reinvent/2019/Building_on_AWS_The_architecture_of_the_Siemens_MindSphere_platform_MFG202.pdf) | [AWS Case Study](https://aws.amazon.com/solutions/case-studies/siemens-mindsphere/)

---

## Part 5: Closing (5 min)

### Slide 16 — Architecture Lessons from the Real World
- All three companies confirm the same pattern: **characteristics first, then pattern**

  | | Rolls-Royce | John Deere | Siemens |
  |---|---|---|---|
  | **Dominant characteristics** | Reliability · Observability | Scalability · Interoperability | Interoperability · Scalability |
  | **Pattern used** | Edge-Cloud Hybrid + Digital Twin | Multi-Cloud Hybrid + Microservices | Microservices + EDA + Hybrid Cloud |
  | **Critical decision** | Digital Twin + Azure Databricks | Multi-cloud by workload type | 200+ protocol support via MindConnect |
  | **Architecture lesson** | IoT system = the product | Design for 10× growth day 1 | Protocol moat > analytics stack |

- **No system used only one pattern** — all are hybrids matched to their characteristic priorities

### Slide 17 — Key Takeaways
1. **IoT challenges are structural** — heterogeneity, scale, offline operation, long lifecycle create unique architecture pressures
2. **Identify architecture characteristics before selecting patterns** — they are the bridge between business requirements and structural decisions
3. **Every pattern has a breaking point** — know when Layered breaks (~1K devices), when Serverless breaks (continuous high volume), when Edge-Cloud breaks (inexperienced team)
4. **Real-world systems combine patterns** — Rolls-Royce, John Deere, and Siemens all use hybrid combinations tuned to their top 2–3 characteristics
5. **Good architecture enables new business models** — "Power by the Hour," plant-level precision agriculture, and 200-protocol industrial PaaS are architecture-dependent business innovations

### Slide 18 — Q&A
- Open discussion
- Discussion prompts:
  - *What IoT architecture challenges does your team currently face?*
  - *Given your system's constraints, which 2–3 architecture characteristics matter most?*
  - *Which pattern does your team use today — and is it the right fit for your scale?*

---

## Speaker Notes

### Timing Guide
| Section | Target Time | Cumulative | Slides |
|---|---|---|---|
| Introduction to IoT | 8 min | 8 min | 1–3 |
| Architecture Characteristics | 10 min | 18 min | 4–6 |
| Architecture Styles & Tradeoffs | 15 min | 33 min | 7–12 |
| Real-World Case Studies | 12 min | 45 min | 13–15 |
| Closing & Q&A | 5 min | 50 min | 16–18 |
| **Total** | **50 min** | | **18 slides** |

### Tips for Delivery
- **Slide 3 → Slide 4 transition**: "These challenges are why we need a framework for thinking about what the system must be *good at* — not just what it does. That framework is architecture characteristics."
- **Slide 6**: Ask the audience: "Which of these 4 tradeoffs do you think is the hardest to get right in practice?"
- **Slide 7**: Use the mapping table as an interactive exercise — "What's your top characteristic? Find your starting pattern."
- **Slides 8–12**: For each pattern, pause after the "When it breaks down" bullet and ask if anyone has experienced this in their own projects
- **Slides 13–15**: For each case study, highlight the connection back to characteristics: "Notice how Rolls-Royce's dominant characteristic — Reliability — directly leads to their Edge-Cloud Hybrid choice"
- **Slide 17**: Map each takeaway back to a specific slide or case study for reinforcement
