# IoT Architecture Characteristics

## 1. What Are Architecture Characteristics?

When designing an IoT system, architects face two distinct types of questions:

- **Functional requirements:** *What does the system do?* — "Collect temperature readings every 30 seconds," "Alert when pressure exceeds 150 PSI," "Display live metrics on a dashboard."
- **Architecture characteristics:** *What must the system be good at?* — "Handle 1 million simultaneous device connections," "Continue operating when the cloud is unreachable," "Respond to a safety trigger within 10 milliseconds."

Architecture characteristics (also called **quality attributes** or **non-functional requirements**) are the measurable properties of the system that constrain which structural decisions are valid. They do not describe features — they describe fitness.

In software architecture theory (Richards & Ford, *Fundamentals of Software Architecture*), the correct design flow is:

```
Domain Requirements
       ↓
Architecture Characteristics  ←── this document
       ↓
Architecture Pattern Selection  ←── see 02-patterns.md
       ↓
Technology & Implementation
```

The seminar skipping this middle step is the most common reason IoT projects pick the wrong pattern — and pay for it at scale.

### Why IoT Characteristics Differ from Web/Enterprise Software

IoT systems operate in three tiers simultaneously: **physical devices**, **edge compute**, and **cloud services**. Each tier introduces characteristic pressures that web applications never face:

| Tier | Unique Pressures |
|---|---|
| Device | Power constraints, physical attack surface, 10+ year lifespan, firmware updates |
| Edge | Offline operation, local autonomy, resource-constrained hardware |
| Cloud | Massive connection counts, heterogeneous data formats, fleet management at scale |

An IoT characteristic assessment must consider all three tiers independently.

---

## 2. The 8 IoT Architecture Characteristics

### 2.1 Scalability

**Definition:** The ability of the system to handle growth in device count, data volume, and management operations without requiring re-architecture.

**Why IoT scalability is different from web scalability:** Web systems scale one dimension — concurrent users or requests. IoT scales two dimensions independently:
- **Connection scalability:** 100 devices → 100,000 → 10 million simultaneous MQTT connections
- **Throughput scalability:** 1 reading/device/min → 100 readings/device/sec (7 orders of magnitude)
- **Fleet management scalability:** OTA updates, config pushes, monitoring across a growing fleet

A system designed for 1,000 devices that must handle 1 million is not a configuration change — it is a re-architecture.

**Concrete metrics:**

| Scale Tier | Devices | Events/sec | Implied Architecture |
|---|---|---|---|
| POC/Startup | < 1,000 | < 1,000 | Layered monolith |
| SME | 1,000–100,000 | 1,000–100,000 | Microservices + managed broker |
| Enterprise | 100,000–10M | 100,000–1M+ | Microservices + EDA + horizontal sharding |
| Industrial/City | 10M+ | 10M+ | Edge-Cloud Hybrid + federated brokers |

**Benchmark references:** EMQX broker: 100M MQTT connections on commodity hardware. AWS IoT Core: millions of simultaneous devices. Apache Kafka: 1M+ messages/sec per cluster.

**Critical IoT constraint:** Devices cannot be upgraded to speak a newer, more scalable protocol. Horizontal scaling must happen in the backend — the device API must remain backward-compatible as the platform scales.

> **Pattern driver:** Scalability is the primary reason to migrate from Layered to **Event-Driven Architecture** and eventually **Microservices**. See [02-patterns.md](02-patterns.md).

---

### 2.2 Reliability & Availability

**Definition:** The probability that the system delivers its intended service, including at the device tier when network or cloud connectivity is lost.

**Why IoT reliability is different:** Web system reliability is about the server being up. IoT reliability has two distinct dimensions:
- **Platform availability:** cloud services uptime (99.9% = 8.7 hours downtime/year; 99.99% = 52 min/year)
- **Device-level autonomy:** the device must continue its primary function even when cloud-disconnected

