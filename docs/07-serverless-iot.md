# Serverless Architecture in IoT — Deep Dive

> A technical deep-dive into FaaS/BaaS patterns for connected device systems —
> when serverless is the right tool, when it fails, and how to operate it at scale.

> **Prerequisite:** This document assumes familiarity with the foundational serverless
> overview in [02-patterns.md § 4](02-patterns.md#4-serverless-architecture). It expands
> that coverage with concrete cost numbers, architecture patterns, cloud provider specifics,
> and operational guidance backed by authoritative sources.

---

## Table of Contents

1. [Why Serverless Fits IoT](#1-why-serverless-fits-iot)
2. [Core Concepts: FaaS and BaaS in IoT](#2-core-concepts-faas-and-baas-in-iot)
3. [Cloud Provider Serverless IoT Stacks](#3-cloud-provider-serverless-iot-stacks)
4. [IoT Serverless Architecture Patterns](#4-iot-serverless-architecture-patterns)
5. [Cold Start Deep Dive](#5-cold-start-deep-dive)
6. [Cost Model Analysis](#6-cost-model-analysis)
7. [Vendor Lock-in Assessment](#7-vendor-lock-in-assessment)
8. [State Management](#8-state-management)
9. [Security Considerations](#9-security-considerations)
10. [Observability](#10-observability)
11. [Real-World Implementations](#11-real-world-implementations)
12. [Decision Framework](#12-decision-framework)
13. [References](#13-references)

---

## 1. Why Serverless Fits IoT

### 1.1 The Structural Alignment: Bursty Traffic Meets Pay-Per-Invocation

Most IoT workloads are **bursty and irregular**, not continuous. A fleet of 10,000 parking sensors fires events sporadically when cars arrive or depart — not every second. A temperature sensor in a cold-chain truck sends an alert once per hour during normal operation, then 50 alerts per minute if the refrigeration fails. A device provisioning service handles 500 new devices at product launch, then near-zero for weeks.

This traffic profile aligns structurally with serverless pricing:

- **Lambda/Functions at zero traffic = zero cost.** An always-on ECS container still costs $30–$80/month regardless of whether it processes any messages.
- **Lambda scales to zero automatically** — no idle spend during quiet periods.
- **Lambda scales to millions** — no capacity planning for peak provisioning windows.

The fundamental reason serverless emerged as a viable IoT pattern is not developer convenience — it is cost efficiency under non-uniform load.

*(Sources: [AWS Lambda — When to use Lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html), [Datadog State of Serverless 2024](https://www.datadoghq.com/state-of-serverless/))*

### 1.2 Three IoT Workload Archetypes

| Workload Archetype | Example | Event Frequency | Per-Event Duration | Serverless Fit |
|---|---|---|---|---|
| **Sporadic alerting** | Temperature threshold breach | Rare (< 1/sec) | < 500ms | ✅ Excellent |
| **Device lifecycle events** | Provisioning, OTA, decommission | Very rare (< 1/min) | 1–15 min | ✅ Excellent (+ Step Functions) |
| **Scheduled aggregation** | Hourly rollup of device metrics | Fixed schedule | 1–5 min | ✅ Good |
| **Moderate telemetry** | 1,000 devices × 1 msg/min | ~17 msg/sec | 50–100ms | ⚠️ Evaluate cost |
| **High-frequency telemetry** | 100K devices × 1 msg/sec | 100,000 msg/sec | 50ms | ❌ Use Kinesis + Flink |
| **Continuous stream processing** | Video analytics, vibration FFT | Continuous | Sustained | ❌ Use containers |

The **break-even rule of thumb**: serverless is cost-competitive when average sustained load stays below ~100 requests/second for a given function. Above that threshold, a single ECS Fargate task at fixed cost becomes cheaper than per-invocation billing.

### 1.3 Why Serverless Structurally Fits IoT Device Events

IoT sensor readings are **naturally stateless messages**: a temperature reading of 23°C at 14:00:00 does not depend on the same compute instance having processed the 13:59:00 reading. The stateless function execution model maps directly to this data characteristic.

In contrast, web application serverless struggles with session state because users expect continuity across requests. IoT events have no such expectation — each event is self-contained.

### 1.4 Where the Structural Mismatch Occurs

Serverless imposes hard limits that conflict with specific IoT requirements:

| Constraint | AWS Lambda | Azure Functions (Consumption) | IoT Impact |
|---|---|---|---|
| **Max execution duration** | 15 minutes | 10 minutes | ❌ Cannot run a continuous Flink stream job |
| **Stateless between invocations** | Always | Always | ❌ In-memory sliding window anomaly detection impossible |
| **Cold start latency** | 100ms–3s | 200ms–4s | ❌ Unacceptable for control commands requiring < 100ms |
| **Default concurrency limit** | 1,000/region | 200 (Consumption) | ❌ Large fleet reconnection storm can hit throttle |
| **Max memory** | 10,240 MB | 1,536 MB (Consumption) | ❌ In-process ML inference on large models |

*(Source: [AWS Lambda quotas](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html), [Azure Functions scale and hosting](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale))*

---

## 2. Core Concepts: FaaS and BaaS in IoT

### 2.1 FaaS: Function as a Service

**FaaS** is the compute layer of serverless: stateless functions triggered by events, billed per invocation and per GB-second of execution time. In IoT, each device message can trigger a function.

**Invocation models:**

- **Asynchronous (event-driven):** IoT Core rule → Lambda. The IoT broker delivers the event and does not wait for Lambda's response. Lambda retries on failure (up to 2 retry attempts by default; configurable Dead Letter Queue for failures).
- **Synchronous:** Device HTTP call → API Gateway → Lambda. The device waits for Lambda's response. Used for provisioning, command acknowledgement, and shadow reads.
- **Polling (event source mapping):** Lambda polls a stream (Kinesis, DynamoDB Streams, SQS) and invokes itself with batches of records. Useful for smoothing telemetry ingestion bursts.

### 2.2 BaaS: Backend as a Service

**BaaS** is the managed service layer that replaces self-operated backends. A full serverless IoT stack composes FaaS + BaaS with no self-managed servers:

```
┌─────────────────────────────────────────────────────────────────┐
│                         BaaS Layer                              │
│  ┌───────────────┐  ┌──────────────────┐  ┌──────────────────┐  │
│  │ IoT Core /    │  │  DynamoDB /      │  │  SNS / Event     │  │
│  │ IoT Hub       │  │  Cosmos DB       │  │  Grid / SES      │  │
│  │ (connectivity)│  │  (state / data)  │  │  (notifications) │  │
│  └──────┬────────┘  └────────┬─────────┘  └──────────────────┘  │
│         │ trigger            │ read/write          ▲             │
├─────────┼────────────────────┼─────────────────────┼─────────────┤
│         │           FaaS Layer                     │             │
│         ▼                                          │             │
│  ┌────────────────────────────────────────────┐    │             │
│  │         Lambda / Azure Function            │────┘             │
│  │  (stateless, event-triggered, ephemeral)   │                  │
│  └────────────────────────────────────────────┘                  │
├─────────────────────────────────────────────────────────────────┤
│                       Device Layer                              │
│  Sensors / Actuators → MQTT → IoT Core / IoT Hub               │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 IoT Trigger Taxonomy

| Trigger Type | AWS | Azure | Invocation Semantics | IoT Use Case |
|---|---|---|---|---|
| **MQTT rule** | IoT Core Rules Engine | IoT Hub message routing | Async, at-least-once | Telemetry processing, alerting |
| **Schedule** | EventBridge Scheduler | Timer trigger | Sync, exactly-once | Hourly aggregation, fleet health checks |
| **HTTP** | API Gateway | HTTP trigger | Sync, requires response | Device provisioning, shadow read/write |
| **Queue** | SQS | Service Bus | Async, at-least-once, ordered per queue | Command dispatch, OTA scheduling |
| **Stream** | Kinesis / DynamoDB Streams | Event Hubs | Polling, batched, ordered per shard | High-volume telemetry ingestion |

### 2.4 Event Source Mapping vs. Direct Invocation

When Lambda is triggered by a stream (Kinesis, DynamoDB Streams, SQS), it uses **event source mapping**: Lambda polls the stream and invokes itself with a batch of records.

Key parameters and their IoT implications:

| Parameter | Default | IoT Impact |
|---|---|---|
| **Batch size** | 100 (Kinesis) | Larger batches = higher throughput, higher latency |
| **Tumbling window** | None | Set to 30s to aggregate telemetry before processing |
| **Bisect on error** | Disabled | Enable to isolate a single malformed device message |
| **Polling interval** | 100–300ms | Adds latency between device publish and Lambda execution |
| **Parallelization factor** | 1 | Up to 10: process multiple batches per shard in parallel |

*(Source: [AWS Lambda event source mapping](https://docs.aws.amazon.com/lambda/latest/dg/invocation-eventsourcemapping.html))*

### 2.5 Concurrency Math for IoT

Understanding how many concurrent Lambda executions your IoT fleet requires is critical to avoiding throttling.

**Formula:** `Concurrency = (Events/second) × (Avg execution duration in seconds)`

**Example calculation — 10,000 devices publishing 1 msg/sec, 50ms Lambda execution:**
```
Concurrency = 10,000 × 0.05 = 500 concurrent executions
```
AWS default limit: 1,000 concurrent per region per account. At 20,000 devices with the same profile, you hit the default limit and throttling begins.

**Mitigation:** Request a limit increase (AWS Support; typically approved to 10,000+), or use reserved concurrency per function to isolate critical alert processing from telemetry ingestion.

*(Source: [AWS Lambda concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html), [CNCF Serverless Whitepaper v2](https://github.com/cncf/wg-serverless/blob/master/whitepapers/serverless-overview/cncf_serverless_whitepaper_v2.pdf))*

---

## 3. Cloud Provider Serverless IoT Stacks

### 3.1 AWS Serverless IoT Stack

The most mature and IoT-specific serverless offering as of 2025:

| Component | Service | Role |
|---|---|---|
| **Device connectivity** | AWS IoT Core | Managed MQTT broker; handles 100M+ connections |
| **Event filtering/routing** | IoT Rules Engine | SQL-based rules route messages to 15+ targets without code |
| **Compute** | AWS Lambda | FaaS; up to 15 min, 10 GB memory |
| **Workflow orchestration** | AWS Step Functions | Multi-step provisioning and OTA rollout state machines |
| **Streaming** | Amazon Kinesis Data Streams | Buffer and smooth high-frequency telemetry |
| **State/data** | Amazon DynamoDB | Serverless NoSQL; on-demand capacity for device state |
| **Notifications** | Amazon SNS | Fan-out alerts to SMS, email, HTTP, Lambda, SQS |
| **Secrets** | AWS Secrets Manager | Encrypted storage for API keys, certificates |
| **Observability** | CloudWatch + X-Ray | Metrics, logs, distributed tracing |

**Key differentiator — IoT Rules Engine:** SQL-like rules that filter and route MQTT messages at the broker level, before invoking Lambda. This single feature eliminates the need for a "fan-out Lambda" that reads all messages and decides routing — the broker does it, at no additional compute cost.

```sql
-- Example: Only invoke Lambda when temperature exceeds 90°C on industrial sensors
SELECT *, topic(2) as deviceId, timestamp() as receivedAt
FROM 'factory/+/sensors/temperature'
WHERE temperature > 90 AND sensorType = 'industrial'
```

*(Source: [AWS IoT Rules Engine SQL reference](https://docs.aws.amazon.com/iot/latest/developerguide/iot-sql-reference.html), [AWS IoT Rules Engine actions](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rule-actions.html))*

### 3.2 Azure Serverless IoT Stack

| Component | Service | Role |
|---|---|---|
| **Device connectivity** | Azure IoT Hub | Managed MQTT/AMQP/HTTP broker; device twin, C2D messaging |
| **Event routing** | IoT Hub message routing | Route D2C messages to Event Hubs, Service Bus, Storage |
| **Compute** | Azure Functions (Consumption) | FaaS; up to 10 min (Consumption), unlimited (Premium/Dedicated) |
| **Workflow orchestration** | Durable Functions | Stateful orchestration; fan-out/fan-in, long-running OTA |
| **Streaming** | Azure Event Hubs | Kafka-compatible; partition-based parallel processing |
| **State/data** | Azure Cosmos DB (serverless) | Multi-model, global distribution; serverless billing mode |
| **Notifications** | Azure Event Grid | Event-based push delivery; schema validation |
| **Secrets** | Azure Key Vault | Managed HSM for certificates and secrets |

**Key differentiator — Durable Functions:** Azure's native orchestration for stateful serverless workflows. IoT device provisioning flows (validate → register → configure → test) map directly to Durable Functions orchestrator pattern without managing state machines externally.

*(Source: [Azure Functions triggers and bindings](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings), [Azure IoT Hub overview](https://learn.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub))*

### 3.3 GCP Serverless IoT Stack

> **Critical note:** Google Cloud IoT Core was **deprecated on August 16, 2023** and shut down on August 16, 2023.
> Projects using Cloud IoT Core must migrate. Recommended paths: (1) HiveMQ Cloud (managed MQTT broker on GCP), (2) EMQX Cloud, (3) AWS IoT Core / Azure IoT Hub for the broker layer with GCP for compute and storage.

| Component | Service | Role |
|---|---|---|
| **Device connectivity** | Pub/Sub (+ third-party MQTT broker) | Message ingestion; not a native MQTT broker |
| **Compute** | Cloud Functions (2nd gen) | FaaS; up to 60 min, 32 GB memory |
| **Workflow orchestration** | Cloud Workflows | Step-based serverless orchestration |
| **Streaming** | Pub/Sub + Dataflow | Pub/Sub for ingestion; Dataflow (managed Beam) for processing |
| **State/data** | Firestore (serverless mode) | Document store; real-time listeners |
| **Notifications** | Pub/Sub push subscriptions | HTTP push to Cloud Functions or external webhooks |

*(Source: [GCP Cloud IoT Core deprecation](https://cloud.google.com/iot-core/docs/release-notes), [GCP Cloud Functions pricing](https://cloud.google.com/functions/pricing))*

### 3.4 Cross-Provider Comparison

| Capability | AWS | Azure | GCP |
|---|---|---|---|
| **FaaS** | Lambda | Functions (Consumption) | Cloud Functions (2nd gen) |
| **Native IoT broker** | IoT Core (MQTT, HTTP, WSS) | IoT Hub (MQTT, AMQP, HTTP) | ❌ Deprecated (Cloud IoT Core) |
| **Workflow orchestration** | Step Functions | Durable Functions | Cloud Workflows |
| **Serverless NoSQL** | DynamoDB (on-demand) | Cosmos DB (serverless) | Firestore |
| **Stream trigger** | Kinesis → Lambda ESM | Event Hubs → Functions | Pub/Sub → Cloud Functions |
| **Free tier (invocations/mo)** | 1M | 1M | 2M |
| **Price per 1M invocations** | $0.20 | $0.20 | $0.40 |
| **Max execution duration** | 15 min | 10 min (Consumption) | 60 min (2nd gen) |
| **Max memory** | 10,240 MB | 1,536 MB (Consumption) | 32,768 MB |
| **Cold start (Node.js, p50)** | ~100ms | ~200ms | ~100ms |
| **Direct MQTT → FaaS trigger** | ✅ IoT Rules Engine | ✅ IoT Hub routing | ⚠️ Via Pub/Sub adapter |
| **Device shadow / twin** | IoT Device Shadow | IoT Hub Device Twin | ❌ Not native |

*(Sources: [AWS IoT Core pricing](https://aws.amazon.com/iot-core/pricing/), [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/), [Azure Functions pricing](https://azure.microsoft.com/en-us/pricing/details/functions/), [GCP Cloud Functions pricing](https://cloud.google.com/functions/pricing))*

---

## 4. IoT Serverless Architecture Patterns

### 4.1 Telemetry Processing Pipeline

**Use case:** 10,000 environmental sensors publish readings every 60 seconds. Each reading must be validated, enriched with device metadata, and persisted to a time-series store for dashboards and historical analysis.

**Why Kinesis in the middle:** Sensors often synchronize their wakeup intervals (all sensors configured for "every 60 seconds" wake simultaneously at :00 of each minute). Without a buffer, 10,000 simultaneous MQTT publishes create a concurrency spike: 10,000 Lambda invocations at once, exhausting the default 1,000 concurrent limit and causing throttling. Kinesis absorbs the spike and delivers records to Lambda in controlled batches.

```
Devices (10K)          AWS Cloud
┌───────────┐   MQTT   ┌────────────────────────────────────────────────────────┐
│ Sensor    │─────────▶│ IoT Core                                                │
│ (every 60s│          │ Rule: SELECT * FROM 'sensors/+/telemetry'               │
└───────────┘          │ Action: PutRecord → Kinesis                             │
                       └──────────────────┬─────────────────────────────────────┘
                                          │ PutRecord
                                          ▼
                          ┌───────────────────────────┐
                          │  Kinesis Data Stream       │
                          │  1 shard (1K records/sec)  │
                          │  Absorbs synchronized spike│
                          └────────────┬──────────────┘
                                       │ Event Source Mapping
                                       │ BatchSize=100, TumblingWindow=30s
                                       ▼
                          ┌────────────────────────────┐
                          │  Lambda: TelemetryIngest    │
                          │  • Validate JSON schema     │
                          │  • Enrich: DynamoDB lookup  │
                          │    (device metadata)        │
                          │  • Write batch to storage   │
                          └──────────┬─────────────────┘
                         ┌───────────┴────────────┐
                         ▼                        ▼
               ┌──────────────────┐   ┌───────────────────────┐
               │  DynamoDB        │   │  S3 (Parquet via       │
               │  (hot: 30 days)  │   │  Kinesis Firehose)    │
               │  Time-series     │   │  (cold: archive)       │
               └──────────────────┘   └───────────────────────┘
```

**Cost at 10K devices × 1 msg/min:**
- IoT Core: 14.4M msgs/day × $1/1M = **$14.40/day**
- Kinesis (1 shard): **$0.015/hr × 24 = $0.36/day**
- Lambda (14.4M invocations × 100ms avg): **~$2.88/day**
- DynamoDB writes (on-demand): **~$18/day**
- S3 storage: **~$0.50/day** (50 GB/day × $0.023/GB)
- **Total: ~$36/day ($1,080/month)**

*(Sources: [AWS Lambda event source mapping with Kinesis](https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis.html), [AWS IoT Core pricing](https://aws.amazon.com/iot-core/pricing/))*

---

### 4.2 Real-Time Alerting Pipeline

**Use case:** Industrial sensors on a factory floor. When temperature exceeds 90°C, an on-call engineer must be notified via SMS and email within 5 seconds. Alert deduplication required — do not re-alert for the same device within 30 minutes.

**Critical optimization — filter at the broker:** The IoT Rules Engine SQL `WHERE` clause filters events **before** invoking Lambda. Lambda is only invoked for threshold-crossing events, not for every temperature reading. If a factory has 500 sensors publishing every 10 seconds (50 events/sec), and alerts occur 0.1% of the time, Lambda is invoked 0.05 times/sec rather than 50 times/sec — a 1,000× cost and concurrency reduction.

```
Sensor publishes temperature=95°C
      │
      ▼
┌─────────────────────────────────────┐
│  IoT Core — Rules Engine             │
│  SQL: SELECT *, topic(2) as deviceId │
│       FROM 'factory/+/temperature'   │
│       WHERE temperature > 90         │◀── Normal readings (< 90°C):
└────────────────┬────────────────────┘    Lambda NOT invoked → zero cost
                 │ Trigger (anomaly only)
                 ▼
┌──────────────────────────────────────┐
│  Lambda: AlertProcessor              │
│                                      │
│  1. Check DynamoDB:                  │
│     GetItem(deviceId, alertType)     │◀── External state: DynamoDB (TTL=1800s)
│                                      │
│  2. If alert exists within 30min:    │
│     → return (deduplicate, no noise) │
│                                      │
│  3. If new alert:                    │
│     → PutItem with TTL=1800s         │
│     → Publish to SNS topic           │
└────────────────┬─────────────────────┘
                 │
                 ▼
┌──────────────────────────────────────┐
│  SNS Topic: iot-factory-alerts       │
├──────────────────────────────────────┤
│  Subscription 1: SMS (via Lambda →   │───▶ On-call engineer phone
│                  Twilio API)         │
│  Subscription 2: SES Email           │───▶ Team distribution list
│  Subscription 3: PagerDuty webhook   │───▶ Incident management system
└──────────────────────────────────────┘
```

**Latency budget analysis:**

| Step | Warm Lambda | Cold Start (p99) |
|---|---|---|
| IoT Core rule processing | ~50ms | ~50ms |
| Lambda cold start | 0ms | ~800ms |
| Lambda execution (DynamoDB + SNS) | ~100ms | ~100ms |
| SNS → SMS delivery | ~200ms | ~200ms |
| **Total** | **~350ms** | **~1,150ms** |

Both are within the 5-second SLA. Cold start p99 of 800ms is acceptable for alerting; it would not be acceptable for real-time control commands.

*(Sources: [AWS IoT Rules Engine SQL](https://docs.aws.amazon.com/iot/latest/developerguide/iot-sql-reference.html), [AWS SNS fanout](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html), [DynamoDB TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html))*

---

### 4.3 Device Provisioning Flow

**Use case:** A device boots for the first time and must register itself, receive its X.509 certificate, and be admitted to its assigned device group. This is a **rare, latency-tolerant event** (once per device lifetime) — an ideal serverless fit. Latency of up to 10 seconds is acceptable.

**Why Step Functions here:** Device provisioning involves multiple sequential steps (validate claim → check quota → create IoT thing → attach policy → issue certificate → register in fleet DB → notify billing system). Orchestrating this in a single Lambda creates a monolithic function with complex error handling. Step Functions provides:
- Built-in retry with exponential backoff for each step
- Compensating transactions on failure (rollback partial registration)
- Visual workflow for debugging failures
- History of every provisioning attempt

```
New Device (factory-issued claim cert)
┌─────────────────────┐
│ Device w/ claim cert │─── HTTPS POST /provision ──────────────────────▶
│ (hardware attestation│                                                  │
│  or manufacturer X.509)                                                 │
└─────────────────────┘                                                  │
                                                                         ▼
                                               ┌────────────────────────────────┐
                                               │  API Gateway                    │
                                               │  + AWS WAF (rate limit:         │
                                               │    100 provisioning req/min)    │
                                               └──────────────┬─────────────────┘
                                                              │
                                                              ▼
                                               ┌─────────────────────────────────────────────┐
                                               │  Step Functions: DeviceProvisioningWorkflow  │
                                               │                                              │
                                               │  State 1: ValidateClaim (Lambda)             │
                                               │  • Verify manufacturer cert chain            │
                                               │  • Check claim cert not previously used      │
                                               │                                              │
                                               │  State 2: CheckQuota (Lambda)                │
                                               │  • Verify customer has device slots          │
                                               │  • Check against billing system API          │
                                               │                                              │
                                               │  State 3: RegisterDevice (Lambda)            │
                                               │  • IoT Core: CreateThing + AttachPolicy      │
                                               │  • Generate unique device certificate        │
                                               │  • DynamoDB: Write device record             │
                                               │                                              │
                                               │  State 4: ActivateDevice (Lambda)            │
                                               │  • Invalidate claim cert (single-use)        │
                                               │  • Notify billing system of activation       │
                                               │  • Return device cert + endpoint to client   │
                                               │                                              │
                                               │  On failure: Compensate (rollback states)    │
                                               └─────────────────────────────────────────────┘
```

**Security requirements for provisioning Lambda:**
- Lambda execution role: `iot:CreateThing`, `iot:AttachPolicy`, `iot:CreateCertificate` only — no wildcard permissions
- Lambda VPC placement: private subnet; NAT Gateway for external API calls
- WAF rules: rate limit per source IP (prevent mass provisioning from compromised factory)
- Claim certificates: invalidated after first use; short TTL (24 hours)

**AWS Fleet Provisioning alternative:** AWS IoT Core's built-in [Fleet Provisioning](https://docs.aws.amazon.com/iot/latest/developerguide/provision-wo-cert.html) handles claim-based registration natively without Lambda. For simple provisioning flows (no external billing system, no custom quota logic), prefer Fleet Provisioning as a pure BaaS solution.

*(Sources: [AWS IoT Fleet Provisioning](https://docs.aws.amazon.com/iot/latest/developerguide/provision-wo-cert.html), [AWS Step Functions error handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html), [AWS WAF rate limiting](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html))*

---

### 4.4 OTA Update Orchestration

**Use case:** Deploy firmware v2.3.1 to 50,000 devices in the field. Requirements: phased rollout (canary → broad), automatic rollback if error rate exceeds 5%, no more than 5% of devices updating simultaneously (bandwidth management).

```
┌──────────────────────────────────────────────────────────────────┐
│  Firmware Release Trigger (CI/CD pipeline or manual)             │
│  → S3: Upload firmware artifact + manifest                       │
└──────────────────────────┬───────────────────────────────────────┘
                           │ S3 event → Lambda
                           ▼
          ┌────────────────────────────────────┐
          │  Lambda: OTAOrchestrator           │
          │  • Sign S3 URL (expires 48h)       │
          │  • Create IoT Jobs with            │
          │    rollout config:                 │
          │    exponentialRate {               │
          │      baseRatePerMinute: 50,        │
          │      incrementFactor: 1.5          │
          │    }                               │
          │  • Start Step Functions execution  │
          └─────────────┬──────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│  Step Functions: OTARolloutMonitor                                  │
│                                                                     │
│  Phase 1: Canary (1% of fleet = 500 devices)                        │
│  ┌─────────────────────────────────────────┐                        │
│  │ Lambda: CheckJobProgress (every 5 min)  │                        │
│  │ • Query IoT Jobs API for completion %   │                        │
│  │ • Query DynamoDB for error reports      │                        │
│  │ If error rate > 5%: → Compensate State  │                        │
│  │ If 95%+ success: → Phase 2             │                        │
│  └─────────────────────────────────────────┘                        │
│                                                                     │
│  Phase 2: Broad rollout (10% → 50% → 100%)                          │
│  ┌─────────────────────────────────────────┐                        │
│  │ Lambda: UpdateJobConfig                 │                        │
│  │ • Increase rollout rate                 │                        │
│  │ • Continue health monitoring            │                        │
│  └─────────────────────────────────────────┘                        │
│                                                                     │
│  Compensate State (on failure):                                     │
│  ┌─────────────────────────────────────────┐                        │
│  │ Lambda: CancelOTAJob                    │                        │
│  │ • Cancel IoT Jobs for remaining devices │                        │
│  │ • Publish rollback alert to SNS         │                        │
│  │ • Write incident record to DynamoDB     │                        │
│  └─────────────────────────────────────────┘                        │
└──────────────────────────┬──────────────────────────────────────────┘
                           │ IoT Jobs API
                           ▼
                  ┌─────────────────┐
                  │  IoT Core Jobs  │
                  │  (50,000 devices│
                  │   in field)     │
                  └────────┬────────┘
                           │ MQTT Job notification
                           ▼
                  ┌─────────────────┐
                  │  Device         │
                  │  • Download from│
                  │    signed S3 URL│
                  │  • Flash + reboot│
                  │  • Report status│
                  └─────────────────┘
```

*(Sources: [AWS IoT Jobs](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html), [AWS Step Functions wait-for-callback pattern](https://docs.aws.amazon.com/step-functions/latest/dg/connect-to-resource.html))*

---

## 5. Cold Start Deep Dive

### 5.1 What Causes Cold Starts

A cold start occurs when no warm execution environment exists for the function. The provider must:
1. Allocate and boot a container (~50–200ms)
2. Initialize the language runtime (~50–500ms depending on runtime)
3. Download and unpack the deployment package (~20–300ms depending on size)
4. Execute the function's initialization code (outside the handler — module imports, SDK clients, DB connections)
5. Execute the handler

Steps 1–4 happen **before** the handler runs and add cold start latency.

### 5.2 Cold Start Duration by Runtime

Data from Datadog's State of Serverless report, p50 and p99 cold start durations on AWS Lambda (128 MB, us-east-1):

| Runtime | p50 Cold Start | p99 Cold Start | Notes |
|---|---|---|---|
| **Node.js 20** | ~80ms | ~400ms | Fastest startup; most IoT Lambda use this |
| **Python 3.12** | ~150ms | ~600ms | Fast; common for data processing |
| **Go 1.x** | ~70ms | ~300ms | Native binary; very fast |
| **Java 21 (JVM)** | ~700ms | ~2,500ms | JVM initialization is expensive |
| **Java 21 (SnapStart)** | ~80ms | ~350ms | SnapStart eliminates most JVM cold start |
| **.NET 8** | ~200ms | ~800ms | Ahead-of-time compilation improves this |

*(Source: [Datadog State of Serverless 2024](https://www.datadoghq.com/state-of-serverless/))*

### 5.3 Mitigation Strategies (Ranked by Effectiveness)

**1. Provisioned Concurrency (most effective — eliminates cold starts)**

Pre-warms a fixed number of execution environments. Those environments remain initialized and ready to execute immediately.

```
Normal Lambda:  Request → [cold start 500ms] → handler execution
Provisioned:    Request → handler execution (0ms cold start)
```

Cost: ~10% more expensive than on-demand Lambda (you pay for idle provisioned environments). Recommended for latency-sensitive paths (control command processing, user-facing provisioning API).

*(Source: [AWS Lambda Provisioned Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html))*

**2. Lambda SnapStart (Java only — near-eliminates cold starts)**

Lambda takes a snapshot of the initialized execution environment and restores it on subsequent cold starts. Reduces Java cold starts from 2–3s to ~100ms.

*(Source: [AWS Lambda SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html))*

**3. ARM64 / Graviton2 Architecture (20–34% faster, 20% cheaper)**

Lambda functions deployed on arm64 architecture (AWS Graviton2) consistently show lower cold start duration and lower execution cost than x86_64 for equivalent workloads.

*(Source: [AWS Lambda arm64 architecture](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html))*

**4. Minimize deployment package size**

Lambda downloads and unpacks the deployment package on cold start. Smaller packages = faster cold starts.
- Node.js: avoid bundling entire `node_modules`; use tree-shaking (esbuild, webpack)
- Python: use Lambda layers for large dependencies; avoid bundling unused libraries
- Target: < 5 MB unzipped for sub-100ms cold start contribution from package loading

**5. Move heavy initialization outside the handler**

SDK clients, database connections, and configuration loading that happen inside the handler run on every invocation. Move them to module-level (outside the handler) — they execute once during initialization, then reuse across warm invocations.

```javascript
// ❌ Initialized on every invocation (bad)
export const handler = async (event) => {
  const ddb = new DynamoDBClient({});  // new client every call
  ...
};

// ✅ Initialized once, reused across warm invocations (good)
const ddb = new DynamoDBClient({});
export const handler = async (event) => {
  ...
};
```

**6. Keep-warm pings (NOT recommended for IoT)**

Scheduling an EventBridge rule to invoke the function every 5 minutes prevents cold starts — but also prevents scale-to-zero cost savings, the primary reason to use serverless. For IoT, where traffic is naturally sporadic, keep-warm eliminates the cost advantage. Use Provisioned Concurrency instead for latency-critical paths.

### 5.4 IoT-Specific Cold Start Tolerance

| IoT Function | Max Acceptable Latency | Cold Start Tolerance | Recommendation |
|---|---|---|---|
| **Threshold alerting** | 5s SLA | ✅ Tolerate (p99 < 1s) | Default Lambda |
| **Device provisioning** | 10s SLA | ✅ Tolerate | Default Lambda |
| **OTA orchestration** | Minutes | ✅ Irrelevant | Default Lambda |
| **Control commands** (turn off pump) | < 200ms | ❌ Unacceptable | Provisioned Concurrency or move off serverless |
| **User-facing device API** | < 500ms | ⚠️ Marginal | Provisioned Concurrency |

---

## 6. Cost Model Analysis

### 6.1 Pricing Components (AWS, as of 2024)

| Service | Metric | Price |
|---|---|---|
| **AWS Lambda** | Per 1M requests | $0.20 |
| **AWS Lambda** | Per GB-second | $0.0000166667 |
| **AWS IoT Core** | Per million messages (≤ 5KB) | $1.00 |
| **AWS IoT Core** | Per million connection-minutes | $0.08 |
| **Amazon Kinesis** | Per shard-hour | $0.015 |
| **Amazon DynamoDB** | Per million write request units | $1.25 |
| **Amazon DynamoDB** | Per million read request units | $0.25 |
| **Amazon SNS** | Per million notifications (email/SQS) | $0.50 |
| **AWS Step Functions** | Per 1,000 state transitions | $0.025 |

*(Sources: [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/), [AWS IoT Core pricing](https://aws.amazon.com/iot-core/pricing/), [AWS DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/))*

### 6.2 Three Fleet Scale Scenarios

#### Scenario A: 1,000 Devices × 1 msg/min (Serverless wins clearly)

| Component | Daily Volume | Daily Cost |
|---|---|---|
| IoT Core messages | 1.44M | $1.44 |
| Lambda invocations (1.44M × 100ms, 128MB) | 1.44M | $0.31 |
| DynamoDB writes (on-demand) | 1.44M | $1.80 |
| **Serverless total** | | **~$3.55/day (~$107/month)** |
| **ECS Fargate equivalent** (0.25 vCPU, 0.5 GB) | Fixed | **$7.30/day ($220/month)** |

**Verdict:** Serverless is 2× cheaper at this scale.

#### Scenario B: 100,000 Devices × 1 msg/min (Evaluate carefully)

| Component | Daily Volume | Daily Cost |
|---|---|---|
| IoT Core messages | 144M | $144 |
| Kinesis (2 shards needed) | — | $0.72 |
| Lambda invocations (144M × 100ms, 256MB) | 144M | $62.30 |
| DynamoDB writes | 144M | $180 |
| **Serverless total** | | **~$387/day (~$11,600/month)** |
| **ECS Fargate** (4 tasks, 2 vCPU, 4GB each) | Fixed | **~$140/day ($4,200/month)** |

**Verdict:** Containers are 2.75× cheaper at sustained load. DynamoDB on-demand pricing becomes the dominant cost — consider DynamoDB provisioned capacity or switching to a time-series database (Amazon Timestream, InfluxDB on EC2).

#### Scenario C: 1,000,000 Devices × 1 msg/min (Serverless is cost-prohibitive)

Estimated serverless cost: **~$3,870/day ($116,000/month)** — dominated by IoT Core messaging and DynamoDB writes. At this scale, purpose-built streaming infrastructure (Kafka + Flink, Kinesis + EMR) running on reserved EC2 reduces total cost to **$8,000–$15,000/month**.

### 6.3 Break-Even Summary

| Fleet Size | Recommended Pattern | Estimated Monthly Cost |
|---|---|---|
| < 5,000 devices | Serverless (Lambda + DynamoDB on-demand) | < $500 |
| 5,000–50,000 devices | Serverless (Lambda + DynamoDB provisioned) | $500–$5,000 |
| 50,000–500,000 devices | Hybrid: Serverless for alerts + Kinesis + containers for ingestion | $5,000–$30,000 |
| > 500,000 devices | Streaming-first (Kafka/Kinesis + Flink/Spark) + serverless only for alerts | > $30,000 |

### 6.4 Hidden Costs to Account For

- **NAT Gateway:** If Lambda runs in a VPC (recommended for security), outbound traffic to AWS services routes through NAT Gateway at $0.045/GB. A function that writes 1 KB to DynamoDB per invocation at 10M invocations/day = 10 GB/day = $0.45/day in NAT costs. At scale, this is significant.
- **X-Ray tracing:** $5.00 per million traces sampled. At 100M invocations/day with 1% sampling = 1M traces/day = $5/day.
- **CloudWatch Logs:** $0.50/GB ingested. Verbose logging at scale adds up; use structured logging and set appropriate log levels.
- **IoT Core connection minutes:** A device that maintains a persistent MQTT connection 24/7 costs $0.08/million connection-minutes = $0.115/device/month at continuous connection.

---

## 7. Vendor Lock-in Assessment

### 7.1 Lock-in Surface in Serverless IoT

| Layer | Lock-in Component | Portability |
|---|---|---|
| **Trigger wiring** | IoT Rules Engine SQL (AWS), IoT Hub routing (Azure) | ❌ Provider-specific; rewrite required |
| **Function code** | Node.js / Python handler code | ✅ Portable across providers and on-premises |
| **Orchestration** | Step Functions (AWS), Durable Functions (Azure) | ❌ Provider-specific APIs; moderate rewrite |
| **IAM / auth** | AWS IAM policies, Azure RBAC | ❌ Provider-specific |
| **Observability** | CloudWatch, X-Ray, Application Insights | ⚠️ Abstraction possible with OpenTelemetry |
| **Storage bindings** | DynamoDB SDK, Cosmos DB SDK | ❌ Rewrite requires; data migration also needed |

**The honest assessment:** For IoT projects that already depend on a managed IoT broker (IoT Core / IoT Hub), the serverless function layer adds marginal lock-in on top of existing broker lock-in. Accepting serverless lock-in is a reasonable trade-off when the IoT broker is already a dependency.

### 7.2 Abstraction Strategies

| Tool | What It Abstracts | What It Does NOT Abstract |
|---|---|---|
| **Serverless Framework** | Deployment config, packaging | Trigger types, IAM, SDK calls in function code |
| **AWS SAM** | CloudFormation for Lambda/API GW | AWS-only; no cross-provider portability |
| **Terraform** | Infrastructure provisioning | Function code, SDK calls |
| **OpenTelemetry** | Observability instrumentation | Compute execution model, triggers |

**Practical recommendation:** Use Terraform for infrastructure (portable definitions) and keep function code provider-SDK-free by abstracting storage behind a thin repository interface. This limits rewrite scope to the repository implementation if migrating providers.

*(Source: [CNCF Serverless Whitepaper v2](https://github.com/cncf/wg-serverless/blob/master/whitepapers/serverless-overview/cncf_serverless_whitepaper_v2.pdf))*

---

## 8. State Management

### 8.1 The Fundamental Problem

Serverless functions are stateless — every invocation starts with a clean execution environment (warm or cold). IoT systems inherently need state: "Has this device already been alerted in the last 30 minutes?", "What is the current firmware version of device X?", "Is device Y currently in a maintenance window?"

The solution is to externalize all state to a managed store and fetch it at the start of each invocation.

### 8.2 Three External State Patterns

#### Pattern A: DynamoDB for Device State (Most Common)

Best for: device shadow data, alert deduplication, provisioning records, configuration state.

```
Lambda receives event
  │
  ├── GetItem(deviceId) → device state (< 5ms, DynamoDB Global Tables for low latency)
  │
  ├── Process event with current state
  │
  └── PutItem / UpdateItem (conditional write for idempotency)
```

Use DynamoDB on-demand for sporadic access patterns; use DynamoDB provisioned capacity when access is predictable (reduces cost 60–70% at steady load).

#### Pattern B: ElastiCache Redis for Sub-Millisecond State Lookups

Best for: rate limiting, real-time counters, session tokens, hot device metadata.

```
Lambda (in VPC) → ElastiCache Redis (1–2ms) → cached device config
                                             (DynamoDB miss-path: ~5ms)
```

Cost: ElastiCache Redis `cache.t3.micro` = ~$12/month. Worthwhile when DynamoDB read latency (3–10ms) is too slow for the use case.

#### Pattern C: Step Functions for Workflow State

Best for: multi-step provisioning, OTA rollout phases, long-running approval workflows.

Step Functions stores workflow state durably between Lambda invocations, including retry counts, input/output of each state, and workflow history. Functions become pure business logic — no state serialization/deserialization code required.

### 8.3 Idempotency: Critical for IoT

IoT Core delivers messages **at-least-once** — a device message may be delivered to Lambda more than once under failure conditions. Lambda functions must be idempotent (same input produces same output, with no harmful side effects from duplicate execution).

**Implementation pattern — DynamoDB conditional write as idempotency token:**

```javascript
// Use eventId (from IoT Core) as idempotency key
const params = {
  TableName: 'IoTEvents',
  Item: { eventId: event.messageId, deviceId: event.deviceId, processedAt: Date.now() },
  ConditionExpression: 'attribute_not_exists(eventId)'  // Fail if duplicate
};

try {
  await ddb.put(params).promise();
  // Process event — only reaches here on first delivery
} catch (e) {
  if (e.code === 'ConditionalCheckFailedException') {
    return; // Duplicate event, skip processing
  }
  throw e;
}
```

*(Sources: [AWS Lambda idempotency](https://docs.aws.amazon.com/lambda/latest/dg/invocation-idempotency.html), [AWS Lambda Powertools idempotency](https://docs.powertools.aws.dev/lambda/python/latest/utilities/idempotency/))*

### 8.4 Anti-Pattern: Lambda Calling Lambda Synchronously

```
❌ DO NOT DO THIS:
Lambda A → (sync invoke) → Lambda B → (sync invoke) → Lambda C
```

Problems: cascading latency (total = sum of all cold starts), hidden coupling (A cannot succeed without B and C), concurrency amplification (1 outer invocation consumes 3 Lambda concurrency slots), error propagation complexity.

**Use instead:** Step Functions for orchestration (states, not functions, coordinate the flow), or SQS queues for loose coupling between Lambda functions.

---

## 9. Security Considerations

### 9.1 IAM Least-Privilege for Lambda

Lambda execution roles must follow least-privilege. A common mistake in IoT projects is granting `iot:*` or `dynamodb:*` to Lambda execution roles — this grants excessive access.

**Correct approach — scope to exact resources:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["iot:DescribeThing", "iot:UpdateThingShadow"],
      "Resource": "arn:aws:iot:us-east-1:123456789:thing/factory-*"
    },
    {
      "Effect": "Allow",
      "Action": ["dynamodb:GetItem", "dynamodb:PutItem"],
      "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/DeviceState"
    }
  ]
}
```

*(Source: [AWS Lambda security best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html#function-configuration))*

### 9.2 VPC Placement

Lambda functions that access private resources (RDS, ElastiCache, private APIs) must run in a VPC. For IoT functions that only access AWS-managed services (DynamoDB, SNS, IoT Core), VPC placement adds latency (NAT Gateway for outbound) without security benefit — run them outside VPC.

**When to use VPC for IoT Lambda:**
- Lambda → private ElastiCache Redis (no public endpoint)
- Lambda → private time-series database (InfluxDB on EC2)
- Lambda → on-premises systems via AWS Direct Connect / VPN
- Lambda → third-party APIs that require fixed IP whitelisting (use NAT Gateway EIP)

### 9.3 Secrets Management

Never store credentials, API keys, or certificates in Lambda environment variables (they appear in CloudWatch logs and Lambda configuration).

**Correct pattern:**
```javascript
// ✅ Fetch from Secrets Manager at cold start, cache in module scope
const secretsClient = new SecretsManagerClient({});
let cachedSecret;

const getSecret = async (secretName) => {
  if (!cachedSecret) {
    const response = await secretsClient.send(new GetSecretValueCommand({ SecretId: secretName }));
    cachedSecret = JSON.parse(response.SecretString);
  }
  return cachedSecret;
};
```

Cache the secret at module level (fetched once per warm Lambda instance, not per invocation). Use Secrets Manager rotation for automatic credential rotation without redeploying Lambda.

*(Source: [AWS Secrets Manager with Lambda](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_lambda.html))*

### 9.4 IoT-Specific Security: Mutual TLS

All major IoT platforms require or strongly recommend mutual TLS (mTLS) for device-to-cloud connections:
- **AWS IoT Core:** Requires X.509 client certificates for MQTT connections. Supports certificate rotation via MQTT Last Will and Testament + Lambda automation.
- **Azure IoT Hub:** Supports X.509 CA-signed certificates and SAS tokens. Certificates recommended for production.
- **Device certificate lifecycle:** Use IoT Core Jobs or IoT Hub Direct Methods to trigger certificate rotation on devices before expiry. Lambda can automate issuance of new certificates and delivery to devices.

*(Source: [AWS IoT Core security and identity](https://docs.aws.amazon.com/iot/latest/developerguide/iot-security-identity.html), [AWS IoT Serverless Well-Architected Lens](https://docs.aws.amazon.com/wellarchitected/latest/iot-lens/welcome.html))*

### 9.5 Dead Letter Queues for Failed Events

Configure a DLQ (SQS queue) for asynchronous Lambda invocations. If Lambda fails (exception, timeout, OOM) after all retries, the event goes to the DLQ rather than being silently dropped. IoT events represent real device state — dropping them silently creates data gaps.

```
IoT Core → Lambda → [failure after 2 retries] → SQS DLQ
                                                     │
                                                     └── Alert: DLQ depth > 0
                                                          → Manual inspection
                                                          → Replay after fix
```

---

## 10. Observability

### 10.1 Three Pillars in Serverless IoT Context

**Metrics (CloudWatch Lambda Insights)**
- Invocation count, duration (p50/p99), error rate, throttle count
- Memory utilization, CPU utilization (Lambda Insights extension)
- Custom metrics: alert deduplication hit rate, provisioning success rate

**Traces (AWS X-Ray)**
- Traces follow a request across: IoT Core → Lambda → DynamoDB → SNS
- X-Ray service map visualizes the full event processing path
- Latency breakdown by segment: function initialization + handler execution + downstream service calls

**Logs (CloudWatch Logs — Structured JSON)**
```javascript
// Use structured logging; avoid console.log strings
console.log(JSON.stringify({
  level: 'INFO',
  event: 'alert_processed',
  deviceId: event.deviceId,
  temperature: event.temperature,
  alertSent: true,
  latencyMs: Date.now() - startTime
}));
```

Use [AWS Lambda Powertools](https://docs.powertools.aws.dev/lambda/) (available for Python, TypeScript, Java) for structured logging, tracing, and metrics with minimal boilerplate.

### 10.2 Cold Start Monitoring

Lambda logs include `Init Duration` field for cold start invocations. Parse this in CloudWatch Logs Insights to track cold start frequency and duration:

```
fields @timestamp, @duration, @initDuration, @message
| filter ispresent(@initDuration)
| stats avg(@initDuration) as avgColdStart,
        pct(@initDuration, 99) as p99ColdStart,
        count(*) as coldStartCount by bin(1h)
```

Alert if `p99(@initDuration) > 1000ms` for latency-sensitive functions.

### 10.3 Key Alerts for IoT Serverless Functions

| Alert | Threshold | Meaning |
|---|---|---|
| **Error rate** | > 0.1% of invocations | Function exceptions — check DLQ |
| **Throttle count** | > 0 | Hitting concurrency limit; request increase |
| **Duration p99** | > 80% of timeout | Risk of timeout under load; optimize or increase memory |
| **DLQ depth** | > 0 | Failed events accumulating; investigate and replay |
| **Cold start p99** | > 1,000ms | Consider Provisioned Concurrency for this function |
| **IoT Rules Engine error rate** | > 0 | Rule SQL syntax error or downstream target unreachable |

*(Sources: [AWS Lambda Powertools](https://docs.powertools.aws.dev/lambda/), [AWS X-Ray developer guide](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html), [AWS CloudWatch Lambda Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Lambda-Insights.html))*

---

## 11. Real-World Implementations

### 11.1 AWS IoT Greengrass + Lambda at the Edge

AWS IoT Greengrass enables the same Lambda function code to run **on edge devices** (industrial gateways, Raspberry Pi, AWS Outposts) as well as in the cloud. A temperature alerting Lambda developed and tested in the cloud can be deployed to factory-floor gateways to provide < 10ms local processing without cloud round-trips.

This is a unique capability: **serverless function model extended to the edge**. The same IAM, deployment, and monitoring model applies at edge and cloud.

Key constraint: Greengrass Lambda runs on edge hardware with real resource limits. A 10 GB Lambda that works in the cloud cannot run on a gateway with 512 MB RAM. Keep edge Lambda functions small (< 50 MB) and purpose-specific.

*(Source: [AWS IoT Greengrass — Run Lambda functions](https://docs.aws.amazon.com/greengrass/v2/developerguide/run-lambda-functions.html))*

### 11.2 Smart Building: Threshold Alerting at Scale

**Context:** Smart building management systems (BMS) — 50,000+ sensors (HVAC, occupancy, energy meters, access control) per large commercial building campus.

**Serverless fit:** HVAC sensors publish every 30 seconds; alerts trigger when thresholds are crossed (< 0.1% of messages). IoT Core Rules Engine SQL filters at the broker — Lambda is invoked for < 100 alert events/hour across a 50,000-sensor building. Lambda cost: near zero. An always-on server would cost $100+/month for this alerting function.

The pattern used by multiple building automation vendors (documented in AWS IoT partner case studies) is precisely the alerting pipeline from § 4.2: IoT Core rule with WHERE clause → Lambda → SNS → facility management system.

*(Source: [AWS IoT for Smart Buildings reference architecture](https://aws.amazon.com/iot/solutions/smart-buildings/), [AWS Well-Architected IoT Lens](https://docs.aws.amazon.com/wellarchitected/latest/iot-lens/welcome.html))*

### 11.3 Sporadic Workloads: Parking and Asset Tracking

**Context:** City-scale parking sensor networks. Sensors detect car presence and report only on state change (car arrives / car departs) — not continuously.

**Serverless alignment:** A parking space changes state 3–10 times per day on average. For 10,000 spaces, that is 30,000–100,000 events per day — sustained rate of < 2 events/second. Lambda at this scale costs < $0.02/day in invocations. An always-on container for the same throughput costs $30–$50/month.

This "report on change" pattern (vs. "report periodically") is the IoT traffic profile most aligned with serverless economics.

### 11.4 Observed Pattern Across All Real Implementations

After reviewing public AWS case studies, Azure IoT customer stories, and engineering blog posts:

> **All documented serverless IoT production deployments use serverless for notification pipelines, event-triggered enrichment, and device lifecycle management — not for primary telemetry ingestion at scale.**

The telemetry ingestion path (high-frequency, high-volume, sustained) consistently uses streaming infrastructure (Kinesis, Kafka, Event Hubs). Serverless handles the **edges** of the data pipeline: filtering, alerting, and lifecycle management.

---

## 12. Decision Framework

### 12.1 Two-Question Quick Filter

```
Question 1: Is sustained event frequency > 10 events/second?
  ├── YES → ❌ Do NOT use serverless as primary ingestion path.
  │          Use Kinesis/Kafka + stream processor. Serverless for alerts only.
  └── NO  → Continue to Question 2.

Question 2: Does per-event processing require more than 15 minutes?
  ├── YES → ❌ Exceeds Lambda max duration.
  │          Use ECS/Fargate, EC2, or batch processing.
  └── NO  → ✅ Serverless is a candidate. Evaluate cost and cold start requirements.
```

### 12.2 Scenario Decision Table

| Scenario | Device Count | Event Rate | Serverless? | Recommended Pattern |
|---|---|---|---|---|
| **POC / MVP alerting** | < 1,000 | Sporadic | ✅ Yes | Lambda + IoT Core direct rule |
| **Production alerting** | Any | < 1/sec per fleet | ✅ Yes | Lambda + DynamoDB deduplication |
| **Device provisioning** | Any | < 10/min | ✅ Yes | Step Functions + Lambda |
| **OTA orchestration** | Any | < 1/hr | ✅ Yes | Step Functions + IoT Jobs |
| **Telemetry ingestion** | < 5,000 | 1 msg/min | ✅ Yes | Lambda ESM + Kinesis (1 shard) |
| **Telemetry ingestion** | 50,000+ | 1 msg/min | ❌ No | Kinesis + Flink or Kinesis Firehose |
| **Control commands** | Any | Any | ❌ No (cold start) | Containers or Provisioned Concurrency |
| **ML inference per event** | Any | > 1/sec | ❌ No (cost) | SageMaker endpoints or containers |

### 12.3 Evolution Path

```
Stage 1: Startup / MVP (< 1K devices)
  → Lambda + IoT Core + DynamoDB (on-demand)
  → Serverless everything: alerting, ingestion, provisioning
  → Monthly cost: < $200

Stage 2: Growth (1K–20K devices)
  → Lambda for alerting + provisioning (serverless stays)
  → Kinesis + Lambda ESM for telemetry (stream processing)
  → DynamoDB provisioned capacity (reduces cost)
  → Monthly cost: $500–$3,000

Stage 3: Scale (20K–500K devices)
  → Lambda for alerting + lifecycle ONLY
  → Kinesis + Flink (EMR or Kinesis Data Analytics) for telemetry
  → DynamoDB provisioned or Amazon Timestream for time-series
  → Monthly cost: $5,000–$50,000

Stage 4: Enterprise (500K+ devices)
  → Serverless retained only for alerting pipelines (sporadic by nature)
  → All ingestion: MSK (Managed Kafka) + Flink
  → Edge-Cloud Hybrid for latency-sensitive paths
  → Monthly cost: $50,000+
```

**Migration trigger:** Migrate telemetry ingestion from serverless to streaming when **monthly Lambda + DynamoDB bill for ingestion exceeds $1,500** — at that point, a Kinesis + Flink solution on reserved EC2 is demonstrably cheaper.

---

## 13. References

### AWS Documentation

- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)
- [AWS Lambda — Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
- [AWS Lambda — Event source mapping (Kinesis)](https://docs.aws.amazon.com/lambda/latest/dg/with-kinesis.html)
- [AWS Lambda — Provisioned Concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html)
- [AWS Lambda — SnapStart](https://docs.aws.amazon.com/lambda/latest/dg/snapstart.html)
- [AWS Lambda — Security best practices](https://docs.aws.amazon.com/lambda/latest/dg/best-practices.html)
- [AWS Lambda — Idempotency](https://docs.aws.amazon.com/lambda/latest/dg/invocation-idempotency.html)
- [AWS Lambda — ARM64 / Graviton2](https://docs.aws.amazon.com/lambda/latest/dg/foundation-arch.html)
- [AWS Lambda — Quotas and limits](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html)
- [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/)
- [AWS IoT Core — Lambda rule action](https://docs.aws.amazon.com/iot/latest/developerguide/iot-lambda-rule.html)
- [AWS IoT Core — Rules Engine SQL reference](https://docs.aws.amazon.com/iot/latest/developerguide/iot-sql-reference.html)
- [AWS IoT Core — Rules Engine actions](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rule-actions.html)
- [AWS IoT Core — Fleet Provisioning](https://docs.aws.amazon.com/iot/latest/developerguide/provision-wo-cert.html)
- [AWS IoT Core — Security and identity](https://docs.aws.amazon.com/iot/latest/developerguide/iot-security-identity.html)
- [AWS IoT Core — Jobs (OTA)](https://docs.aws.amazon.com/iot/latest/developerguide/iot-jobs.html)
- [AWS IoT Core Pricing](https://aws.amazon.com/iot-core/pricing/)
- [AWS IoT Greengrass — Run Lambda functions](https://docs.aws.amazon.com/greengrass/v2/developerguide/run-lambda-functions.html)
- [AWS Step Functions — Error handling](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html)
- [AWS Well-Architected — Serverless Applications Lens](https://docs.aws.amazon.com/wellarchitected/latest/serverless-applications-lens/welcome.html)
- [AWS Well-Architected — IoT Lens](https://docs.aws.amazon.com/wellarchitected/latest/iot-lens/welcome.html)
- [AWS Secrets Manager with Lambda](https://docs.aws.amazon.com/secretsmanager/latest/userguide/retrieving-secrets_lambda.html)
- [AWS DynamoDB — TTL](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [AWS DynamoDB Pricing](https://aws.amazon.com/dynamodb/pricing/)
- [AWS CloudWatch Lambda Insights](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Lambda-Insights.html)
- [AWS X-Ray Developer Guide](https://docs.aws.amazon.com/xray/latest/devguide/aws-xray.html)
- [AWS SNS — Common scenarios (fan-out)](https://docs.aws.amazon.com/sns/latest/dg/sns-common-scenarios.html)
- [AWS WAF — Rate-based rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rule-statement-type-rate-based.html)
- [AWS IoT for Smart Buildings](https://aws.amazon.com/iot/solutions/smart-buildings/)

### Microsoft Azure Documentation

- [Azure Functions — Scale and hosting](https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale)
- [Azure Functions — Triggers and bindings overview](https://learn.microsoft.com/en-us/azure/azure-functions/functions-triggers-bindings)
- [Azure Functions Pricing](https://azure.microsoft.com/en-us/pricing/details/functions/)
- [Azure IoT Hub — Concepts and IoT Hub](https://learn.microsoft.com/en-us/azure/iot-hub/iot-concepts-and-iot-hub)

### Google Cloud Documentation

- [GCP Cloud IoT Core — Release notes (deprecation notice)](https://cloud.google.com/iot-core/docs/release-notes)
- [GCP Cloud Functions Pricing](https://cloud.google.com/functions/pricing)

### Industry Reports and Standards

- [Datadog State of Serverless 2024](https://www.datadoghq.com/state-of-serverless/) — cold start measurements, invocation patterns, runtime comparison
- [CNCF Serverless Whitepaper v2.0](https://github.com/cncf/wg-serverless/blob/master/whitepapers/serverless-overview/cncf_serverless_whitepaper_v2.pdf) — vendor-neutral serverless concepts, FaaS/BaaS taxonomy, vendor lock-in analysis

### Developer Tools

- [AWS Lambda Powertools](https://docs.powertools.aws.dev/lambda/) — structured logging, tracing, idempotency utilities for Lambda

---

> *Document compiled for the "Software Architecture for IoT" seminar — March 2026.*
> *See also: [02-patterns.md](02-patterns.md) for the foundational 5-pattern overview, and [06-architecture-characteristics.md](06-architecture-characteristics.md) for the 8 IoT architecture characteristics framework.*
