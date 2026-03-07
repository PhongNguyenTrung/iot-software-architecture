# Edge-Cloud Hybrid Architecture trong IoT — Phân tích Chuyên sâu

> **Phong cách trình bày:** Tài liệu này trình bày Edge-Cloud Hybrid Architecture theo cấu trúc của cuốn
> *Fundamentals of Software Architecture* (Richards & Ford, O'Reilly 2020): topology → cơ chế hoạt động
> → đánh giá architecture characteristics → các biến thể pattern → khi nào dùng / khi nào không dùng
> → trade-offs → framework quyết định.

> **Tài liệu liên quan:**
> - [02-patterns.md §5](02-patterns.md#5-edge-cloud-hybrid-architecture) — tổng quan ngắn gọn về Edge-Cloud Hybrid
> - [06-architecture-characteristics.md](06-architecture-characteristics.md) — framework đánh giá đặc tính
> - [08-serverless-deep-dive.md](08-serverless-deep-dive.md) — deep dive Serverless IoT (report cùng series)

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Topology](#2-topology)
3. [Cơ chế Hoạt động](#3-cơ-chế-hoạt-động)
4. [Đánh giá Architecture Characteristics](#4-đánh-giá-architecture-characteristics)
5. [Các Biến thể Kiến trúc](#5-các-biến-thể-kiến-trúc)
6. [Edge Hardware và Runtime Platform](#6-edge-hardware-và-runtime-platform)
7. [Phân tích Trade-offs Sâu](#7-phân-tích-trade-offs-sâu)
8. [Bảo mật](#8-bảo-mật)
9. [Observability](#9-observability)
10. [Triển khai Thực tế](#10-triển-khai-thực-tế)
11. [Khi nào Dùng / Khi nào Không Dùng](#11-khi-nào-dùng--khi-nào-không-dùng)
12. [Framework Quyết định](#12-framework-quyết-định)
13. [Tài liệu Tham khảo](#13-tài-liệu-tham-khảo)

---

## 1. Tổng quan

### 1.1 Định nghĩa

**Edge-Cloud Hybrid Architecture** là một mô hình kiến trúc IoT trong đó công việc xử lý được **phân tán có chủ đích** giữa hai tầng: tầng **edge** (nằm gần thiết bị vật lý — gateway, industrial PC, hoặc edge server tại hiện trường) và tầng **cloud** (trung tâm dữ liệu hoặc public cloud). Không có tầng nào là "chính" hay "phụ" — mỗi tầng xử lý loại công việc mà nó phù hợp nhất.

Sự khác biệt cơ bản so với kiến trúc cloud-only:

| Tiêu chí | Cloud-Only | Edge-Cloud Hybrid |
|---|---|---|
| **Nơi quyết định** | Luôn ở cloud | Edge cho thời gian thực; cloud cho fleet-level |
| **Khi mất kết nối** | Hệ thống ngừng hoạt động | Edge tiếp tục tự trị |
| **Latency** | 50–200ms (WAN round-trip) | < 1ms on-device; < 10ms edge gateway |
| **Bandwidth** | 100% data lên cloud | 1–5% data lên cloud (lọc tại edge) |
| **Data sovereignty** | Data ra khỏi facility | Data có thể ở lại hoàn toàn tại site |

Edge không phải là thiết bị IoT (sensor, actuator) — đó là **lớp xử lý trung gian** được triển khai tại hiện trường. Một edge node có thể là Raspberry Pi $50, một NVIDIA Jetson $1,500, hay một edge server Xeon $10,000 — tùy thuộc vào yêu cầu tính toán.

### 1.2 Lịch sử và Bối cảnh

| Năm | Sự kiện |
|---|---|
| **1960s–1990s** | **SCADA/DCS** — hệ thống điều khiển công nghiệp đã luôn có "local intelligence" tại hiện trường; tiền thân của edge computing |
| **2000s** | **CDN (Content Delivery Network)** — distributed caching tại "edge" của internet; khái niệm edge đầu tiên trong web |
| **2012** | Bonomi et al. (Cisco) công bố whitepaper **"Fog Computing"** — chính thức đặt tên cho mô hình xử lý phân tán gần thiết bị IoT |
| **2014** | **OpenFog Consortium** thành lập (Cisco, ARM, Dell, Intel, Microsoft); IEEE Fog Computing specification bắt đầu |
| **2016** | **AWS Greengrass v1** ra mắt — Lambda functions chạy trực tiếp trên Raspberry Pi và thiết bị IoT |
| **2017** | **Azure IoT Edge** ra mắt — Docker containers triển khai đến edge nodes |
| **2018** | **NVIDIA Jetson Xavier** — AI inference chip cho edge; bắt đầu kỷ nguyên "AI at the Edge" |
| **2019** | **K3s** (CNCF) — Kubernetes lightweight cho edge; Kubernetes chính thức đến hiện trường |
| **2020** | **5G commercial rollout** — ultra-low latency network kích hoạt edge use cases mới; MEC (Multi-access Edge Computing) |
| **2022** | **NVIDIA Jetson Orin** — 275 TOPS AI performance; factory AI inference trở thành mainstream |
| **2023** | **AWS IoT Greengrass v2 + SageMaker Edge** — ML model lifecycle management tại edge được tích hợp |
| **2025** | Edge-Cloud Hybrid là **dominant pattern cho enterprise IoT** — Gartner: 75% enterprise IoT sẽ dùng edge processing vào 2025 |

*(Nguồn: [Bonomi et al. 2012](https://dl.acm.org/doi/10.1145/2342509.2342513), [Gartner IoT Edge Report 2023](https://www.gartner.com/en/information-technology/insights/internet-of-things))*

### 1.3 Vị trí trong Hệ sinh thái Kiến trúc IoT

Edge-Cloud Hybrid chiếm góc **phức tạp nhất** trong bản đồ 5 pattern IoT — nhưng cũng là pattern có khả năng đáp ứng **nhiều đặc tính quan trọng nhất**:

```
                        Độ phức tạp vận hành
         Thấp ◄────────────────────────────────► Cao
          │                                        │
Ít        │  Layered      Serverless               │
thiết bị  │  Monolith                              │
          │                                        │
          │              Microservices              │
          │                 + EDA                  │
Nhiều     │                                Edge-Cloud
thiết bị  │                                Hybrid  ◄── Bạn đang ở đây
          └────────────────────────────────────────┘

Offline resilience?  Chỉ Edge-Cloud Hybrid có.
Latency < 10ms?      Chỉ Edge-Cloud Hybrid đảm bảo.
Bandwidth efficiency? Chỉ Edge-Cloud Hybrid xử lý tại source.
```

### 1.4 Tại sao Edge-Cloud Hybrid là Dominant Pattern cho Enterprise IoT

Bốn áp lực kỹ thuật hội tụ buộc các tổ chức lớn phải dùng hybrid:

**1. Latency không thể giải quyết bằng tối ưu cloud:**
Tốc độ ánh sáng trong cáp quang: ~200,000 km/giây. Từ Berlin đến Frankfurt (200km) và trở lại = tối thiểu 2ms overhead vật lý. Với processing overhead thực tế, một cloud round-trip từ factory floor tới trung tâm dữ liệu gần nhất thường là **50–200ms** — không thể dưới 10ms dù tối ưu đến đâu.

**2. Reliability của WAN:**
Hầu hết công nghiệp không chấp nhận "fail when internet is down". Ngay cả 99.9% uptime WAN = **8.7 giờ downtime/năm** — không thể chấp nhận cho safety systems hay continuous production.

**3. Bandwidth economics:**
Một nhà máy 500 camera HD gửi ~1 Gbps raw video lên cloud = **~$3,000–$10,000/tháng** bandwidth cost. Xử lý tại edge, chỉ gửi kết quả (anomaly flags, metadata) = **95% tiết kiệm**.

**4. Regulatory và data sovereignty:**
GDPR (EU), HIPAA (US Healthcare), Industrial security standards (IEC 62443) yêu cầu dữ liệu nhạy cảm không rời khỏi cơ sở vật chất. Edge processing là cơ chế kỹ thuật đảm bảo điều này.

---

## 2. Topology

### 2.1 3-Tier Topology Cơ bản

Edge-Cloud Hybrid là kiến trúc **3 tầng không đối xứng**: mỗi tầng có vai trò, latency, và khả năng khác nhau.

```
┌────────────────────────────────────────────────────────────────────┐
│                          CLOUD TIER                                │
│                                                                    │
│  ┌──────────────┐  ┌───────────────┐  ┌──────────┐  ┌──────────┐  │
│  │  ML Training │  │  Long-term    │  │  Fleet   │  │ Dashboard│  │
│  │  & Retraining│  │  Storage      │  │  Mgmt &  │  │ & APIs   │  │
│  │  (batch)     │  │  (S3, ADLS)   │  │  OTA Hub │  │          │  │
│  └──────────────┘  └───────────────┘  └──────────┘  └──────────┘  │
│                                                                    │
│  Latency: 50–200ms WAN   |   Scale: elastic   |   Cost: OpEx      │
└────────────────────────────┬───────────────────────────────────────┘
                             │  Sync channel (aggregated data,
                             │  ML model updates, commands,
                             │  configuration)
                             │  Protocols: MQTT/TLS, HTTPS, AMQP
┌────────────────────────────┴───────────────────────────────────────┐
│                           EDGE TIER                                │
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Rules Engine│  │  Local       │  │ AI/ML    │  │ Protocol │  │
│  │  (real-time  │  │  Time-Series │  │ Inference│  │ Bridge   │  │
│  │   decisions) │  │  Storage     │  │ Engine   │  │ (MQTT,   │  │
│  │              │  │  (buffer)    │  │          │  │  OPC UA, │  │
│  └──────────────┘  └──────────────┘  └──────────┘  │  Modbus) │  │
│                                                     └──────────┘  │
│  Latency: 1–50ms local   |   Scale: fixed CapEx   |   Always-on  │
└────────────────────────────┬───────────────────────────────────────┘
                             │  Local protocols
                             │  (MQTT, OPC UA, Modbus, BACnet,
                             │   CAN bus, Zigbee, BLE)
              ┌──────────────┼──────────────────┐
              │              │                  │
         ┌────┴─────┐  ┌─────┴────┐  ┌─────────┴──┐
         │ Sensors  │  │ Cameras  │  │ Actuators   │
         │(temp, pres│  │(vision,  │  │(valves,     │
         │  vib...) │  │ thermal) │  │  motors...) │
         └──────────┘  └──────────┘  └─────────────┘
                         DEVICE TIER
         Latency: < 1ms hardwired   |   Physical world interface
```

### 2.2 Edge Node Internal Architecture

Bên trong một edge node là một hệ thống đầy đủ chứa nhiều thành phần phối hợp:

```
┌────────────────────────────────────────────────────────────────┐
│                        EDGE NODE                               │
│                                                                │
│  ┌────────────────────┐    ┌───────────────────────────────┐  │
│  │  PROTOCOL BRIDGE   │    │    CLOUD SYNC AGENT           │  │
│  │                    │    │                               │  │
│  │  MQTT ↔ CoAP       │    │  - TLS mutual auth to cloud   │  │
│  │  OPC UA ↔ REST     │    │  - Store-and-forward buffer   │  │
│  │  Modbus ↔ JSON     │    │  - Conflict detection         │  │
│  │  BACnet ↔ MQTT     │    │  - Delta sync (send only diff)│  │
│  └────────┬───────────┘    └──────────────┬────────────────┘  │
│           │ normalized events             │ bidirectional      │
│           ▼                               ▼                    │
│  ┌────────────────────────────────────────────────────────┐   │
│  │              LOCAL EVENT BUS / MESSAGE BROKER          │   │
│  │              (Mosquitto, NATS, or in-process)          │   │
│  └────────┬───────────────────────────────────────────────┘   │
│           │                                                    │
│   ┌───────┼──────────────────────────────────────┐            │
│   │       │                                      │            │
│   ▼       ▼                                      ▼            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐    │
│  │  RULES ENGINE│  │  AI INFERENCE│  │  LOCAL STORAGE   │    │
│  │              │  │  ENGINE      │  │                  │    │
│  │  CEP rules   │  │  ONNX Runtime│  │  Time-series DB  │    │
│  │  Threshold   │  │  TensorRT    │  │  (InfluxDB,      │    │
│  │  checks      │  │  OpenVINO    │  │   SQLite,        │    │
│  │  Anomaly     │  │              │  │   TimescaleDB)   │    │
│  │  logic       │  │  Model store │  │                  │    │
│  └──────┬───────┘  └──────────────┘  │  Retention: 7–30d│    │
│         │ commands                   └──────────────────┘    │
│         ▼                                                     │
│  ┌──────────────────────────────────────────────────────┐    │
│  │           ACTUATOR / OUTPUT INTERFACE                │    │
│  │  Direct control: relay, valve, motor driver          │    │
│  │  Latency: < 1ms for hardwired; < 10ms for network   │    │
│  └──────────────────────────────────────────────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

### 2.3 Full Topology — Data Flow, Command Flow, Sync Flow

Topology đầy đủ thể hiện **ba luồng dữ liệu độc lập** vận hành đồng thời:

```
  DEVICES                EDGE TIER                     CLOUD TIER
  ┌──────────┐
  │ Sensor   │──raw──▶┌─────────────────┐
  │ 1..N     │        │  Protocol Bridge│
  └──────────┘        │  + Rules Engine │──────── [1] TELEMETRY FLOW ──────────▶
                      │                 │  Aggregated/filtered data (1–5% of raw)
  ┌──────────┐        │  [local action] │  Protocol: MQTT over TLS / HTTPS       │
  │ Actuator │◀──cmd──│  < 10ms latency │                                        │
  └──────────┘        │                 │◀─────── [2] COMMAND FLOW ──────────────┤
                      │                 │  Fleet commands, config updates         │
  ┌──────────┐        │  AI Inference   │  Protocol: MQTT C2D / AMQP             │
  │ Camera   │──raw──▶│  Engine         │                                        │
  │ (video)  │        │  [30fps local]  │◀─────── [3] MODEL UPDATE FLOW ─────────┘
  └──────────┘        │                 │  ML model packages (signed)
                      │  Local Storage  │  OTA software updates
                      │  [7–30 day buf] │  Configuration changes
                      └────────────────┘

  ─────────── ONLINE MODE: all 3 flows active ───────────────────────────────────

  ┌──────────┐         ┌─────────────────┐          ╔══════════════╗
  │ Sensor   │──raw──▶ │  Rules Engine   │          ║ CLOUD        ║
  │ (all)    │         │  [autonomous]   │──X──── ✗ ║ UNREACHABLE  ║
  └──────────┘         │                 │          ╚══════════════╝
                       │  Local Storage  │
  ┌──────────┐         │  [buffering all]│
  │ Actuator │◀──cmd── │  AI Inference   │
  │ (still   │  local  │  [still running]│
  │  works)  │  rules  │                 │
  └──────────┘         └─────────────────┘

  ─────────── OFFLINE MODE: edge operates autonomously ──────────────────────────

                       ┌─────────────────┐         ┌──────────────┐
                       │  Cloud Sync     │──flush──▶│ Cloud        │
                       │  Agent          │  buffer  │ (reconnected)│
                       │  [replay buffer]│◀─────────│              │
                       └─────────────────┘ commands └──────────────┘

  ─────────── RECONNECT MODE: buffer flush + state reconciliation ────────────────
```

### 2.4 Các Thành phần Cốt lõi

| Thành phần | Vai trò | Ví dụ triển khai |
|---|---|---|
| **Protocol Bridge** | Chuyển đổi giữa device protocols (Modbus, OPC UA, BACnet) và cloud protocols (MQTT, HTTPS) | Kepware, Azure IoT Edge OPC Publisher, Node-RED |
| **Local Rules Engine** | Đánh giá điều kiện real-time; ra lệnh cho actuators không qua cloud | Drools, Node-RED, AWS Lambda Greengrass |
| **AI Inference Engine** | Chạy ML models cục bộ; không cần cloud round-trip | ONNX Runtime, TensorRT, OpenVINO, TFLite |
| **Local Time-Series Storage** | Buffer telemetry khi offline; local query; retention 7–30 ngày | InfluxDB, TimescaleDB, SQLite, OSIsoft PI |
| **Store-and-Forward Buffer** | Hàng đợi bền vững cho cloud sync; đảm bảo at-least-once delivery | Mosquitto persistence, NATS JetStream, local Kafka |
| **Cloud Sync Agent** | Quản lý kết nối cloud; delta sync; conflict resolution | AWS Greengrass nucleus, Azure IoT Edge runtime |
| **OTA Agent** | Nhận và áp dụng software/model updates từ cloud | Mender, Balena, AWS Greengrass OTA |

---

## 3. Cơ chế Hoạt động

### 3.1 Luồng Xử lý Dữ liệu — Từ Device đến Cloud

```
  Device                Edge Node                   Cloud
    │                       │                          │
    │── raw data ──────────▶│                          │
    │   (1,000 msg/sec)     │                          │
    │                       │[1] Parse & normalize     │
    │                       │[2] Apply threshold rules │
    │                       │[3] Aggregate (30s window)│
    │                       │[4] AI inference          │
    │                       │[5] Write local storage   │
    │                       │                          │
    │                       │── aggregated event ─────▶│
    │                       │   (1–5% of raw volume)   │
    │                       │                          │
    │◀── actuator command ──│  (if anomaly detected)   │
    │   (< 10ms local)      │                          │
    │                       │── alert metadata ───────▶│
                                                        │
                                               [Fleet analytics]
                                               [ML model training]
                                               [Dashboard update]
```

**Tỷ lệ lọc dữ liệu điển hình:**

| Loại deployment | Raw data tại device | Sau edge filtering | Ratio |
|---|---|---|---|
| Smart factory (500 sensors) | 500 msg/sec | 5–25 msg/sec | 1–5% |
| Video analytics (50 cameras) | 50 × 30fps = 1,500 frames/sec | Metadata + anomaly thumbnails | < 0.1% |
| Oil rig (200 sensors) | 200 msg/sec | Hourly aggregates + alerts | < 1% |
| Smart city traffic (1,000 sensors) | 1,000 msg/sec | Aggregated flow metrics | 2–10% |

### 3.2 Local Decision Making — Ba Mức Latency

Edge-Cloud Hybrid không phải một latency target — đây là **ba tầng quyết định** với latency khác nhau:

```
  LATENCY TIER 1: On-Device (< 1ms)
  ─────────────────────────────────
  Sensor → Firmware logic → Actuator
  Ví dụ: Emergency stop khi áp suất > 200 PSI
  Không qua network; pure hardware/firmware interlock
  Không thể fail: safety-critical, IEC 61511 compliance

  LATENCY TIER 2: Edge Gateway (1–50ms)
  ──────────────────────────────────────
  Device → Edge node → Local actuator
  Ví dụ: HVAC adjustment khi nhiệt độ > 28°C
  Network hop trong local LAN/industrial Ethernet
  Có thể fail gracefully: fallback về tier 1 hoặc safe state

  LATENCY TIER 3: Edge Server/Fog (50–200ms)
  ─────────────────────────────────────────────
  Device → Edge node → Edge server → Response
  Ví dụ: Computer vision defect detection trên production line
  AI inference trên edge GPU; kết quả trong 50–200ms
  Latency do model inference, không phải network

  LATENCY TIER 4: Cloud (200ms – nhiều giây)
  ──────────────────────────────────────────
  Device → Edge → Cloud → Response
  Ví dụ: Fleet-wide anomaly correlation, long-term trending
  WAN round-trip + cloud processing
  Chỉ dùng cho non-time-critical decisions
```

### 3.3 Store-and-Forward — Cơ chế Offline Resilience

Store-and-forward là cơ chế cho phép edge **hoạt động tự trị khi mất kết nối cloud** và **đảm bảo không mất dữ liệu** khi reconnect:

```
  ONLINE STATE:
  Device data → Edge buffer (RAM/disk) → Cloud sync → ACK → Purge buffer

  OFFLINE STATE (cloud unreachable):
  Device data → Edge buffer (disk) → [held, not purged]
                                       buffer grows…
                                       up to retention limit (7–30 days)

  RECONNECT:
  Cloud sync agent detects connectivity restored
  → Start flushing buffer (oldest-first)
  → Rate-limited: không flood cloud (backpressure)
  → Deduplicate against cloud state (idempotency keys)
  → Purge successfully ACK'd messages
  → Cloud state converges with edge state
```

**Giới hạn của Store-and-Forward:**

| Thông số | Typical | Notes |
|---|---|---|
| **Buffer size** | 7–30 ngày data | Phụ thuộc local storage capacity |
| **Replay rate** | 2–5× real-time | Không replay quá nhanh để tránh cloud overload |
| **Ordering** | FIFO per device | Cross-device ordering không đảm bảo |
| **Conflict window** | Thời gian offline | Nếu cloud và edge update cùng state → cần conflict resolution |

### 3.4 Năm Synchronization Patterns

Mỗi pattern phù hợp với một loại data và consistency requirement khác nhau:

**Pattern A — Store-and-Forward (Durable Queue)**
```
Edge → [durable queue] → Cloud (when connected)
Semantics: at-least-once delivery; idempotent consumer required
Dùng cho: telemetry, sensor readings, events (append-only)
Không dùng cho: commands (idempotency risk), mutable state
```

**Pattern B — Eventual Consistency (Periodic Sync)**
```
Edge state → [periodic delta sync, every 5min/1hour] → Cloud state
Cloud state → [periodic pull or push] → Edge state
Semantics: eventual consistency; conflicts possible during offline periods
Dùng cho: device configuration, non-critical telemetry aggregates
```

**Pattern C — Edge-First (Edge Owns Truth)**
```
Edge makes decisions autonomously
Cloud receives decisions as facts (not proposals)
Cloud learns from edge, does not override
Dùng cho: safety systems, autonomous robots, vehicles
```

**Pattern D — Cloud-First (Cloud Owns Truth)**
```
Cloud publishes desired state (twin/shadow)
Edge reads desired state, applies it, reports actual state back
Dùng cho: configuration management, firmware update policies, fleet commands
```

**Pattern E — Bi-directional Sync (CRDT/OT)**
```
Both edge and cloud can modify state concurrently
Merge algorithm ensures convergence (no data loss)
Dùng cho: shared state that both tiers write (e.g., alarm acknowledgements)
Complexity cao nhất; dùng khi thực sự cần thiết
```

### 3.5 OTA Model Update Delivery

Đưa ML model mới đến hàng nghìn edge nodes là một trong những vận hành phức tạp nhất:

```
  Cloud (ML Training Pipeline)
       │
       │[1] Train new model (new data, retraining schedule)
       │[2] Validate accuracy on holdout set (must exceed threshold)
       │[3] Package model artifact (ONNX/TensorRT/OpenVINO format)
       │[4] Sign package (cryptographic signature)
       │[5] Upload to model registry
       │
       ▼
  OTA Rollout Orchestrator
       │
       │[6] Staged rollout:
       │    Canary → 1% sites (24h observation)
       │    Early  → 10% sites (48h observation)
       │    Main   → 50% sites (72h observation)
       │    Full   → 100% sites
       │
       │[7] Health checks after each stage:
       │    - Model inference accuracy on validation subset
       │    - Edge CPU/memory impact
       │    - No increase in error rates
       │
       │[8] Auto-rollback if health check fails
       │
       ▼
  Edge Nodes (1,000+ sites)
       │
       │[9] OTA agent downloads signed package
       │[10] Verify signature before installation
       │[11] Hot-swap model (zero-downtime model switching)
       │[12] Report health status to cloud
```

### 3.6 Command Flow — Cloud to Device (C2D)

Lệnh từ cloud xuống device là **synchronous path ngược chiều** với telemetry:

```
  Cloud                     Edge Node              Device
    │                           │                     │
    │── command ───────────────▶│                     │
    │   {deviceId, action,      │                     │
    │    params, timeout,       │                     │
    │    correlationId}         │                     │
    │                           │[validate command]   │
    │                           │[check device state] │
    │                           │── cmd ─────────────▶│
    │                           │   (local delivery)  │
    │                           │◀── ACK ─────────────│
    │◀── delivery ACK ──────────│                     │
    │   {correlationId,         │                     │
    │    status: delivered}     │                     │
    │                           │                     │
    │ [timeout check]           │                     │
    │◀── execution result ──────│◀── result ──────────│
    │   {correlationId,         │                     │
    │    status: success/fail}  │                     │
```

**Timeout và retry:**
- Command timeout: thường 30s–5min (tùy action)
- Nếu edge offline: command được queue tại cloud; delivered khi edge reconnect
- Idempotency: mỗi command có unique `correlationId`; edge không thực thi lại nếu đã nhận

---

## 4. Đánh giá Architecture Characteristics

> **Phương pháp đánh giá:** Theo framework của Richards & Ford trong *Fundamentals of Software Architecture*,
> mỗi architecture style được đánh giá trên các dimensions chính bằng thang ★☆☆☆☆ (1–5 sao).
> Edge-Cloud Hybrid có **profile đặc trưng nhất** trong 5 pattern IoT: xuất sắc ở các đặc tính
> vật lý (performance, reliability, offline) nhưng phức tạp nhất về vận hành.

### 4.1 Bảng Đánh giá Tổng hợp

| Architecture Characteristic | Rating | Giải thích cho IoT Context |
|---|---|---|
| **Scalability** | ★★★★☆ | Cloud tier scale elastic tốt. Edge tier scale theo số sites — mỗi site mới là CapEx investment. Không thể "add a new region in 5 minutes" như cloud-only. |
| **Elasticity** | ★★★☆☆ | Edge hardware là fixed capacity — không tự scale. Cloud tier elastic hoàn toàn. Khi một edge site cần thêm compute, phải mua/deploy hardware mới. |
| **Fault Tolerance** | ★★★★★ | **Đặc tính mạnh nhất.** Edge tiếp tục vận hành hoàn toàn khi cloud unreachable. Ngay cả khi WAN down 48 giờ, factory vẫn chạy, data được buffer. Không pattern nào đạt được đặc tính này. |
| **Performance** | ★★★★★ | **Đặc tính mạnh thứ hai.** < 1ms hardwired on-device; < 10ms edge local network; không phụ thuộc WAN latency. Đây là lý do duy nhất nhiều use case bắt buộc phải dùng edge. |
| **Cost Efficiency** | ★★★☆☆ | CapEx edge hardware cao ($50–$10,000/site). Tiết kiệm bandwidth đáng kể (95%). Break-even phụ thuộc số sites và bandwidth volume. Không phải lựa chọn rẻ nhất để khởi đầu. |
| **Security** | ★★★★☆ | Network segmentation xuất sắc — devices không bao giờ expose trực tiếp ra internet. Rủi ro chính: **physical security** của edge hardware tại site. |
| **Observability** | ★★★☆☆ | Multi-tier tracing phức tạp. Offline periods tạo gaps trong logs và metrics. Cần giải pháp observability riêng cho edge fleet (không chỉ cloud metrics). |
| **Maintainability** | ★★☆☆☆ | **Điểm yếu lớn nhất.** Synchronization bugs rất subtle và nguy hiểm. Deployment đến edge sites phức tạp. Cần expertise cả embedded systems, distributed systems, và cloud. |
| **Reliability** | ★★★★★ | Combination of: (1) edge offline resilience, (2) local fallback, (3) cloud redundancy. Toàn bộ hệ thống không có single point of failure. |
| **Deployability** | ★★☆☆☆ | Physical deployment đến mỗi site (không thể "push a button" cho hardware). OTA software updates phức tạp (10,000 edge nodes × staged rollout). |
| **Offline Resilience** | ★★★★★ | **Đặc tính độc quyền** — không pattern nào trong 5 patterns IoT có. Hệ thống tiếp tục hoạt động đầy đủ (rules, AI inference, local control) khi hoàn toàn mất kết nối cloud. |
| **Bandwidth Efficiency** | ★★★★★ | **Đặc tính độc quyền thứ hai.** Xử lý tại source; chỉ gửi 1–5% dữ liệu raw lên cloud. Với video/image workloads: tiết kiệm 95–99% bandwidth. |
| **Energy Efficiency** | ★★★☆☆ | Edge hardware luôn bật — không scale-to-zero như serverless. Tuy nhiên: bằng cách không gửi raw data lên cloud, tiết kiệm đáng kể energy của network transmission và cloud compute. |

### 4.2 Phân tích Chi tiết theo Dimension Quan trọng nhất

#### Fault Tolerance & Offline Resilience ★★★★★

Đây là lý do **không thể thay thế** của Edge-Cloud Hybrid. Khi so sánh với 4 patterns còn lại:

```
Pattern             | Cloud down 1 giờ | Cloud down 24 giờ
--------------------|------------------|--------------------
Layered Monolith    | Hệ thống ngừng   | Hệ thống ngừng
Microservices       | Hệ thống ngừng   | Hệ thống ngừng
Event-Driven (EDA)  | Hệ thống ngừng   | Hệ thống ngừng
Serverless          | Hệ thống ngừng   | Hệ thống ngừng
Edge-Cloud Hybrid   | Tiếp tục 100%    | Tiếp tục 100%*
```

*\*Giả sử edge buffer chưa đầy; local storage đủ capacity*

**Ví dụ thực tế:** AWS region outage tháng 12/2021 khiến Roomba robots không connect được cloud (iRobot dùng pure cloud). Các factory IIoT với edge processing tiếp tục vận hành bình thường trong suốt outage.

#### Performance ★★★★★

Không thể đạt latency < 10ms với cloud-only, bất kể cloud provider:

```
Latency budget cho safety interlock (ví dụ: emergency stop):
- Sensor detect anomaly:              ~1ms (hardware)
- Edge local decision:                ~1–5ms (software)
- Actuator activation:                ~1ms (hardware)
TOTAL (edge path):                    ~3–7ms ✅

Với cloud round-trip (Berlin → Frankfurt):
- Network to nearest PoP:             ~2ms
- Cloud processing:                   ~10ms
- Network return:                     ~2ms
TOTAL (cloud path):                   ~14–50ms ❌ (too slow)
```

#### Maintainability ★★☆☆☆

Đây là cost thực sự của Edge-Cloud Hybrid. Các failure modes đặc trưng:

- **Split-brain**: Edge và cloud có conflicting state sau offline period — cần merge strategy
- **Cascading sync failures**: Một edge node sync failure block subsequent syncs
- **OTA gone wrong**: Bad model update đến 10% sites trước khi phát hiện — staged rollout bắt buộc
- **Physical security**: Edge node tại site bị tamper, extract cloud credentials

---

## 5. Các Biến thể Kiến trúc

Không có một "standard" Edge-Cloud Hybrid — có **4 biến thể chính** tùy theo compute power, budget, và use case.

---

### Pattern 1: Lightweight Gateway

**Mô tả:** Edge node là thiết bị nhỏ, giá thấp (Raspberry Pi, industrial PC đơn giản). Chủ yếu làm protocol bridge, thresholding đơn giản, và store-and-forward. Không có AI inference on-device.

**Topology:**

```
  Devices (< 50/site)
  ┌────────────┐           ┌────────────────────────────────────────┐
  │ Temp/Humid │──MQTT────▶│         Lightweight Edge Node          │
  │ Sensors    │           │         (Raspberry Pi 4 / Atom PC)     │
  └────────────┘           │                                        │
                           │  [Mosquitto MQTT broker]               │
  ┌────────────┐           │  [Node-RED rules engine]               │──▶ Cloud
  │ Door/Window│──Zigbee──▶│  [SQLite local storage]                │    (MQTT/TLS)
  │ Sensors    │           │  [Python threshold scripts]            │
  └────────────┘           │                                        │
                           │  Latency: 10–100ms                     │
  ┌────────────┐           │  Cost: $50–$200/site                   │
  │ Energy     │──Modbus──▶│  Team requirement: 1–2 engineers       │
  │ Meters     │           └────────────────────────────────────────┘
  └────────────┘
```

**Stack điển hình:**

| Layer | Technology |
|---|---|
| OS | Raspberry Pi OS / Ubuntu Server |
| Message broker | Eclipse Mosquitto |
| Rules engine | Node-RED, Home Assistant |
| Local DB | SQLite, InfluxDB Lite |
| Cloud connector | AWS Greengrass Lite, Azure IoT Edge core |
| Language | Python scripts |

**Trade-off Analysis:**

| Dimension | Đánh giá | Chi tiết |
|---|---|---|
| **Cost** | ✅ Thấp | $50–$200/site hardware; low power consumption |
| **Complexity** | ✅ Thấp | Node-RED visual programming; không cần embedded expertise |
| **AI/ML** | ❌ Không có | Compute không đủ cho inference; cần cloud |
| **Latency** | ⚠️ 10–100ms | Đủ cho HVAC, access control; không đủ cho safety systems |
| **Throughput** | ⚠️ Giới hạn | < 1,000 messages/giây; bị giới hạn bởi Pi CPU |
| **Resilience** | ✅ Tốt | Store-and-forward hoạt động tốt; 7-day buffer |

**Khi dùng Pattern này:**
- Smart building, retail monitoring, agriculture sensors
- < 50 devices/site; latency 10–100ms OK
- Budget/site rất hạn chế; không cần AI inference

---

### Pattern 2: Intelligent Edge (AI at the Edge)

**Mô tả:** Edge node có GPU/NPU chuyên dụng để chạy ML models cục bộ. Inference xảy ra tại site — không có cloud round-trip cho AI decisions. Model updates được push từ cloud theo lịch.

**Topology:**

```
  Devices (cameras, sensors)
  ┌────────────┐           ┌──────────────────────────────────────────┐
  │ HD Camera  │──GigE────▶│         Intelligent Edge Node            │
  │ (1080p,    │           │         (NVIDIA Jetson AGX Orin)         │
  │  30fps)    │           │                                          │
  └────────────┘           │  ┌────────────────────────────────────┐  │
                           │  │  AI Inference Pipeline             │  │
  ┌────────────┐           │  │                                    │  │
  │ Vibration  │──EtherCAT▶│  │  Camera frames                    │  │
  │ Sensor     │           │  │      → Preprocessing (GPU)         │  │──▶ Cloud
  │ (FFT data) │           │  │      → YOLOv8 Detection           │  │   (anomaly
  └────────────┘           │  │      → Classification             │  │    metadata
                           │  │      → Alert if defect            │  │    + thumbs)
  ┌────────────┐           │  │  275 TOPS; 30fps real-time        │  │
  │ Thermal    │──USB──────▶│  └────────────────────────────────────┘  │
  │ Camera     │           │                                          │
  └────────────┘           │  Latency AI: 5–50ms (inference only)     │
                           │  Cost: $600–$2,500/site                  │
                           └──────────────────────────────────────────┘
```

**Stack điển hình:**

| Layer | Technology |
|---|---|
| Hardware | NVIDIA Jetson AGX Orin, Intel Alder Lake + Movidius |
| AI framework | TensorRT, ONNX Runtime, OpenVINO, TFLite |
| Container runtime | Docker / containerd |
| Model serving | Triton Inference Server (NVIDIA) |
| Edge runtime | AWS Greengrass v2, Azure IoT Edge |
| Cloud ML pipeline | SageMaker, Azure ML, Vertex AI |

**Trade-off Analysis:**

| Dimension | Đánh giá | Chi tiết |
|---|---|---|
| **AI Inference** | ✅ Xuất sắc | 30fps real-time; 5–50ms latency; no cloud round-trip |
| **Bandwidth** | ✅ Rất tốt | Chỉ gửi anomaly thumbnails + metadata; 99% bandwidth saved |
| **Cost** | ⚠️ Trung bình | $600–$2,500/unit; giảm dần theo scale |
| **Model lifecycle** | ⚠️ Phức tạp | Cloud train → stage → push → validate trên mỗi device |
| **Power** | ⚠️ 10–65W | Always-on GPU; cần power infrastructure |
| **Latency (inference)** | ✅ Tốt | 5–50ms; đủ cho production line quality control |

**Khi dùng Pattern này:**
- Factory defect detection qua computer vision
- Medical imaging at point of care (portable ultrasound, radiology)
- Autonomous equipment guidance (AGV, drone inspection)
- Cần AI inference < 100ms; bandwidth cost prohibitive cho raw video

---

### Pattern 3: Fog Computing (Edge Cluster)

**Mô tả:** Thay vì một edge node đơn lẻ, triển khai **cluster nhỏ** (3–10 nodes) tại site. Kubernetes lightweight (K3s) orchestrate containers. Cung cấp high availability và horizontal scaling tại edge.

**Topology:**

```
  Site (large hospital / smart city district / large campus)
  ┌───────────────────────────────────────────────────────────────┐
  │                    EDGE CLUSTER (K3s)                         │
  │                                                               │
  │  ┌───────────┐  ┌───────────┐  ┌───────────┐                 │
  │  │ Edge Node │  │ Edge Node │  │ Edge Node │  (3–10 nodes)   │
  │  │ (Master)  │  │ (Worker)  │  │ (Worker)  │                 │
  │  └─────┬─────┘  └─────┬─────┘  └─────┬─────┘                │
  │        └──────────────┴──────────────┘                       │
  │                         │ K3s cluster networking              │
  │  ┌──────────────────────▼──────────────────────────────────┐  │
  │  │              K3s Workloads                              │  │
  │  │  [InfluxDB cluster]  [Kafka edge]  [ML inference pods]  │  │──▶ Cloud
  │  │  [Rules engine]      [API gateway] [Monitoring stack]   │  │
  │  └─────────────────────────────────────────────────────────┘  │
  │                                                               │
  │  High availability: node failure tolerated                   │
  │  Scale: add nodes for more capacity                          │
  │  Cost: $3,000–$30,000/site                                   │
  └───────────────────────────────────────────────────────────────┘
          │                        │
  ┌───────┴───────┐        ┌───────┴──────┐
  │ 500+ devices  │        │ 50+ cameras  │
  │ (sensors,     │        │ (RTSP stream)│
  │  actuators)   │        └──────────────┘
  └───────────────┘
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Chi tiết |
|---|---|---|
| **High Availability** | ✅ Cao | Node failure tolerated (1/3 nodes); no single point of failure |
| **Throughput** | ✅ Cao | Horizontal scale trong cluster; Kafka at edge cho high-volume |
| **Cost** | ❌ Cao | $3,000–$30,000/site; justified chỉ khi site lớn |
| **Complexity** | ❌ Cao | K3s administration; distributed storage (Longhorn); cluster networking |
| **Team requirement** | ❌ Cao | Cần Kubernetes expertise; không phải mọi IoT team có |
| **Observability** | ⚠️ Trung bình | Prometheus/Grafana stack tại edge; nhưng cluster-level tracing |

**Khi dùng Pattern này:**
- Smart city district (> 1,000 devices/site)
- Large hospital campus (critical care + monitoring)
- Enterprise campus với mixed workloads (OT + IT convergence)
- High availability là yêu cầu (không thể có single point of failure tại edge)

---

### Pattern 4: Disconnected Operations (Offline-First)

**Mô tả:** Edge vận hành **hoàn toàn tự chủ** — cloud là thứ yếu, không phải primary. Thiết kế cho environments có connectivity unreliable hoặc không có (oil rigs, remote mines, ships). Cloud sync xảy ra theo lịch hoặc cơ hội (khi có satellite/radio window).

**Topology:**

```
  Remote Site (oil rig, mine, ship)
  ┌──────────────────────────────────────────────────────────────┐
  │                 OFFLINE-FIRST EDGE SYSTEM                    │
  │                                                              │
  │  ┌────────────────────────────────────────────────────────┐  │
  │  │              LOCAL HISTORIAN (OSIsoft PI / InfluxDB)   │  │
  │  │  - Full sensor data: 1 year+ retention                 │  │
  │  │  - All process variables: 1-second resolution          │  │
  │  │  - Event log: complete audit trail                     │  │
  │  └────────────────────────────────────────────────────────┘  │
  │                                                              │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐   │
  │  │ DCS / SCADA  │  │ Safety System│  │ Local Operator   │   │
  │  │ (full control│  │ (SIL-rated)  │  │ HMI Dashboards   │   │
  │  │  w/o cloud)  │  │              │  │ (no cloud needed) │   │
  │  └──────────────┘  └──────────────┘  └──────────────────┘   │
  │                                                              │
  │  Connected to cloud: 0–8 hours/day (satellite window)        │
  │  During sync: compressed delta upload + command receive      │
  └──────────────────────────────────────────────────────────────┘
          │ when connected (satellite, 4G LTE, ship-to-shore)
          ▼
     ┌──────────────────────────────────────────┐
     │  Cloud (secondary, not primary)          │
     │  - Receive compressed delta: daily batch │
     │  - Push config/firmware/model updates    │
     │  - Fleet benchmarking, trends            │
     └──────────────────────────────────────────┘
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Chi tiết |
|---|---|---|
| **Offline Resilience** | ✅ Maximum | Designed for months of disconnected operation |
| **Data completeness** | ✅ Hoàn toàn | Full historian: không mất data dù offline bao lâu |
| **Real-time cloud** | ❌ Không có | Cloud chỉ nhận data theo batch; không có real-time fleet visibility |
| **Cloud integration** | ❌ Phức tạp | Delta compression, conflict resolution, reconnect logic |
| **Cost** | ❌ Cao | Full historian + DCS/SCADA + rugged hardware |
| **Compliance** | ✅ Tốt | OPC UA, IEC 62443, complete audit trail |

**Khi dùng Pattern này:**
- Oil & gas: offshore platforms, remote wellsites
- Mining: underground mines, open-cut remote sites
- Maritime: ship systems, submarine cables monitoring
- Military/Defense: forward operating bases
- Cần full operational capability khi hoàn toàn mất kết nối

---

### So sánh 4 Patterns

| Tiêu chí | P1: Lightweight GW | P2: Intelligent Edge | P3: Fog Cluster | P4: Disconnected Ops |
|---|---|---|---|---|
| **Hardware cost/site** | $50–$200 | $600–$2,500 | $3,000–$30,000 | $10,000–$100,000 |
| **AI inference** | Không | ✅ Real-time | ✅ Distributed | ✅ Local only |
| **Offline duration** | Giờ–ngày | Giờ–ngày | Ngày–tuần | Tháng |
| **Team complexity** | Thấp | Trung bình | Cao | Rất cao |
| **Best for** | Smart building, agri | Factory, medical | Smart city, campus | Oil, mining, maritime |
| **Scale per site** | < 50 devices | 50–500 devices | 500–10,000 devices | 10–2,000 devices |

---

## 6. Edge Hardware và Runtime Platform

### 6.1 Edge Hardware Tiers

| Tier | Đại diện | CPU | GPU/NPU | RAM | Power | Cost | Use case |
|---|---|---|---|---|---|---|---|
| **Microgateway** | Raspberry Pi 4, Orange Pi | ARM Cortex-A72, 4-core | Không | 4–8 GB | 5–15W | $50–$150 | Smart building, agriculture |
| **Industrial Gateway** | Moxa UC-8200, Beckhoff CX | ARM/x86, 2–4 cores | Không | 2–8 GB | 10–25W | $300–$800 | Factory gateway, substation |
| **AI Edge** | NVIDIA Jetson Orin NX | Arm Cortex-A78, 8-core | 1024 CUDA cores, 32 TOPS | 16 GB | 25–40W | $500–$800 | Machine vision, robotics |
| **AI Edge Max** | NVIDIA Jetson AGX Orin | Arm Cortex-A78, 12-core | 2048 CUDA, 275 TOPS | 32–64 GB | 40–65W | $600–$1,500 | Multi-camera, LLM edge |
| **Industrial PC** | Dell Edge 3200, HPE GL20 | Intel Core i5–i9 | Integrated | 16–64 GB | 45–100W | $1,000–$4,000 | Fog computing, OT gateway |
| **Edge Server** | Dell PowerEdge XR12, HPE Edgeline EL8000 | Xeon, 16–32 cores | Optional GPU | 64–512 GB | 300–600W | $5,000–$20,000 | Large site fog node |

*(Nguồn: [NVIDIA Jetson Lineup 2024](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/), [Dell Edge Computing Portfolio](https://www.dell.com/en-us/dt/solutions/edge-computing/))*

### 6.2 Edge Runtime Platforms

**Managed Edge Runtimes (cloud-integrated):**

| Platform | Vendor | Model | Cơ chế deployment | Strengths | Limitations |
|---|---|---|---|---|---|
| **AWS IoT Greengrass v2** | AWS | Open source core + AWS managed | Docker containers + Lambda at edge | Native AWS integration; SageMaker Edge ML | Requires AWS IoT Core |
| **Azure IoT Edge** | Microsoft | Open source runtime | Docker modules | Rich marketplace; Digital Twin integration | Requires Azure IoT Hub |
| **GCP IoT Edge** | Google | Coral Edge TPU + Cloud IoT (deprecated) | Custom | ⚠️ IoT Core khai tử 2023 — không khuyến khích | Không có native managed edge runtime |

**Open Source / Vendor-Neutral Runtimes:**

| Platform | Maintained by | Strengths | IoT fit |
|---|---|---|---|
| **K3s** | CNCF (Rancher/SUSE) | Full Kubernetes API; 512MB footprint; ARM support | ✅ Fog computing, multi-workload sites |
| **MicroK8s** | Canonical | Snap install; minimal; addon system | ✅ Ubuntu-based edge |
| **Eclipse ioFog** | Eclipse Foundation | Microservice mesh at edge; vendor-neutral | ✅ Multi-cloud edge |
| **Balena** | Balena | Fleet management; OTA-first; developer UX | ✅ Consumer IoT, prototyping |
| **Mosquitto + custom** | Eclipse | Lightweight MQTT broker only | ✅ Simple gateway, no orchestration |

### 6.3 AWS Greengrass vs Azure IoT Edge — So sánh Chi tiết

| Tiêu chí | AWS Greengrass v2 | Azure IoT Edge |
|---|---|---|
| **Core architecture** | Component model (Java-based nucleus) | Docker module runtime |
| **Deployment unit** | Greengrass component (zip + recipe) | Docker container |
| **Lambda at edge** | ✅ Lambda functions chạy locally | ❌ Không native (cần Function app container) |
| **ML at edge** | ✅ SageMaker Edge Manager tích hợp | ✅ Azure ML + ONNX Runtime |
| **Device shadow** | ✅ IoT Core Device Shadow sync | ✅ IoT Hub Device Twin sync |
| **Offline operation** | ✅ MQTT local broker; offline capable | ✅ Offline mode; local message routing |
| **OTA updates** | ✅ Greengrass deployment API | ✅ Automatic module updates |
| **Community/ecosystem** | Lớn hơn; IoT-focused | Lớn; enterprise IT-focused |
| **Pricing** | Không phụ phí Greengrass; pay per IoT Core | IoT Hub tier pricing |
| **Minimum hardware** | 1 GB RAM, 500 MB disk | 1 GB RAM, 4 GB disk (Docker overhead) |
| **Best for** | AWS-native IoT stacks | Azure/Microsoft enterprise stacks |

---

## 7. Phân tích Trade-offs Sâu

### 7.1 Mô hình Chi phí — CapEx vs OpEx

Edge-Cloud Hybrid có **profile chi phí ngược với Serverless**: CapEx cao upfront, nhưng OpEx thấp hơn ở scale.

**So sánh tổng chi phí 3 năm: Factory 500 cameras**

```
OPTION A: Pure Cloud (AWS Rekognition Video)
─────────────────────────────────────────────
Video ingestion: 500 cameras × 30fps × 8h/day
= 500 × 108,000 frames/day = 54,000,000 frames/day

AWS Rekognition: $0.001/1,000 frames analyzed
Cost/day: 54,000,000 × $0.001/1,000 = $54/day
Cost/year: $54 × 365 = $19,710/year
Cost 3 years: ~$59,130

Bandwidth (1080p video upload):
500 cameras × 3 Mbps = 1.5 Gbps sustained
AWS data transfer: $0.09/GB
Monthly: 1.5 Gbps × 3,600s/h × 8h × 30d / 8 × $0.09 ≈ $14,580/month
→ $175,000/year bandwidth alone ❌

OPTION B: Edge-Cloud Hybrid (NVIDIA Jetson per 5 cameras)
──────────────────────────────────────────────────────────
Hardware: 100 Jetson AGX Orin × $1,500 = $150,000 (Year 1 CapEx)
Maintenance/support: $15,000/year
Cloud (metadata only): $200/month = $2,400/year
Power: 100 × 65W × 24h × 365 × $0.12/kWh = $6,820/year

Year 1 total: $150,000 + $15,000 + $2,400 + $6,820 = $174,220
Year 2–3 total: ($15,000 + $2,400 + $6,820) × 2 = $48,440

3-year total: $174,220 + $48,440 = $222,660

VERDICT: Pure cloud bandwidth cost alone ($525,000) >> Edge CapEx ($222,660)
Edge saves ~$300,000 over 3 years at this scale.
```

**Break-even Formula:**

```
Edge investment justified when:
  (Annual cloud cost savings) × (Hardware lifetime in years) > Edge CapEx

Annual savings = Cloud compute cost + Cloud bandwidth cost - Edge OpEx
Hardware lifetime = 5–7 years for industrial hardware

Rule of thumb: Edge justified when:
  - Video analytics workloads at any scale (bandwidth dominates)
  - > 1,000 events/sec from sensors AND cloud processing > $0.0001/event
  - Offline requirement exists (cloud cost savings = ∞, can't operate without edge)
```

### 7.2 Synchronization Complexity — Conflict Resolution

Đây là **nguồn gốc của hầu hết bugs** trong Edge-Cloud Hybrid. Khi edge và cloud đều có thể modify state, xung đột là tất yếu sau offline periods.

**Ba chiến lược conflict resolution:**

**Strategy 1: Last-Write-Wins (LWW)**
```
Mô tả: Bên nào có timestamp mới hơn thì thắng
Ưu: Đơn giản để implement
Nhược: Timestamp drift giữa edge và cloud có thể gây wrong winner
       Mất updates nếu hai bên write trong cùng window
Dùng cho: Device configuration (latest config là correct config)
```

**Strategy 2: Edge-Wins for Operational Data**
```
Mô tả: Operational data (sensor readings, events) từ edge luôn là truth
        Cloud không được overwrite actual sensor data
Ưu: Không mất sensor data; semantically correct (edge đo thực tế)
Nhược: Cloud-side manual corrections bị ignore
Dùng cho: Telemetry, sensor readings, events
```

**Strategy 3: CRDT (Conflict-free Replicated Data Types)**
```
Mô tả: Data structures được thiết kế để merge deterministically
        Không cần timestamp comparison; merge luôn đúng
Ví dụ CRDT cho IoT:
  - G-Counter (grow-only): total device uptime hours
  - LWW-Register: device configuration (last-write-wins với metadata)
  - OR-Set (observed-remove): active alert set
Ưu: Mathematically guaranteed consistency; không mất data
Nhược: Chỉ một số data types phù hợp với CRDT structure
Dùng cho: Distributed counters, sets, registries
```

*(Nguồn: [Martin Kleppmann — CRDTs for IoT](https://crdt.tech/), [Shapiro et al. "A Comprehensive Study of CRDTs"](https://inria.hal.science/inria-00555588))*

### 7.3 OTA Management ở Scale

Cập nhật software đến 10,000 edge nodes là một trong những vận hành phức tạp nhất:

**Staged Rollout Strategy:**

```
Phase 1 — Canary (1% = 100 sites)
  - Chọn sites đại diện (nhỏ, ít critical, có monitoring tốt)
  - Deploy, observe 24–48 giờ
  - Metrics: error rate, latency, CPU/RAM, sync health
  - Go/No-Go: auto-rollback nếu error rate tăng > 2%

Phase 2 — Early Adopter (10% = 1,000 sites)
  - Sites được chọn có technical owner active
  - Observe 48–72 giờ
  - Go/No-Go: manual review by team

Phase 3 — General Release (50% → 100%)
  - Batch sizes tăng dần
  - Auto-rollback vẫn active
  - Full deployment hoàn thành trong 7–14 ngày
```

**Health Check Criteria (phải pass trước khi advance):**

| Metric | Threshold | Action nếu fail |
|---|---|---|
| Error rate delta | < +1% vs baseline | Auto-rollback |
| P99 latency delta | < +20% vs baseline | Auto-rollback |
| CPU usage | < 90% average | Hold; investigate |
| Memory usage | < 85% | Hold; investigate |
| Sync lag | < 60 seconds | Hold; investigate |
| Model inference accuracy | > 95% of baseline | Hold; investigate |

### 7.4 Latency Tiering — Quyết định Xử lý ở Tầng Nào

Một trong những quyết định kiến trúc quan trọng nhất:

| Latency requirement | Tầng xử lý | Ví dụ | Công nghệ |
|---|---|---|---|
| **< 1ms** | On-device firmware | Emergency stop, safety interlock | Hardwired logic, FPGA, RTOS |
| **1–10ms** | Edge gateway (same LAN) | HVAC control, conveyor speed | Local rules engine |
| **10–100ms** | Edge server (AI inference) | Quality control vision, anomaly detection | TensorRT, ONNX Runtime |
| **100ms–1s** | Cloud (simple analytics) | Alert aggregation, dashboard update | Cloud functions |
| **> 1s** | Cloud (complex analytics) | Trend analysis, ML training, fleet benchmarking | Spark, Databricks |

**Nguyên tắc:** Đẩy quyết định về phía thiết bị càng nhiều càng tốt, chỉ escalate lên tầng trên khi thiếu context hoặc cần fleet-level intelligence.

### 7.5 Physical Security — Attack Surface của Edge Hardware

Edge hardware tại site là **attack surface vật lý** không tồn tại trong cloud-only architecture:

| Rủi ro | Kịch bản | Giảm thiểu |
|---|---|---|
| **Physical access** | Attacker lấy edge device → extract TLS certs → impersonate edge | TPM-backed keys; Secure Boot; tamper detection |
| **Credential extraction** | Private keys trong filesystem → replay cloud connection | Keys chỉ trong TPM; không export được |
| **Firmware tampering** | Flash custom firmware → bypass security → plant backdoor | Secure Boot + code signing; attestation |
| **Network sniffing** | Capture local MQTT traffic between devices and edge | TLS between devices and edge (even on LAN) |
| **Supply chain** | Malicious firmware/component from manufacturer | Vendor attestation; verify before deploy |

---

## 8. Bảo mật

### 8.1 Network Segmentation — Defense in Depth

Edge-Cloud Hybrid tự nhiên tạo ra network segmentation tốt — thiết bị không bao giờ expose trực tiếp ra internet:

```
INTERNET
    │ HTTPS/MQTT TLS
    ▼
┌─────────────────────────────────────────────────────────────┐
│                         CLOUD DMZ                           │
│  IoT Core / IoT Hub (managed broker)                        │
│  API Gateway (authenticated endpoints)                      │
└─────────────────────────────┬───────────────────────────────┘
                              │ mTLS (mutual TLS)
                              │ Edge certificate pinned
┌─────────────────────────────▼───────────────────────────────┐
│                      EDGE ZONE (site)                       │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐    │
│  │  Edge Node (protocol bridge + compute)              │    │
│  │  - Only outbound connections to cloud               │    │
│  │  - No inbound ports open from internet              │    │
│  └─────────────────────────────────────────────────────┘    │
│                           │ MQTT/TLS (local LAN)             │
│                           │ OPC UA with security             │
└───────────────────────────┼─────────────────────────────────┘
                            │ Local network only
┌───────────────────────────▼─────────────────────────────────┐
│                    OT ZONE (devices)                        │
│  Sensors, actuators, PLCs, cameras                          │
│  No direct internet access — ever                           │
└─────────────────────────────────────────────────────────────┘
```

**Nguyên tắc bảo mật từng zone:**
- **OT Zone**: Thiết bị không bao giờ có IP route ra internet; firewall hardcoded
- **Edge Zone**: Chỉ outbound connections; không listening ports từ WAN; mTLS với cloud
- **Cloud DMZ**: IoT broker là single entry point; per-device authorization policies

### 8.2 Zero Trust cho Edge Nodes

Edge node **không được tin tưởng mặc định**, dù nằm trong corporate network:

| Nguyên tắc Zero Trust | Áp dụng cho Edge |
|---|---|
| **Verify explicitly** | Edge authenticate bằng X.509 certificate; cloud verify mỗi connection |
| **Least privilege access** | Edge chỉ có permission publish/subscribe đúng topics; không có admin access |
| **Assume breach** | Nếu edge bị compromise: chỉ site đó bị ảnh hưởng; credential rotation isolates it |

### 8.3 Quản lý Certificate ở Scale

10,000 edge nodes mỗi node có TLS certificate — rotation là operational challenge:

**Certificate lifecycle automation:**

```
CA (Certificate Authority)
    │
    │[1] Bootstrap: factory-install device certificate (one-time)
    │[2] Operational: auto-renew 90 days trước expiry
    │    - SCEP (Simple Certificate Enrollment Protocol)
    │    - EST (Enrollment over Secure Transport)
    │    - AWS IoT Certificate rotation automation
    │[3] Revocation: if edge compromised → immediate revoke → device quarantined
    │[4] Rotation: new cert deployed via OTA; hot-swap without restart
```

**Thực hành tốt nhất:**
- Certificate lifetime: 1 năm (không vĩnh viễn)
- Lưu private key trong TPM — không export được
- Monitor expiry: alert 30 ngày trước expire
- Immediate revocation endpoint: phải phản hồi < 1 giây

### 8.4 Bảo mật Software Supply Chain tại Edge

```
Cloud ML Pipeline              Edge Node
[Train model] ──────────────▶  [Download package]
[Sign artifact] ─────────────▶  [Verify signature]
  (Sigstore/cosign)              (public key pinned in edge config)
[Upload to registry] ────────▶  [Unpack if valid]
                                 [Reject if invalid — alert + log]
```

- **Container images**: signed với Sigstore/cosign; edge verifies trước khi pull
- **OTA packages**: signed với vendor private key; edge verifies hash
- **ML models**: ONNX files signed; version-locked in deployment manifest
- **Dependency scanning**: CI/CD pipeline scan tất cả edge container images

---

## 9. Observability

### 9.1 Multi-Tier Observability Architecture

Edge-Cloud Hybrid cần observability **tại mỗi tầng** vì data có thể không đến cloud ngay:

```
  DEVICE TIER METRICS (collected by edge)
  ├── Device connectivity status (connected/disconnected)
  ├── Message publish rate (messages/sec per device)
  └── Last-seen timestamp

  EDGE TIER METRICS (local + sync to cloud)
  ├── Edge node health: CPU%, RAM%, disk%, temperature
  ├── Connectivity to cloud: connected/disconnected, latency
  ├── Sync lag: seconds since last successful sync
  ├── Local buffer size: MB queued for cloud sync
  ├── Inference latency P50/P99 (if AI)
  └── Rules engine: decisions/sec, false positive rate

  CLOUD TIER METRICS (full visibility when online)
  ├── Fleet health: % edge nodes reporting
  ├── Sync completeness: % messages delivered vs expected
  ├── Model performance: accuracy per edge node
  └── End-to-end latency: device event → cloud receipt
```

### 9.2 Distributed Tracing Qua Tiers

Để trace một anomaly từ sensor đến cloud alert:

```
  Correlation ID: deviceId + eventTimestamp (immutable)

  Device         Edge           Cloud
  [T=0ms]        [T=3ms]        [T=850ms]
  sensor read → edge receive → cloud receive
  correlationId  correlationId  correlationId
  = "dev42:14:30:00.123"  (same ID propagated)

  Trace reconstruction:
  CloudWatch / Application Insights / Jaeger
  → Query by correlationId
  → See full path: device time → edge processing time → cloud receipt time
  → Identify: where did latency come from?
```

### 9.3 Offline Period Observability Gaps

Khi edge offline, cloud mất visibility — đây là một **inherent trade-off**:

| Trạng thái | Cloud visibility | Giải pháp |
|---|---|---|
| Edge online | Full real-time | Không cần giải pháp |
| Edge offline (< buffer size) | Mất real-time; có retroactive sau reconnect | Buffer replay; timestamp-aware dashboards |
| Edge offline (> buffer size) | Permanent data loss | Tăng buffer; alert khi buffer near full |

**Monitoring pattern:**
```
Alert khi: edge_last_seen > 5 minutes → "Edge X offline"
Alert khi: buffer_size > 80% capacity → "Edge X buffer near full — data loss risk"
Dashboard: show "last seen" timestamp, không phải current data, khi offline
```

### 9.4 Key Metrics và Ngưỡng Alert

| Metric | Ngưỡng Warning | Ngưỡng Critical | Action |
|---|---|---|---|
| Edge connectivity | Offline > 5min | Offline > 30min | PagerDuty alert; on-call response |
| Sync lag | > 60 seconds | > 300 seconds | Check network; check buffer |
| Buffer fill rate | > 70% | > 90% | Increase cloud sync rate; alert data loss risk |
| Edge CPU | > 80% | > 95% | Scale hardware; optimize workload |
| Inference latency P99 | > 200ms | > 500ms | Model optimization; hardware upgrade |
| Certificate expiry | 30 days | 7 days | Trigger renewal workflow |
| OTA success rate | < 95% sites | < 80% sites | Pause rollout; investigate failures |

---

## 10. Triển khai Thực tế

### Case Study 1: Rolls-Royce IntelligentEngine — Jet Engine Predictive Maintenance

**Bối cảnh:**
Rolls-Royce vận hành dịch vụ **TotalCare** — giám sát liên tục 4,500+ động cơ phản lực thương mại toàn cầu. Mô hình kinh doanh "Power by the Hour": hãng hàng không trả tiền theo giờ bay, không theo sự kiện bảo dưỡng. Nếu Rolls-Royce không dự đoán được hỏng hóc, họ phải chịu chi phí. Mỗi động cơ Trent mang **hàng nghìn cảm biến** — nhiệt độ, áp suất, vận tốc, rung động — tạo ra luồng dữ liệu liên tục trong suốt chuyến bay.

**Kiến trúc Edge-Cloud Hybrid:**

```
  JET ENGINE (in flight)
  ┌─────────────────────────────────────────────────────┐
  │  1,000+ sensors/engine                              │
  │  ECU (Engine Control Unit) — EDGE NODE              │
  │  - Local preprocessing: filter, compress            │
  │  - Anomaly flag detection (firmware-level)          │
  │  - Data buffering for transmission windows          │
  └──────────────────────────────────┬──────────────────┘
                                     │ ACARS (Aircraft Communications)
                                     │ Satellite / VHF radio
                                     │ (periodic transmission windows)
                                     ▼
  ┌─────────────────────────────────────────────────────┐
  │  Microsoft Azure IoT Backend                        │
  │  - Data normalization (multiple engine variants)    │
  │  - Azure Databricks analytics + ML                  │
  │  - Digital Twin (real-time virtual engine model)    │
  │  - Rolls-Royce Availability Centers (24/7 monitor)  │
  └─────────────────────────────────────────────────────┘
```

**Đặc tính Edge nổi bật:**
- **Offline operation**: Máy bay ở tầng cao không có liên tục ground connection — ECU buffer data và transmit khi có window
- **Bandwidth constraint**: ACARS bandwidth rất hạn chế (< 2,400 baud truyền thống); ECU phải compress và filter cực mạnh trước khi gửi
- **Latency**: Safety-critical decisions (engine shutdown) phải xảy ra tại ECU (< 1ms) — không thể cloud

**Kết quả:**
- **Hàng triệu USD** tiết kiệm từ tránh unscheduled maintenance (Microsoft Customer Story)
- **4,500+ engines** giám sát real-time toàn cầu
- **Digital twin** accuracy: model virtual đồng bộ với sensor data thực
- **TotalCare commercial model** được kích hoạt: chỉ có thể bán "Power by the Hour" khi có predictive capability

**Bài học architecture:**
- **Business model và architecture là không thể tách rời**: kiến trúc IoT không phải feature — đây là sản phẩm
- **ECU là edge node**: không phải chỉ là sensor collector; là một computing platform đầy đủ
- **Bandwidth constraint drives architecture**: ACARS giới hạn buộc xử lý maximum tại edge

*(Nguồn: [Microsoft Customer Story — Rolls-Royce](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks), [RTInsights Analysis](https://www.rtinsights.com/rolls-royce-jet-engine-maintenance-iot/))*

---

### Case Study 2: John Deere — Precision Agriculture tại 500 Triệu Acres

**Bối cảnh:**
John Deere chuyển đổi từ nhà sản xuất thiết bị thành **công ty dữ liệu và công nghệ**. Platform kết nối quản lý 1.5 triệu+ máy móc (máy cày, máy gặt, máy phun, máy trồng) trên **500 triệu acres**. Mục tiêu: AI quyết định lượng hạt giống, phân bón, thuốc trừ sâu tối ưu **cho từng cây trong từng hàng** — xử lý hàng nghìn cây mỗi giây. Data volume tăng gấp đôi mỗi năm.

**Kiến trúc Edge-Cloud Hybrid:**

```
  FIELD MACHINE (tractor / combine)
  ┌─────────────────────────────────────────────────────┐
  │  Edge Compute Platform (JDLink + onboard computer)  │
  │                                                     │
  │  - GPS + RTK: sub-inch field positioning            │
  │  - AI inference: seed rate per plant (< 10ms)       │
  │  - Local prescription map: pre-loaded from cloud    │
  │  - Sensor fusion: soil, weather, plant data         │
  │  - Actuator control: seeder, sprayer, planter       │
  │                                                     │
  │  Operates FULLY offline during field work           │
  │  (cellular in rural fields: intermittent)           │
  └──────────────────────────────────┬──────────────────┘
                                     │ Cellular (4G/LTE)
                                     │ When available
                                     │ WiFi at farm HQ (daily sync)
                                     ▼
  ┌─────────────────────────────────────────────────────┐
  │  AWS Cloud Platform                                 │
  │  - Operations Center (farmer dashboard)             │
  │  - Field data aggregation + analytics               │
  │  - ML model training (next season prescriptions)    │
  │  - Fleet management: software updates               │
  │  - John Deere Financial integration                 │
  └─────────────────────────────────────────────────────┘
```

**Đặc tính Edge nổi bật:**
- **Offline-first by necessity**: Cánh đồng 500 triệu acres bao gồm nhiều vùng không có cellular — máy móc PHẢI hoạt động offline
- **< 10ms AI inference**: Seeder chạy 10km/h qua field; mỗi hàng cần decision trong < 10ms — không thể cloud
- **Prescription map pre-loaded**: Cloud train AI model → tạo prescription map for specific field → push to tractor before operation

**Scale:**
- 1.5 triệu+ connected machines
- 500 triệu+ acres managed
- Xử lý hàng nghìn cây/giây trên mỗi máy

**Kết quả được công bố:**
- Individual-plant precision: yield tăng 5–10% với đầu vào ít hơn (fertilizer, water, pesticide)
- Farmer ROI: data-driven decisions giảm input cost
- John Deere revenue shift: recurring data service revenue supplement hardware sales

**Bài học architecture:**
- **Offline-first không phải edge-first**: Connectivity là không đáng tin cậy trong agriculture — architecture phải thiết kế offline như default
- **Prescription map = edge pre-computation**: Cloud làm ML training; edge làm inference at speed
- **The machine IS the edge**: Không cần separate gateway hardware — compute tích hợp vào máy móc

*(Nguồn: [Databricks Blog — John Deere Industrial AI](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html), [Harvard D3 Case Study](https://d3.harvard.edu/platform-digit/submission/farm-to-data-table-john-deere-and-data-in-precision-agriculture/))*

---

### Case Study 3: Siemens — Industrial Edge tại Factory Floor

**Bối cảnh:**
Siemens vận hành **Industrial Edge** platform để đưa IT compute đến OT factory floor. Target: nhà máy sản xuất có SCADA/DCS truyền thống muốn thêm AI và analytics mà không thay thế hạ tầng OT hiện có. Siemens là cả **người dùng** (trong nhà máy của mình) và **người bán** platform.

**Kiến trúc:**

```
  FACTORY FLOOR (OT environment)
  ┌─────────────────────────────────────────────────────┐
  │  Existing OT Infrastructure (không thay đổi)        │
  │  PLCs, SCADA, CNC machines, robots                  │
  │  Protocols: OPC UA, Profinet, Modbus                │
  └──────────────────────────┬──────────────────────────┘
                             │ OPC UA (OT data extraction)
  ┌──────────────────────────▼──────────────────────────┐
  │  Siemens Industrial Edge (IED — Industrial Edge Device)│
  │                                                      │
  │  - OPC UA data collection from PLCs                 │
  │  - Edge apps marketplace (300+ apps):               │
  │    - Siemens AI Inference Server                    │
  │    - Siemens Notifier (real-time alerting)          │
  │    - Third-party analytics apps                     │
  │  - Docker-based deployment                          │
  │  - Managed from cloud (Industrial Edge Management)  │
  └──────────────────────────┬──────────────────────────┘
                             │ MQTT/TLS to cloud
                             │ Selective data forwarding
  ┌──────────────────────────▼──────────────────────────┐
  │  Siemens MindSphere (Cloud)                         │
  │  - Fleet-wide production analytics                  │
  │  - ML model training                                │
  │  - OEE (Overall Equipment Effectiveness) tracking  │
  │  - Remote maintenance                               │
  └─────────────────────────────────────────────────────┘
```

**Đặc tính nổi bật:**
- **Non-invasive**: Thêm edge capabilities mà không thay thế OT systems hiện có — chỉ thêm edge device kết nối qua OPC UA
- **Marketplace**: 300+ edge apps từ Siemens và third-party; giảm development time
- **OT/IT convergence**: Bridge giữa PLC data (millisecond precision) và IT analytics (second precision)

**Kết quả tại Siemens factories:**
- Amberg Electronics Factory: 99.99885% quality rate — edge AI phát hiện defects real-time
- Predictive maintenance: giảm unplanned downtime 50% tại một số dây chuyền
- Energy optimization: edge analytics identify waste patterns; 10–15% energy reduction

**Bài học architecture:**
- **Non-invasive integration là key cho OT adoption**: Factory operators không muốn thay PLC; edge layer là "IT on top of OT"
- **Edge app marketplace accelerates deployment**: Không cần build từ đầu — curate và configure
- **OPC UA là lingua franca của IIoT edge**: Standardize OT data extraction trước khi process

*(Nguồn: [Siemens Industrial Edge](https://www.siemens.com/global/en/products/automation/industry-software/industrial-edge.html), [Siemens Amberg Factory Case Study](https://www.siemens.com/global/en/company/stories/industry/the-twin-plant.html))*

---

### Tổng kết Case Studies

| Công ty | Scale | Pattern chính | Offline requirement | Kết quả nổi bật |
|---|---|---|---|---|
| **Rolls-Royce** | 4,500 engines, global | Pattern 4: Disconnected Ops | Bắt buộc (ACARS limited) | Millions USD savings; TotalCare enabled |
| **John Deere** | 1.5M machines, 500M acres | Pattern 4 + Pattern 2 | Bắt buộc (rural coverage) | Per-plant precision; 5–10% yield increase |
| **Siemens Industrial Edge** | Hàng ngàn factories | Pattern 2: Intelligent Edge | Có (factory OT resilience) | 99.99885% quality; 50% less downtime |

---

## 11. Khi nào Dùng / Khi nào Không Dùng

> Đây là mục quan trọng nhất theo style *Fundamentals of Software Architecture*: architecture tốt là
> architecture phù hợp với context — không có pattern "tốt nhất" tuyệt đối.

### 11.1 Khi nào Dùng Edge-Cloud Hybrid ✅

**1. Latency < 10ms là yêu cầu cứng (hard real-time)**
- Ví dụ: Safety interlock tại factory (emergency stop < 5ms)
- Tiêu chí: WAN latency không thể guarantee < 10ms; bất kỳ outage nào là unacceptable
- Không có lựa chọn thay thế nào đáp ứng được yêu cầu này ngoài edge

**2. Offline operation là bắt buộc**
- Ví dụ: Oil rig operations, underground mining, ship systems, remote agriculture
- Tiêu chí: Cloud unavailability > 0 là không chấp nhận được về operations
- Pattern 4 (Disconnected Ops) là mandatory

**3. Bandwidth cost prohibitive**
- Ví dụ: 500+ cameras, high-frequency vibration sensors (10kHz sampling)
- Tiêu chí: Raw data bandwidth > $5,000/tháng; processing at source saves > 90%
- Break-even analysis sẽ justify edge CapEx trong < 18 tháng

**4. Data sovereignty và compliance bắt buộc**
- Ví dụ: GDPR — patient data không được rời EU hospital; HIPAA — PHI không rời facility
- Tiêu chí: Regulatory requirement cụ thể; không phải preference
- Edge processing đảm bảo data không rời facility về mặt kỹ thuật

**5. AI inference real-time tại nguồn**
- Ví dụ: Defect detection trên production line (30fps, < 50ms), autonomous vehicle guidance
- Tiêu chí: Cloud round-trip không đáp ứng latency; local GPU/NPU justified bởi accuracy requirement
- Pattern 2 (Intelligent Edge) phù hợp

**6. Physical/industrial environment không thể kết nối cloud liên tục**
- Ví dụ: Machinery trong hầm, di chuyển, môi trường RF-shielded
- Tiêu chí: Connectivity intermittent by physics, không phải design choice

---

### 11.2 Khi nào KHÔNG Dùng Edge-Cloud Hybrid ❌

**1. Team thiếu expertise về distributed systems**
- Lý do: Synchronization bugs, offline/reconnect logic, OTA at scale cần deep expertise
- Ngưỡng: Team < 5 người; không có distributed systems hoặc embedded experience
- Hậu quả: Data loss, split-brain, undetected sync failures — catastrophic trong production
- Thay thế: Bắt đầu với Serverless hoặc Microservices; thêm edge khi team đủ capability

**2. Budget CapEx không cho phép edge hardware**
- Lý do: Edge hardware $50–$30,000/site; upfront cost lớn
- Ngưỡng: CapEx budget < $1,000/site và volume cần xử lý low
- Thay thế: Cloud-only với Serverless (tốt cho sporadic, low-volume workloads)

**3. Cloud latency 50–200ms là chấp nhận được**
- Lý do: Nếu không có hard real-time requirement, complexity của edge không justified
- Ngưỡng: Latency SLA > 500ms cho tất cả critical paths
- Thay thế: Microservices + EDA trên cloud

**4. Early-stage product cần pivot nhanh**
- Lý do: Edge hardware là committed CapEx; khó thay đổi khi product direction thay đổi
- Ngưỡng: < 6 tháng old product; business model chưa validated
- Thay thế: Pure cloud architecture → validate product-market fit → add edge khi scale

**5. Connectivity reliable và data không nhạy cảm, volume thấp**
- Lý do: Nếu cloud-only xử lý được, không cần thêm operational complexity
- Ngưỡng: Uptime > 99.9%; data không nhạy cảm; < 1,000 events/sec
- Thay thế: Serverless Pattern 1 hoặc Microservices + EDA

**"Edge-Cloud Hybrid breaks down when" — Bảng tóm tắt:**

| Điều kiện | Hậu quả | Giải pháp |
|---|---|---|
| Team thiếu sync expertise | Data loss sau reconnect; split-brain | Hire hoặc train; hoặc dùng managed sync (Greengrass/IoT Edge) |
| Buffer đầy trước khi reconnect | Permanent data loss | Tăng storage; giảm retention; alert sớm |
| OTA failed tại 10% sites | Inconsistent fleet state; hard to debug | Staged rollout; auto-rollback |
| Physical security breach | Edge credentials compromised → cloud access | Certificate revocation; physical tamper detection |
| Too many edge nodes to manage | Operational overhead exceeds benefit | Managed edge platform (Greengrass, IoT Edge) |

---

## 12. Framework Quyết định

### 12.1 Six Questions Framework

Trả lời 6 câu hỏi theo thứ tự. Câu đầu tiên trả lời **Có** sẽ chỉ ra Edge-Cloud Hybrid là bắt buộc:

```
Q1: Có yêu cầu latency < 10ms cho bất kỳ critical path nào không?
    → Có: Edge-Cloud Hybrid bắt buộc → chọn Pattern 1, 2, 3, hoặc 4
    → Không: Tiếp tục

Q2: Có yêu cầu offline operation (cloud unavailability > 0)?
    → Có: Edge-Cloud Hybrid bắt buộc → Pattern 3 hoặc 4
    → Không: Tiếp tục

Q3: Bandwidth cost có > $3,000/tháng nếu gửi raw data lên cloud?
    → Có: Edge justified bởi economics → tính break-even
    → Không: Tiếp tục

Q4: Có regulatory requirement về data residency?
    → Có: Edge processing đảm bảo compliance → mandatory
    → Không: Tiếp tục

Q5: AI inference có cần < 100ms và volume cao (> 10fps video)?
    → Có: Pattern 2 (Intelligent Edge) phù hợp
    → Không: Cloud AI inference có thể đủ

Q6: Team có đủ distributed systems + embedded expertise?
    → Không: Đừng dùng Edge-Cloud Hybrid — risks outweigh benefits
    → Có: Proceed với pattern selection
```

### 12.2 Decision Flowchart

```
                           START
                             │
              Latency < 10ms required?
             ┌──YES──────────────────────────────────────────────┐
             │                                                   │
             NO                                         Pattern 1, 2, 3, or 4
             │                                          (based on compute needs)
             ▼
    Offline operation required?
   ┌──YES─────────────────────────────────────────────┐
   │                                                  │
   NO                                         Pattern 4 (Disconnected)
   │                                          or Pattern 3 (Fog Cluster)
   ▼
  Bandwidth > $3K/month (raw)?
 ┌──YES──────────────────────────────────────────────┐
 │                                                   │
 NO                              Break-even OK?  YES → Edge (Pattern 1 or 2)
 │                                            NO → Cloud-only
 ▼
Data sovereignty required?
┌──YES─────────────────────────────────────────────┐
│                                                  │
NO                                        Edge mandatory (any pattern)
│
▼
Team has distributed systems expertise?
┌──NO──────────────────────────────────────────────┐
│                                                  │
YES                                      Cloud-only Architecture
│                                        (Serverless / Microservices)
▼
EVALUATE Edge for optimization
(not necessity — weigh complexity vs benefit)
```

### 12.3 Pattern Selection cho Edge-Cloud Hybrid

Khi đã quyết định dùng Edge-Cloud Hybrid, chọn pattern nào:

| Điều kiện | Pattern |
|---|---|
| Budget < $300/site AND no AI needed | Pattern 1: Lightweight Gateway |
| AI inference needed AND budget OK | Pattern 2: Intelligent Edge |
| > 1,000 devices/site AND HA required | Pattern 3: Fog Cluster |
| Connectivity < 50% uptime | Pattern 4: Disconnected Ops |
| Multiple conditions above | Hybrid: Pattern 2 nodes in Pattern 3 cluster |

### 12.4 Decision Matrix — 8 Use Case IoT Điển hình

| Use Case | Edge-Cloud Hybrid? | Pattern | Lý do |
|---|---|---|---|
| **Smart home** (< 1K devices) | ❌ Không cần | Serverless | Cloud latency OK; budget; small team |
| **Factory safety system** | ✅ Bắt buộc | Pattern 1/2 | < 10ms; offline; IEC 61511 |
| **Factory computer vision** | ✅ Bắt buộc | Pattern 2 | AI inference real-time; bandwidth cost |
| **Oil rig operations** | ✅ Bắt buộc | Pattern 4 | Offline weeks; safety critical |
| **Smart city traffic** | ✅ Khuyến nghị | Pattern 3 | Latency; scale; HA required |
| **Connected vehicle fleet** | ✅ Khuyến nghị | Pattern 2 | Offline driving; AI navigation |
| **Healthcare wearables** | ⚠️ Tùy | Pattern 1 | HIPAA nếu PHI; cloud OK nếu not |
| **Agricultural sensors** | ✅ Bắt buộc | Pattern 1/4 | Rural coverage; offline field work |

### 12.5 Evolution Path — Khi nào Thêm Edge vào Hệ thống Hiện có

```
Stage 1: Cloud-only (startup/MVP)
┌─────────────────────┐
│  Serverless         │  Validate product-market fit
│  or Microservices   │  < 1K devices
└────────┬────────────┘
         │ Pain signals:
         │ - Cloud latency violations
         │ - Offline complaints from customers
         │ - Bandwidth bills exploding
         ▼
Stage 2: Lightweight Edge (Pattern 1)
┌─────────────────────┐
│  Cloud + Raspberry  │  Add protocol bridge + local buffer
│  Pi gateways        │  Solve connectivity & basic latency
└────────┬────────────┘
         │ Pain signals:
         │ - Need AI inference on-site
         │ - More compute needed at edge
         ▼
Stage 3: Intelligent Edge (Pattern 2)
┌─────────────────────┐
│  Cloud + AI edge    │  Add NVIDIA Jetson or Intel NUC
│  (Jetson/OpenVINO)  │  Full ML inference on-site
└────────┬────────────┘
         │ Pain signals:
         │ - Single node failures affect site
         │ - Too many devices for one node
         ▼
Stage 4: Fog Cluster (Pattern 3) or Disconnected (Pattern 4)
┌─────────────────────┐
│  Full enterprise    │  K3s cluster OR historian-based
│  edge deployment    │  Most complex; most capable
└─────────────────────┘
```

---

## 13. Tài liệu Tham khảo

### Sách và Monographs

| Tác giả | Tựa | Nhà xuất bản | Năm | Phần liên quan |
|---|---|---|---|---|
| Richards, M. & Ford, N. | *Fundamentals of Software Architecture* | O'Reilly | 2020 | Ch.4: Architecture Characteristics; Pattern structure |
| Raj, P. & Deka, G.C. (Eds.) | *Handbook of Research on Cloud and Fog Computing Infrastructures for Data Science* | IGI Global | 2018 | Fog computing principles và IoT patterns |
| Dastjerdi, A.V. & Buyya, R. | *Fog Computing: Helping the Internet of Things Realize Its Potential* | IEEE Computer | 2016 | Fog vs cloud trade-off analysis |
| Kramer, G. et al. | *Industrial IoT: Challenges, Design Principles, Applications, and Security* | Springer | 2020 | IIoT edge patterns, OT/IT convergence |

### Standards và Specifications

| Standard | Organization | Relevance |
|---|---|---|
| **IEC 62443** | IEC | Industrial cybersecurity; network segmentation cho OT/IT convergence |
| **OPC UA (IEC 62541)** | OPC Foundation | Standard protocol cho edge-to-cloud IIoT data exchange |
| **ETSI MEC (Multi-Access Edge Computing)** | ETSI | 5G edge computing reference architecture |
| **ISO/IEC 30141** | ISO/IEC | IoT Reference Architecture; 3-tier model |
| **IEC 61511** | IEC | Functional safety cho industrial process; tiêu chuẩn SIL rating |

### Research Papers

| Tác giả | Tựa | Venue | Năm |
|---|---|---|---|
| Bonomi, F. et al. | *Fog Computing and Its Role in the Internet of Things* | ACM MCC Workshop | 2012 |
| Shi, W. et al. | *Edge Computing: Vision and Challenges* | IEEE IoT Journal | 2016 |
| Satyanarayanan, M. | *The Emergence of Edge Computing* | IEEE Computer | 2017 |
| Xu, D. et al. | *Edge Intelligence: Architectures, Challenges, and Applications* | arXiv | 2020 |
| Wang, X. et al. | *In-Edge AI: Intelligentizing Mobile Edge Computing, Caching and Communication with Heterogeneous CNNs* | IEEE Network | 2019 |

### Cloud Provider Documentation

| Nguồn | URL | Phần liên quan |
|---|---|---|
| AWS Greengrass v2 Developer Guide | [docs.aws.amazon.com/greengrass](https://docs.aws.amazon.com/greengrass/v2/developerguide/) | Component model, OTA, offline operation |
| Azure IoT Edge Documentation | [learn.microsoft.com/azure/iot-edge](https://learn.microsoft.com/en-us/azure/iot-edge/) | Module runtime, deployment manifests |
| NVIDIA Jetson Documentation | [developer.nvidia.com/embedded](https://developer.nvidia.com/embedded/develop/software) | TensorRT, DeepStream, Jetson series specs |
| K3s Documentation | [k3s.io](https://k3s.io/) | Lightweight Kubernetes for edge |
| Siemens Industrial Edge | [siemens.com/industrial-edge](https://www.siemens.com/global/en/products/automation/industry-software/industrial-edge.html) | IIoT edge platform, OPC UA integration |

### Case Study Sources

| Công ty | Nguồn |
|---|---|
| Rolls-Royce IntelligentEngine | [Microsoft Customer Story](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks) |
| John Deere Precision Agriculture | [Databricks Blog 2021](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html) |
| Siemens Industrial Edge / Amberg Factory | [Siemens IntelligentOps](https://www.siemens.com/global/en/company/stories/industry/the-twin-plant.html) |

### Industry Reports

| Nguồn | Tựa | Năm |
|---|---|---|
| Gartner | *Predicts 2024: Industrial IoT and Connected Devices* | 2023 |
| IDC | *Worldwide Edge Computing Spending Guide* | 2024 |
| Eclipse Foundation | *IoT & Edge Commercial Adoption Survey 2023* | 2023 |
| LF Edge | *State of the Edge 2023* | 2023 |