A smart factory assembly line cannot pause because AWS had a 12-minute outage. A patient monitor must alert locally even if cellular is unavailable.

**Key reliability mechanisms at the protocol level:**

| Mechanism | Description | Reliability Impact |
|---|---|---|
| MQTT QoS 0 | At-most-once delivery | Messages may be lost |
| MQTT QoS 1 | At-least-once delivery | Guaranteed delivery, possible duplicate |
| MQTT QoS 2 | Exactly-once delivery | Highest guarantee, highest overhead |
| MQTT LWT | Last Will Testament | Detect device disconnection reliably |
| Store-and-Forward | Buffer messages during outage | No data loss during connectivity gaps |

See [03-protocols.md](03-protocols.md) for full QoS explanation.

**Availability targets by domain:**

| Domain | Minimum Availability | Justification |
|---|---|---|
| Smart Home | 99% | Convenience; interruption tolerable |
| Industrial Monitoring | 99.9% | Production impact if down |
| Safety Systems | 99.99% | Human safety may depend on it |
| Medical Devices | 99.999% | Life-critical in some applications |

> **Pattern driver:** The requirement for offline device autonomy is the primary driver for **Edge-Cloud Hybrid** architecture. Edge nodes maintain local rules engines and continue operating without cloud. See [02-patterns.md](02-patterns.md).

---

### 2.3 Performance (Latency & Throughput)

**Definition:** Latency is the time from event to response. Throughput is the volume of events the system can process per unit time.

**Why IoT performance has hard deadlines:** A web page loading in 3 seconds is annoying. A cooling system taking 3 seconds to respond to an overtemperature reading may destroy equipment. IoT systems couple software to physical processes that have their own timing requirements.

**The 5-tier IoT latency model:**

| Latency Tier | Target | Processing Location | Example Use Case |
|---|---|---|---|
| Hard real-time | < 1 ms | On-device firmware | Motor control PID loop |
| Soft real-time | < 10 ms | Edge gateway | Safety emergency shutoff |
| Near real-time | < 100 ms | Edge compute | Anomaly detection alert |
| Interactive | < 1 sec | Cloud | Dashboard refresh |
| Batch/Analytics | Minutes–Hours | Cloud | ML model training |

**Architecture implication:** If a use case requires < 100ms response, the cloud cannot be in the critical path. Processing must move to the edge or device tier.

**Throughput vs. latency tradeoff:** High throughput often requires batching (which increases latency). Real-time systems must make an explicit choice: optimize for latency (send every event immediately) or throughput (batch events, higher latency).

> **Pattern driver:** Latency requirements are the single most important driver separating **Edge-Cloud Hybrid** from pure cloud architectures. Throughput requirements drive **Event-Driven Architecture** with high-performance brokers (Kafka, EMQX). See [02-patterns.md](02-patterns.md).

---

### 2.4 Security

**Definition:** The degree to which the system protects information and resources from unauthorized access, modification, or disclosure — including at the physical device level.

**Why IoT security is a structural characteristic:** In web software, security controls (authentication, encryption, authorization) can often be layered on after the application is built. In IoT, this is not possible. Decisions made at device manufacture time (secure boot, hardware identity, encrypted storage) cannot be changed in the field. Security in IoT is **architectural** — it must be designed in, not bolted on.

**The physical attack surface:** IoT devices exist in uncontrolled environments. Any device can be physically extracted, firmware dumped, and cloned. This attack vector does not exist in web software.

**Security dimensions per layer:**

| Layer | Key Characteristics |
|---|---|
| Device (Confidentiality) | Encrypted storage, no hardcoded credentials, secure boot |
| Device (Integrity) | Signed firmware, TPM/HSM for key storage, tamper detection |
| Device (Authentication) | Unique X.509 certificate per device, provisioned at manufacture |
| Network (Confidentiality) | TLS 1.3 / DTLS for all transport |
| Network (Integrity) | Message authentication codes, certificate pinning |
| Application (Authorization) | Topic-level MQTT ACL, RBAC for APIs, OAuth 2.0 for users |
| Application (Non-repudiation) | Immutable audit logs, event sourcing |

For the full threat model and mitigation details, see [04-security.md](04-security.md).

**Security interacts with every other characteristic** — this makes it unique:
- Security ↔ Performance: TLS adds 50–150ms handshake latency
- Security ↔ Energy Efficiency: AES-256 increases CPU cycles and power draw
- Security ↔ Maintainability: Certificate rotation must be automated at scale

> **Pattern driver:** Security is a cross-cutting structural concern, not specific to one pattern. However, it is the primary driver for **Zero Trust overlay** on any chosen pattern, and the reason why **Microservices** is preferred over monolith in regulated industries (each service boundary = an enforcement point).

---

### 2.5 Maintainability

**Definition:** The ease with which the system (including physical devices) can be modified, updated, debugged, and extended throughout its operational lifetime.

**Why IoT maintainability is an existential concern:** Web software can be updated by deploying new server code in seconds. IoT devices may be deployed on factory floors, in remote agricultural fields, embedded in infrastructure, or implanted in patients. They cannot be physically reached.

A device shipped in 2026 running on a 5-year battery must still receive security patches in 2031. The backend it connects to must maintain backward-compatible APIs for that entire period.

**Key maintainability dimensions:**

| Dimension | Requirement | Mechanism |
|---|---|---|
| Firmware updates | OTA (over-the-air) without physical access | A/B partition, delta updates, staged rollouts |
| Configuration | Remote config without redeployment | Device twin / shadow (desired vs. reported state) |
| Monitoring | Know what every device is doing | Heartbeat messages, LWT, telemetry |
| Backward compatibility | Old devices still work after backend upgrades | API versioning, protocol abstraction layer |
| Device lifecycle | Manage provisioning → active → dormant → decommission | Device registry with state machine |

**Device twin / shadow pattern:** The cloud maintains a virtual representation of each device's desired configuration and last-reported state. Configuration changes are pushed as desired-state updates; the device applies them when it next connects. This decouples the management plane from device connectivity.

**OTA update risk management:**
- Staged rollouts: update 1% of fleet → monitor → 10% → 100%
- Rollback capability: A/B partition ensures fallback to known-good firmware
- Code signing: only signed firmware accepted, preventing malicious update injection

> **Pattern driver:** OTA update capability at scale requires a dedicated **OTA Update Service** (Microservices pattern) and edge delivery infrastructure (Edge-Cloud Hybrid). See [02-patterns.md](02-patterns.md).

---

### 2.6 Interoperability

**Definition:** The ability of the system to communicate and exchange meaningful data with heterogeneous devices, protocols, standards, and external systems.

**Why IoT interoperability is uniquely complex:** No web application must simultaneously speak HTTP, MQTT, CoAP, BLE, Zigbee, LoRaWAN, Modbus, OPC-UA, and CAN bus. IoT systems inherit the protocol diversity of the physical world — because different hardware tiers use radically different communication technologies.

**Three levels of interoperability:**

**Protocol interoperability:** Different radio/transport technologies must be bridged at the gateway layer.
- BLE → MQTT (smart home sensors to cloud)
- CoAP → MQTT (constrained devices to standard broker)
- Modbus → MQTT (legacy industrial PLCs to modern IoT platform)
- AMQP → ERP system (IoT events to enterprise resource planning)

**Data model interoperability:** Devices speak different data formats.
- JSON (most IoT APIs)
- CBOR (constrained binary — CoAP native)
- Protocol Buffers / FlatBuffers (high-throughput, schema-enforced)
- Proprietary binary formats (legacy industrial devices)

**Standards interoperability:** Cross-vendor compatibility requires standards adoption.

| Domain | Standard | Purpose |
|---|---|---|
| Smart Home | Matter (over Thread/WiFi) | One device, all ecosystems |
| Industrial | OPC-UA | Factory equipment interoperability |
| Healthcare | HL7 FHIR | Electronic health record integration |
| General IoT | oneM2M | Cross-platform IoT API |
| General IoT | MQTT v5 | Enhanced pub/sub standard |

For the multi-protocol architecture diagram and protocol comparison, see [03-protocols.md](03-protocols.md).

> **Pattern driver:** The interoperability requirement drives the **Event-Driven Architecture** with a central event bus (which decouples protocol-diverse producers from consumers) and the gateway layer in **Edge-Cloud Hybrid** (which performs protocol bridging before events reach the cloud).

---

### 2.7 Observability

**Definition:** The degree to which the internal state of the system can be inferred from its external outputs — enabling operators to understand, diagnose, and predict system behavior without physical access to devices.

**Why IoT observability is harder than web observability:** You cannot SSH into a temperature sensor. You cannot add a print statement to firmware running in the field. If a device behaves incorrectly, you have only the data it chose to transmit to diagnose the problem. Observability must be **designed in from the start** — it cannot be added after deployment.

**The three pillars of observability in IoT:**

| Pillar | IoT-Specific Implementation |
|---|---|
| **Metrics** | Device health (battery level, RSSI, uptime, error count), platform throughput (messages/sec, connection count), business metrics (sensor reading frequency) |
| **Logs** | Edge gateway event logs, application service logs, security audit logs — structured (JSON), correlated by device ID and correlation ID |
| **Traces** | End-to-end path from device publish → broker → ingestion service → storage → dashboard; correlation IDs propagated across MQTT headers and HTTP headers |

**IoT-specific observability patterns:**

- **Heartbeat messages:** Device publishes a health message on a fixed schedule; absence of heartbeat triggers alert
- **Device shadow / twin:** Cloud's authoritative view of each device's last-reported state; enables "what state is device X in right now?"
- **Last Will Testament (LWT):** MQTT broker publishes a preconfigured "device offline" message if connection drops unexpectedly
- **Anomaly detection:** ML-based detection of unusual device behavior patterns (sudden silence, unexpected reading values)

**Tooling:**
- Metrics: Prometheus + Grafana (platform metrics), InfluxDB (time-series device data)
- Logs: ELK stack (Elasticsearch, Logstash, Kibana)
- Traces: Jaeger, Zipkin, AWS X-Ray
- Dashboards: Grafana, Kibana, custom React applications

> **Pattern driver:** High observability requirements favor **Event-Driven Architecture** (every event is already a structured, traceable unit) and **Microservices** (each service emits structured logs and metrics independently). Observability is also an enabler of **Digital Twins** — the twin is the observability interface for a physical asset.

---

### 2.8 Energy Efficiency

**Definition:** The ability of the system — particularly at the device tier — to accomplish its function while consuming minimal electrical power, enabling battery-powered operation over months or years.

**Why energy efficiency has no analog in enterprise software:** No web server runs on two AA batteries. No enterprise API has to choose between "send the data now" and "wait for scheduled transmission window to save battery." Energy efficiency is a fundamental architectural constraint for any IoT system with battery-powered devices.

**Power budget model:** A device's battery life is determined by:

```
Battery Life (hours) = Battery Capacity (mAh) / Average Current Draw (mA)

AA Battery: 2,500 mAh
Active TX (WiFi): 250 mA → 10 hours
Active TX (LoRaWAN): 20 mA → 125 hours
Deep sleep: 0.01 mA → 25,000 hours (2.8 years)
```

**With duty cycling** (100ms TX every 30 seconds):
- Average current ≈ (100ms × 250mA + 29,900ms × 0.01mA) / 30,000ms ≈ 0.84 mA
- Battery life ≈ 2,500 / 0.84 ≈ **2,976 hours (124 days)**

**Protocol energy comparison:**

| Protocol | TX Current | Relative Power |
|---|---|---|
| WiFi (active TX) | 200–250 mA | Baseline |
| BLE (advertising) | 10–15 mA | 15–20× less |
| LoRaWAN (SF7, 14dBm) | 18–28 mA | 8–12× less |
| NB-IoT (TX peak) | 200–220 mA | ~1× |
| NB-IoT (PSM sleep) | 0.003 mA | Ultra-low standby |

Every architecture decision affects energy:
- QoS level: QoS 2 (4 messages per exchange) vs. QoS 0 (1 message) — 4× the radio time
- Payload format: JSON (text, verbose) vs. CBOR (binary, compact) — smaller payload = shorter TX time
- Transmission frequency: every 1 second vs. every 1 minute = 60× battery impact
- Data aggregation at edge: edge can aggregate 60 readings and send 1, extending device battery 60×

**Energy harvesting as an architectural decision:**
Energy harvesting (solar, thermal, vibration, RF) changes the deployment model — devices become "perpetual" and change site selection, enclosure design, and workload scheduling. This is an architectural choice, not just a hardware detail.

**Green IoT / sustainability angle:** At scale, architecture choices have carbon impact. 10 million devices consuming 1mW less each = 10kW of saved power = measurable emissions reduction.

> **Pattern driver:** Energy efficiency constraints are often what forces protocol selection (LoRaWAN over WiFi), architecture pattern selection (Edge-Cloud Hybrid over cloud-only — fewer round-trips per device), and data representation format choices (CBOR/Protobuf over JSON). See [03-protocols.md](03-protocols.md) for connectivity technology comparison.

---

## 3. Characteristics Tradeoffs

In practice, you cannot maximize all characteristics simultaneously. Every architecture is a set of deliberate tradeoffs. The following are the most common tensions in IoT systems:

| Characteristic A | ↔ | Characteristic B | Tension | Resolution Strategy |
|---|---|---|---|---|
| **Security** (TLS/mTLS) | | **Performance** (latency) | TLS adds 50–150ms handshake overhead | Hardware crypto accelerators (ESP32 AES-NI), TLS session resumption, DTLS for UDP |
| **Security** (AES-256 encryption) | | **Energy Efficiency** | Encryption increases CPU cycles and current draw | Use hardware crypto coprocessors (TPM, ATECC608A); pre-shared keys reduce handshake cost |
| **Reliability** (QoS 2) | | **Performance** (throughput) | QoS 2 uses a 4-message handshake per delivery | Use QoS 1 for telemetry (best tradeoff); QoS 2 only for critical commands |
| **Observability** (continuous logging) | | **Energy Efficiency** | Logging requires radio transmission | Batch logs; transmit during scheduled TX windows; prioritize log levels |
| **Scalability** (microservices) | | **Maintainability** (operational complexity) | More services = more deployments, configs, dependencies | Invest in Kubernetes, GitOps, service mesh from the start |
| **Energy Efficiency** (deep sleep) | | **Observability** (real-time monitoring) | A sleeping device cannot send health metrics | Wake-on-interrupt for critical events; periodic health beacons on schedule |
| **Interoperability** (standards/translation) | | **Performance** (latency) | Protocol translation at gateway adds latency | Minimize translation hops; use binary formats end-to-end where possible |
| **Maintainability** (long device lifetime) | | **Security** (rotating credentials) | 10-year devices need certificate lifecycle management | Automated certificate rotation via device management platform; HSM for key storage |

> "Good architecture is not about maximizing all characteristics — it is about making explicit, deliberate tradeoffs aligned with business priorities."

---

## 4. Characteristics Drive Pattern Selection

This matrix is the bridge between your characteristics assessment and the patterns in [02-patterns.md](02-patterns.md). Identify your top 2–3 characteristics and use this table to narrow the viable pattern space.

| Primary Characteristic Priority | Recommended Starting Pattern | Reasoning |
|---|---|---|
| **Latency** < 100ms | Edge-Cloud Hybrid | Cloud cannot be in the critical path; processing must be local |
| **Scalability** > 100K devices | Microservices + EDA | Independent horizontal scaling; high-throughput event brokers |
| **Reliability** (offline autonomy) | Edge-Cloud Hybrid | Local rules engine must operate without cloud connectivity |
| **Energy Efficiency** (battery-powered) | EDA + Low-QoS MQTT | Minimal wake time per event; no polling |
| **Security** in regulated industry | Any pattern + Zero Trust overlay | Security is cross-cutting; adds mTLS, PKI, topic ACL to any pattern |
| **Maintainability** (large fleet, long lifecycle) | Microservices (OTA Service) + Edge | OTA delivery via edge; services independently deployable |
| **Interoperability** (legacy enterprise systems) | EDA + AMQP bridge | Event bus decouples protocol-diverse producers and consumers |
| **Observability** (mission-critical monitoring) | EDA + Microservices | Structured events enable distributed tracing; services emit independent metrics |
| **Cost efficiency** (startup/sporadic workload) | Serverless | Pay-per-invocation; no idle server cost; auto-scaling |

### Multiple Characteristics → Combined Patterns

Real systems require multiple characteristics simultaneously, which drives pattern combinations:

| Characteristics Profile | Pattern Combination |
|---|---|
| High scale + low latency + reliability | Edge-Cloud Hybrid + Microservices + EDA |
| Energy efficiency + security + observability | EDA + Zero Trust + batched telemetry |
| Interoperability + maintainability + scale | Microservices + EDA + AMQP bridges |
| All enterprise characteristics | Edge-Cloud Hybrid + Microservices + EDA + Zero Trust |

The Smart Factory case study (see [05-case-studies.md](05-case-studies.md)) combines all four patterns precisely because it has high requirements across all characteristics simultaneously.

---

## 5. Architecture Characteristics Assessment Worksheet

Use this worksheet at the start of any IoT project to capture your characteristics profile before selecting patterns.

| Characteristic | Your System's Requirement | Priority (1=Low, 5=Critical) | Implied Constraints |
|---|---|---|---|
| **Scalability** | ___ devices, ___ events/sec peak | | |
| **Reliability** | Target uptime: ___%. Offline operation: Yes / No | | |
| **Performance** | Max acceptable latency: ___ ms. Target throughput: ___ events/sec | | |
| **Security** | Regulatory framework: ☐ None ☐ GDPR ☐ HIPAA ☐ Industry-specific | | |
| **Maintainability** | Device lifecycle: ___ years. OTA required: Yes / No | | |
| **Interoperability** | Protocols in use: ___. Legacy systems to integrate: ___ | | |
| **Observability** | Monitoring depth: ☐ Basic ☐ Full metrics ☐ Distributed traces | | |
| **Energy Efficiency** | Power source: ☐ Wired ☐ Battery. Battery target lifespan: ___ | | |

**After completing this worksheet:**
1. Identify your top 3 characteristics by priority score
2. Use the mapping table in Section 4 to identify candidate patterns
3. Validate against the pattern selection matrix in [02-patterns.md](02-patterns.md)
4. Review real-world examples in [05-case-studies.md](05-case-studies.md) whose characteristics profile matches yours

---

## Navigation

| Next Step | Document |
|---|---|
| Pattern selection | [02-patterns.md](02-patterns.md) — matches characteristics to structural patterns |
| Security characteristic deep-dive | [04-security.md](04-security.md) — full threat model and Zero Trust implementation |
| Protocol selection | [03-protocols.md](03-protocols.md) — impacts Performance, Energy Efficiency, and Interoperability characteristics |
| Characteristics in practice | [05-case-studies.md](05-case-studies.md) — see how characteristics drove decisions in Smart Factory, Smart City, and Healthcare |
| Reference architecture | [01-reference-architecture.md](01-reference-architecture.md) — the 4-layer model as a structural response to IoT characteristics |
