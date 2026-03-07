# Serverless Architecture trong IoT — Phân tích Chuyên sâu

> **Phong cách trình bày:** Tài liệu này trình bày Serverless Architecture theo cấu trúc của cuốn
> *Fundamentals of Software Architecture* (Richards & Ford, O'Reilly 2020): topology → cơ chế hoạt động
> → đánh giá architecture characteristics → các biến thể pattern → khi nào dùng / khi nào không dùng
> → trade-offs → framework quyết định.

> **Tài liệu liên quan:**
> - [02-patterns.md §4](02-patterns.md#4-serverless-architecture) — tổng quan ngắn gọn về Serverless trong IoT
> - [06-architecture-characteristics.md](06-architecture-characteristics.md) — framework đánh giá đặc tính
> - [07-serverless-iot.md](07-serverless-iot.md) — tài liệu kỹ thuật tiếng Anh (cold start, cost, observability chi tiết)

---

## Mục lục

1. [Tổng quan](#1-tổng-quan)
2. [Topology](#2-topology)
3. [Cơ chế hoạt động](#3-cơ-chế-hoạt-động)
4. [Đánh giá Architecture Characteristics](#4-đánh-giá-architecture-characteristics)
5. [Các biến thể kiến trúc](#5-các-biến-thể-kiến-trúc)
6. [So sánh Cloud Provider](#6-so-sánh-cloud-provider)
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

**Serverless Architecture** là một mô hình triển khai và vận hành trong đó nhà cung cấp cloud chịu trách nhiệm toàn bộ việc quản lý cơ sở hạ tầng — provisioning máy chủ, scaling, patching, và high availability. Developer chỉ viết và deploy code (hàm hoặc logic nghiệp vụ); không cần nghĩ đến server, container, hay cluster.

Thuật ngữ "serverless" không có nghĩa là không có máy chủ — mà là developer **không thấy và không quản lý** máy chủ đó. Mô hình này bao gồm hai lớp bổ trợ lẫn nhau:

| Lớp | Tên | Định nghĩa | Ví dụ |
|---|---|---|---|
| **Compute** | FaaS — Function as a Service | Hàm stateless, kích hoạt theo sự kiện, tính tiền theo số lần gọi và thời gian thực thi | AWS Lambda, Azure Functions, Google Cloud Functions |
| **Backend** | BaaS — Backend as a Service | Dịch vụ managed thay thế hạ tầng tự vận hành | AWS IoT Core, DynamoDB, SNS, Cognito; Azure IoT Hub, Cosmos DB |

Một hệ thống IoT serverless thuần túy kết hợp FaaS + BaaS: **không có VM, không có container cluster, không có Kafka cluster tự quản lý** — tất cả đều là managed services và stateless functions.

### 1.2 Lịch sử và Bối cảnh

| Năm | Sự kiện |
|---|---|
| **2006** | Amazon Web Services ra mắt EC2 — IaaS (Infrastructure as a Service) |
| **2010–2012** | PaaS (Heroku, Google App Engine) — quản lý ứng dụng nhưng vẫn cần cấu hình server |
| **2014** | **AWS Lambda** ra mắt tại re:Invent — FaaS đầu tiên ở quy mô thương mại |
| **2016** | Azure Functions và Google Cloud Functions ra mắt — FaaS trở thành mainstream |
| **2018** | CNCF Serverless Working Group xuất bản Serverless Whitepaper v1 — standardize khái niệm |
| **2019–2020** | AWS Lambda tích hợp sâu với IoT Core Rules Engine; Serverless IoT trở thành production-ready |
| **2022** | AWS Lambda hỗ trợ container images; cold start được cải thiện đáng kể với SnapStart |
| **2024** | Datadog State of Serverless: 70%+ tổ chức dùng serverless trong production; IoT là top use case |
| **2025** | Graviton3-powered Lambda, reduced cold start; Azure Functions v4 với durable patterns mới |

*(Nguồn: [CNCF Serverless Whitepaper v2](https://github.com/cncf/wg-serverless), [Datadog State of Serverless 2024](https://www.datadoghq.com/state-of-serverless/))*

### 1.3 Vị trí trong Hệ sinh thái Kiến trúc IoT

Trong bản đồ 5 pattern kiến trúc IoT (xem [02-patterns.md](02-patterns.md)), Serverless chiếm vị trí độc đáo:

```
                        Độ phức tạp vận hành
         Thấp ◄────────────────────────────────► Cao
          │                                        │
Ít        │  Layered      Serverless               │  Edge-Cloud
thiết bị  │  Monolith  ←→ Architecture             │  Hybrid
          │                    ↕                   │
          │              Microservices              │
          │                 + EDA                  │
Nhiều     │                                        │
thiết bị  └────────────────────────────────────────┘
```

Serverless là lựa chọn duy nhất có thể **vừa đơn giản về vận hành** (giống Layered) **vừa elastic về scale** (giống Microservices + EDA). Điều này khiến nó trở thành entry point lý tưởng cho các team IoT nhỏ và các workload có đặc tính bursty.

### 1.4 Sự Căn chỉnh Tự nhiên với IoT: Bursty Traffic Gặp Pay-per-Invocation

Phần lớn workload IoT **không liên tục** — chúng có tính chất bursty và không đều:

- Fleet 10,000 cảm biến đỗ xe chỉ gửi sự kiện khi xe đến/đi, không phải mỗi giây
- Cảm biến nhiệt độ xe tải lạnh gửi 1 alert/giờ bình thường, rồi 50 alert/phút khi tủ hỏng
- Dịch vụ provisioning thiết bị xử lý 500 device lúc ra mắt sản phẩm, sau đó gần như zero

Đặc tính này căn chỉnh trực tiếp với mô hình giá serverless:

| Trạng thái | Container luôn bật | Serverless |
|---|---|---|
| Zero traffic | Vẫn tốn $30–80/tháng/instance | **$0** |
| Bình thường (100 req/s) | Chi phí cố định | Tính theo invocation |
| Peak (10,000 req/s) | Cần pre-scale hoặc bị quá tải | Auto-scale tức thì |
| Sau peak | Vẫn tốn chi phí idle | Scale về 0 tự động |

> **Insight cốt lõi:** Serverless nổi lên không phải vì sự tiện lợi cho developer — mà vì **hiệu quả chi phí dưới tải không đồng đều**. Đây chính xác là đặc điểm của IoT telemetry.
>
> *(Nguồn: [AWS Lambda documentation](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html))*

---

## 2. Topology

### 2.1 FaaS Topology — Lớp Compute

Lớp FaaS (Function as a Service) là trung tâm xử lý logic trong kiến trúc serverless IoT. Mỗi sự kiện từ thiết bị kích hoạt một hàm stateless, ephemeral (tồn tại trong thời gian thực thi):

```
  DEVICE TIER                    FAAS LAYER                    OUTPUTS
  ┌─────────────┐                                           ┌─────────────┐
  │  Sensor A   │──── MQTT ────▶┌──────────────────────┐   │  DynamoDB   │
  │ (temp, hum) │               │                      │──▶│  (telemetry)│
  └─────────────┘               │   IoT Broker /       │   └─────────────┘
                                │   Rules Engine       │
  ┌─────────────┐               │                      │   ┌─────────────┐
  │  Sensor B   │──── MQTT ────▶│   [SQL-like filter]  │──▶│  SNS / SES  │
  │ (pressure)  │               │        │             │   │  (alerts)   │
  └─────────────┘               │        │ trigger     │   └─────────────┘
                                │        ▼             │
  ┌─────────────┐               │  ┌─────────────┐    │   ┌─────────────┐
  │  Gateway    │──── HTTP ────▶│  │   Function  │    │──▶│  S3 / Blob  │
  │ (batch)     │               │  │  (stateless │    │   │  (archive)  │
  └─────────────┘               │  │  ephemeral) │    │   └─────────────┘
                                │  └─────────────┘    │
                                └──────────────────────┘
                                      FaaS Layer
```

**Đặc điểm cốt lõi của FaaS:**
- **Stateless**: Hàm không lưu trạng thái giữa các lần gọi
- **Ephemeral**: Môi trường thực thi tồn tại chỉ trong thời gian xử lý (và đôi khi được tái sử dụng — xem §3.3)
- **Event-triggered**: Không có long-running process; hàm chỉ chạy khi có sự kiện
- **Billed per invocation**: Tính tiền theo số lần gọi và GB-giây thực thi

### 2.2 BaaS Topology — Lớp Managed Services

BaaS là các dịch vụ managed thay thế hạ tầng tự vận hành. Một stack serverless IoT hoàn chỉnh không có server nào do team vận hành:

```
┌─────────────────────────────────────────────────────────────────┐
│                         BAAS LAYER                              │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐  ┌────────────────┐  │
│  │  IoT Broker     │  │  Data Store      │  │  Notification  │  │
│  │                 │  │                  │  │                │  │
│  │ AWS IoT Core    │  │ DynamoDB         │  │ SNS / SES /    │  │
│  │ Azure IoT Hub   │  │ Cosmos DB        │  │ Event Grid /   │  │
│  │ GCP IoT Core    │  │ Firestore        │  │ Pub/Sub        │  │
│  │                 │  │                  │  │                │  │
│  │ (Managed MQTT   │  │ (Serverless,     │  │ (Fan-out to    │  │
│  │  broker: 100M+  │  │  on-demand       │  │  SMS, email,   │  │
│  │  connections)   │  │  capacity)       │  │  mobile push)  │  │
│  └────────┬────────┘  └────────┬─────────┘  └───────┬────────┘  │
│           │ trigger            │ read/write          │ publish   │
├───────────┼────────────────────┼─────────────────────┼──────────┤
│           │              FAAS LAYER                  │          │
│           ▼                                          │          │
│  ┌─────────────────────────────────────────────┐    │          │
│  │            Lambda / Azure Function           │────┘          │
│  │       (stateless, event-triggered)           │               │
│  └──────────────────────┬──────────────────────┘               │
│                         │                                       │
│  ┌──────────────────────▼──────────────────────┐               │
│  │          Orchestration (optional)            │               │
│  │  Step Functions / Durable Functions          │               │
│  │  (stateful workflows: OTA, provisioning)     │               │
│  └─────────────────────────────────────────────┘               │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐                     │
│  │  Secrets Mgmt   │  │  API Gateway     │                     │
│  │                 │  │                  │                     │
│  │ Secrets Manager │  │ API GW / APIM    │                     │
│  │ Key Vault       │  │ (sync HTTP path) │                     │
│  └─────────────────┘  └──────────────────┘                     │
└─────────────────────────────────────────────────────────────────┘
```

### 2.3 Full Serverless IoT Topology — Kết hợp FaaS + BaaS

Topology đầy đủ thể hiện cả **async path** (telemetry) và **sync path** (provisioning/query):

```
  DEVICES                    CLOUD — FULL SERVERLESS IOT TOPOLOGY
  ┌──────────┐
  │ Sensor   │──MQTT──┐
  │ (1–10M)  │        │   ┌──────────────────────────────────────────────────┐
  └──────────┘        │   │  IoT Broker (Managed)                            │
                      ├──▶│                                                  │
  ┌──────────┐        │   │  Rules Engine ── SQL filter ──┬─────────────────┐│
  │ Gateway  │──MQTT──┘   │                               │                 ││
  │          │            └───────────────────────────────┼─────────────────┘│
  └──────────┘                                            │
       │                          ┌─────────────────────────────────────────┐
       │ HTTP (sync)               │                ASYNC PATH               │
       │                          │                                         │
       ▼                          │  ┌──────────────────────────────────┐   │
  ┌──────────────┐                │  │  Lambda: telemetry-processor     │   │
  │  API Gateway │                │  │  - validate & enrich             │   │
  │  (REST/HTTP) │                │  │  - detect anomalies              │   │
  └──────┬───────┘                │  │  - route to outputs              │   │
         │                        │  └──────────┬───────────────────────┘   │
         ▼                        │             │                           │
  ┌──────────────┐                │    ┌────────┴────────┐                  │
  │  Lambda:     │                │    │                 │                  │
  │  provisioner │                │  DynamoDB          SNS                 │
  │  or query    │                │  (device state)   (alerts)             │
  └──────┬───────┘                └─────────────────────────────────────────┘
         │
         ▼
  ┌──────────────┐          ┌─────────────────────────────────────────┐
  │ Step Functions│         │               LIFECYCLE PATH            │
  │ State Machine │         │                                         │
  │ (OTA/Provison)│         │  Provisioning → Cert generation →       │
  └───────────────┘         │  Shadow creation → Health check         │
                            └─────────────────────────────────────────┘
```

### 2.4 Các Thành phần Cốt lõi

| Thành phần | Vai trò | Managed Service tiêu biểu |
|---|---|---|
| **IoT Broker** | Managed MQTT broker; xử lý hàng triệu kết nối thiết bị đồng thời | AWS IoT Core, Azure IoT Hub |
| **Rules Engine** | Lọc và route MQTT message bằng SQL-like syntax; không cần code | IoT Core Rules, IoT Hub Message Routing |
| **Function Runtime** | Môi trường thực thi stateless; auto-scale, billed per invocation | AWS Lambda, Azure Functions, GCF |
| **Managed Data Store** | NoSQL serverless; on-demand capacity, không cần quản lý cluster | DynamoDB, Cosmos DB, Firestore |
| **Notification Bus** | Fan-out sự kiện đến SMS, email, HTTP, Lambda, SQS | SNS, Event Grid, Pub/Sub |
| **Orchestration Engine** | Stateful workflow cho multi-step process (OTA, provisioning) | Step Functions, Durable Functions |
| **API Gateway** | Sync HTTP entry point cho device provisioning và query | API Gateway, Azure APIM |
| **Secrets Manager** | Lưu trữ mã hóa cho keys, certificates, API tokens | Secrets Manager, Key Vault |

---

## 3. Cơ chế Hoạt động

### 3.1 Vòng đời Invocation — Từ Device Message đến Function Response

```
   Device                IoT Broker              Function Runtime
     │                       │                         │
     │── MQTT PUBLISH ───────▶│                         │
     │                       │                         │
     │               [Rules Engine filter]             │
     │               SELECT *, topic(2) as id          │
     │               FROM 'factory/+/temp'             │
     │               WHERE temperature > 90            │
     │                       │                         │
     │                       │── Trigger ─────────────▶│
     │                       │                    [Cold start? Init env]
     │                       │                    [Warm? Reuse container]
     │                       │                         │
     │                       │                   [Execute handler]
     │                       │                   1. Parse event
     │                       │                   2. Validate payload
     │                       │                   3. Write to DynamoDB
     │                       │                   4. Publish to SNS
     │                       │                         │
     │                       │◀── Response (async) ────│
     │                       │    (invocation ID,      │
     │                       │     success/error)      │
```

Với **async invocation** (telemetry processing): device không chờ kết quả. Broker gửi message đến Lambda queue; Lambda xử lý best-effort với retry tự động.

Với **sync invocation** (provisioning qua API Gateway): device gọi HTTP, chờ response trong timeout window.

### 3.2 Ba Mô hình Invocation trong IoT

| Mô hình | Cơ chế | Ví dụ IoT | Đặc điểm |
|---|---|---|---|
| **Async (Event-driven)** | IoT Core Rule → Lambda queue → Function | Telemetry processing, alert generation | Không block device; retry tự động 2 lần; DLQ cho failures |
| **Sync (HTTP)** | Device → API Gateway → Lambda → Response | Device provisioning, shadow read/write, command ACK | Device chờ response; timeout 29s (API GW); không retry tự động |
| **Streaming (Poll)** | Lambda polls Kinesis/SQS → batch invocation | High-volume telemetry, ordered processing | Polling interval 100–300ms thêm latency; batching tăng throughput |

### 3.3 Cold Start Lifecycle — Nguyên nhân và Cơ chế

Cold start xảy ra khi **không có execution environment nào sẵn sàng** để xử lý invocation:

```
  COLD START (lần đầu hoặc sau thời gian không dùng)
  ├─── Init Phase ──────────────────────── 100ms – 3s
  │    ├── Download function code/container
  │    ├── Khởi động runtime (JVM, Python interpreter, Node.js)
  │    ├── Khởi tạo SDK clients (DynamoDB, IoT Core)
  │    └── Chạy initialization code (ngoài handler)
  │
  └─── Invoke Phase ─────────────────────  10ms – 500ms
       └── Chạy handler function

  WARM START (execution env được tái sử dụng)
  └─── Invoke Phase only ────────────────  1ms – 50ms
       └── Chạy handler function (env đã sẵn sàng)
```

**Cold start theo ngôn ngữ runtime** (P50 median, AWS Lambda 512MB):

| Runtime | Cold Start P50 | Cold Start P99 | Ghi chú |
|---|---|---|---|
| **Go / Rust** | ~50ms | ~150ms | Compiled binary; tốt nhất cho IoT latency-sensitive |
| **Python 3.12** | ~100ms | ~400ms | Interpreter start + import; phổ biến nhất |
| **Node.js 20** | ~120ms | ~350ms | V8 startup; tốt cho lightweight handlers |
| **Java 21 (SnapStart)** | ~200ms | ~600ms | Snapshots pre-warmed; cải thiện từ 3s xuống 200ms |
| **Java 21 (không SnapStart)** | ~1,500ms | ~3,000ms | JVM startup; không dùng cho IoT real-time |

*(Nguồn: [Mikhail Shilkov — AWS Lambda Cold Start Performance](https://mikhail.io/serverless/coldstarts/aws/), [Datadog State of Serverless 2024](https://www.datadoghq.com/state-of-serverless/))*

### 3.4 Concurrency Model và Công thức Tính Toán

**Công thức cốt lõi:**

```
Concurrent Executions = (Events/giây) × (Avg execution duration in giây)
```

**Ví dụ với fleet IoT điển hình:**

```
Fleet: 10,000 devices, mỗi device 1 message/giây, Lambda duration = 50ms

Concurrency = 10,000 × 0.05 = 500 concurrent executions

AWS default limit: 1,000/region/account
→ Fleet 20,000 devices với profile này HIT giới hạn → throttling
→ Giải pháp: Request limit increase hoặc dùng reserved concurrency
```

| Fleet Size | Events/giây | Lambda Duration | Concurrency cần | AWS Default Limit |
|---|---|---|---|---|
| 1,000 devices | 1,000/s | 50ms | **50** | 1,000 ✅ |
| 10,000 devices | 10,000/s | 50ms | **500** | 1,000 ✅ |
| 20,000 devices | 20,000/s | 50ms | **1,000** | 1,000 ⚠️ Borderline |
| 100,000 devices | 100,000/s | 50ms | **5,000** | 1,000 ❌ Throttled |

*(Nguồn: [AWS Lambda concurrency docs](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html))*

### 3.5 Retry Semantics và Idempotency

| Invocation Model | Retry Policy | Idempotency Requirement |
|---|---|---|
| **Async** | 2 lần tự động; sau đó DLQ | **Bắt buộc** — Lambda có thể gọi 2–3 lần cho cùng 1 event |
| **Sync** | Không retry tự động — caller quyết định | Nên có — caller có thể retry |
| **Stream (SQS)** | Retry đến max receive count; sau đó DLQ | **Bắt buộc** — message có thể được deliver nhiều lần |

**Pattern idempotency cho IoT:** Dùng `messageId` hoặc `deviceId + timestamp` làm idempotency key; check existence trong DynamoDB trước khi xử lý:

```python
# Idempotency check pattern
def handler(event, context):
    message_id = event['messageId']
    # Conditional write — chỉ insert nếu không tồn tại
    try:
        table.put_item(
            Item={'id': message_id, 'data': event},
            ConditionExpression='attribute_not_exists(id)'
        )
    except ConditionalCheckFailedException:
        return  # Duplicate — bỏ qua
```

---

## 4. Đánh giá Architecture Characteristics

> **Phương pháp đánh giá:** Theo framework của Richards & Ford trong *Fundamentals of Software Architecture*,
> mỗi architecture style được đánh giá trên các dimensions chính bằng thang ★☆☆☆☆ (1–5 sao).
> Đánh giá dưới đây áp dụng đặc thù cho **IoT context** — không phải serverless tổng quát.

### 4.1 Bảng Đánh giá Tổng hợp

| Architecture Characteristic | Rating | Giải thích cho IoT Context |
|---|---|---|
| **Scalability** | ★★★★★ | Scale từ 0 đến hàng triệu tự động, không cần capacity planning. Fleet từ 100 lên 1,000,000 devices không thay đổi cấu hình. |
| **Elasticity** | ★★★★★ | Scale-to-zero trong thời gian idle; spike lên 10,000× trong vài giây. Đây là đặc tính mạnh nhất của serverless. |
| **Fault Tolerance** | ★★★☆☆ | Provider quản lý infra với SLA 99.95%. Tuy nhiên: cold start timeout, function crash, DLQ overflow là failure modes thực tế. |
| **Performance** | ★★☆☆☆ | Cold start 100ms–3s không chấp nhận được cho control paths < 100ms. Warm start < 10ms rất tốt. **Không thể đảm bảo P99 latency.** |
| **Cost Efficiency** | ★★★★☆ | Xuất sắc cho workload sporadic/bursty. Trở nên đắt hơn containers ở mức > ~100 req/s liên tục. |
| **Security** | ★★★☆☆ | IAM per-function granular rất tốt. Shared execution environment (multi-tenant) tạo rủi ro side-channel nhỏ. |
| **Observability** | ★★★☆☆ | Provider metrics và logs tốt. Distributed tracing qua nhiều functions phức tạp; cold start không xuất hiện trong standard metrics. |
| **Maintainability** | ★★★★☆ | Functions nhỏ, single-purpose dễ test và maintain. Rủi ro "function proliferation" nếu không có governance. |
| **Reliability** | ★★★★☆ | Provider SLA ~99.95%. Retry built-in cho async. Nhưng: throttling dưới concurrent limit tạo reliability issue ở IoT spikes. |
| **Deployability** | ★★★★★ | ZIP/container upload; không cần quản lý infra. CI/CD đơn giản hơn container cluster. Blue/green deployment tự động. |
| **Interoperability** | ★★★☆☆ | Standard HTTP/event interfaces. Nhưng: IoT Rules Engine SQL, trigger configurations là vendor-specific. |
| **Offline Resilience** | ★☆☆☆☆ | **Không hỗ trợ.** Serverless là pure-cloud; nếu IoT system cần offline operation, phải dùng Edge-Cloud Hybrid. |
| **Bandwidth Efficiency** | ★★☆☆☆ | Không xử lý tại edge; 100% data phải lên cloud. Với fleet lớn + high-frequency telemetry, chi phí data transfer tăng mạnh. |
| **Energy Efficiency** | ★★★★★ | Scale-to-zero loại bỏ hoàn toàn idle compute. Từ góc nhìn carbon footprint, serverless là hiệu quả nhất trong các patterns. |

### 4.2 Phân tích Chi tiết theo Dimension Quan trọng nhất

#### Scalability & Elasticity ★★★★★

Scalability là điểm mạnh tuyệt đối của Serverless trong IoT. Hai chiều scale độc lập:

```
Chiều 1: Concurrent connections (IoT broker scale)
  AWS IoT Core: 500M+ connections được publish
  Azure IoT Hub: Từ 400K đến 400M devices

Chiều 2: Processing throughput (Lambda scale)
  Lambda: 1,000 concurrent → unlimited với limit increase
  Azure Functions: 200 → unlimited (Premium tier)
```

Khác với container cluster cần 5–15 phút để scale out, Lambda scale trong **vài giây** — phù hợp với spike bất ngờ của IoT (ví dụ: 10,000 devices reconnect đồng thời sau network outage).

#### Performance ★★☆☆☆

Đây là điểm yếu lớn nhất. Serverless **không thể đảm bảo P99 latency** do cold start:

```
Latency distribution cho IoT Lambda function điển hình:
  P50 (median):   ~15ms  (warm start)
  P95:            ~25ms  (warm start, một số cold start)
  P99:           ~800ms  (cold start)
  P99.9:        ~2,500ms (cold start với heavy runtime)
```

**Kết luận:** Serverless phù hợp cho paths có **latency tolerance > 500ms** (alerts, notifications, analytics). Không dùng cho safety control commands (< 10ms) hay real-time display (< 100ms).

#### Cost Efficiency ★★★★☆

Chi tiết phân tích cost tại §7.1. Quy tắc đơn giản:

```
Break-even point: ~100 request/giây sustained

Dưới 100 req/s  → Serverless rẻ hơn container
Trên 100 req/s  → Container (ECS, Kubernetes) rẻ hơn
```

#### Offline Resilience ★☆☆☆☆

Đây là mismatch cơ bản nhất với một số use case IoT. Serverless là **pure-cloud architecture**: không có component nào chạy gần thiết bị. Nếu cloud không đạt được:

- Functions không thể invoke
- IoT broker không nhận message
- Không có local decision making

**Pattern phổ biến để work around:** Kết hợp Serverless (cloud) với thin edge gateway có local buffer — nhưng lúc đó đã không còn là pure serverless.

---

## 5. Các Biến thể Kiến trúc

Serverless IoT không phải một topology duy nhất. Có **4 biến thể chính**, mỗi biến thể phù hợp với một loại workload khác nhau.

---

### Pattern 1: Event-Triggered Telemetry Processing

**Mô tả:** Device gửi telemetry qua MQTT → IoT Broker lọc bằng Rules Engine → Lambda xử lý, lưu trữ, và phát alert khi cần.

**Topology:**

```
  Devices (1–10K)
  ┌────────────┐     MQTT      ┌─────────────────────────────────────┐
  │ Temp/Humid │──────────────▶│  IoT Core / IoT Hub                 │
  │ Sensors    │               │                                     │
  └────────────┘               │  Rules Engine:                      │
                               │  SELECT * FROM 'sensors/+/telemetry'│
  ┌────────────┐     MQTT      │  WHERE temperature > 85             │
  │ Pressure   │──────────────▶│         OR humidity < 20            │
  │ Sensors    │               └──────────────────┬──────────────────┘
  └────────────┘                                  │ Trigger (async)
                                                  ▼
                               ┌──────────────────────────────────────┐
                               │  Lambda: telemetry-processor         │
                               │                                      │
                               │  1. Parse & validate                 │
                               │  2. Enrich (device metadata from DB) │
                               │  3. Anomaly check                    │
                               │  4. Write to DynamoDB                │
                               │  5. If anomaly → SNS.publish()       │
                               └──────────────────┬───────────────────┘
                                        │                  │
                                        ▼                  ▼
                               ┌────────────────┐  ┌──────────────────┐
                               │  DynamoDB      │  │  SNS             │
                               │  (telemetry    │  │  ├─ Email        │
                               │   time-series) │  │  ├─ SMS          │
                               └────────────────┘  │  └─ Mobile Push  │
                                                   └──────────────────┘
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Giải thích |
|---|---|---|
| **Độ phù hợp** | ✅ Xuất sắc | Archetype lý tưởng cho sporadic IoT alerting |
| **Cost** | ✅ Thấp | Chi phí gần 0 khi không có anomaly |
| **Latency alert** | ⚠️ 200ms–2s | Chấp nhận được cho notification; không dùng cho safety control |
| **Scale** | ✅ Auto | Rules Engine + Lambda scale tự động |
| **Operability** | ✅ Thấp | Không cần quản lý broker, consumer, hay cluster |
| **Trạng thái** | ⚠️ Cần external | "Đã gửi alert chưa?" cần check DynamoDB mỗi invocation |

**Khi dùng Pattern này:**
- Workload sporadic: < 1,000 events/giây bình thường
- Latency requirement > 200ms cho alert path
- Team nhỏ (1–3 người) cần ship nhanh
- Budget sensitive — muốn zero cost khi hệ thống idle

---

### Pattern 2: Device Lifecycle Orchestration

**Mô tả:** Các workflow nhiều bước (provisioning mới, OTA update, decommission) được orchestrate bởi Step Functions / Durable Functions. Lambda là các execution steps trong workflow.

**Topology:**

```
  Admin / Provisioning Service           Orchestration Engine
  ┌──────────────────────┐               ┌────────────────────────────────┐
  │ POST /devices/       │──── HTTP ────▶│  API Gateway                   │
  │ {deviceId, model...} │               └──────────────┬─────────────────┘
  └──────────────────────┘                              │ sync invoke
                                                        ▼
                                          ┌────────────────────────┐
                                          │  Lambda: orchestrator  │
                                          │  StartExecution →      │
                                          └───────────┬────────────┘
                                                      │
                                          ┌───────────▼────────────────────────────┐
                                          │  Step Functions State Machine          │
                                          │                                        │
                                          │  [1] ValidateDevice                    │
                                          │       └── Lambda: validate-device      │
                                          │  [2] GenerateCertificate               │
                                          │       └── Lambda: cert-generator       │
                                          │  [3] RegisterInIoTCore                 │
                                          │       └── Lambda: iot-registrar        │
                                          │  [4] CreateDeviceShadow               │
                                          │       └── Lambda: shadow-creator       │
                                          │  [5] HealthCheck (wait 30s)           │
                                          │       └── Lambda: health-checker       │
                                          │  [6] NotifySuccess                     │
                                          │       └── Lambda: notifier             │
                                          │                                        │
                                          │  [ERROR] → CompensateAndRollback       │
                                          └────────────────────────────────────────┘
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Giải thích |
|---|---|---|
| **Workflow phức tạp** | ✅ Xuất sắc | Step Functions xử lý branching, retry, compensation tự động |
| **Execution time** | ✅ OK | Workflow có thể chạy nhiều giờ (Step Functions không bị 15-min limit của Lambda) |
| **Cost** | ⚠️ Cần xem xét | Step Functions tính tiền theo state transition ($0.025/1000 transitions) |
| **Debugging** | ✅ Tốt | Visual execution history trong AWS Console |
| **Vendor lock-in** | ❌ Cao | Step Functions / Durable Functions là highly vendor-specific |
| **Complexity** | ⚠️ Trung bình | State machine definition cần học cú pháp riêng (ASL / JSON) |

**Khi dùng Pattern này:**
- Multi-step workflows có compensation logic (rollback nếu step thất bại)
- Long-running processes: OTA rollout cho 10,000 devices có thể mất nhiều giờ
- Audit trail quan trọng: mọi state transition được log đầy đủ
- Cần human approval step trong workflow

---

### Pattern 3: Scheduled Batch Aggregation

**Mô tả:** Lambda được kích hoạt theo lịch (EventBridge Scheduler / Timer trigger) để tổng hợp dữ liệu từ devices, tạo báo cáo, hoặc thực hiện fleet-wide health checks.

**Topology:**

```
  EventBridge Scheduler         Lambda                 Outputs
  ┌────────────────────┐        ┌──────────────────┐
  │  Every 1 hour      │───────▶│  Lambda:         │──── Aggregate ──▶ DynamoDB
  │  "0 * * * ? *"     │        │  fleet-aggregator│                   (hourly summaries)
  └────────────────────┘        │                  │
                                │  1. Scan devices │──── Export ─────▶ S3
  ┌────────────────────┐        │  2. Compute stats│                   (CSV/Parquet)
  │  Every 24 hours    │───────▶│  3. Detect fleet │
  │  "0 8 * * ? *"     │        │     anomalies    │──── Alert ──────▶ SNS
  └────────────────────┘        │  4. Generate     │                   (fleet health)
                                │     report       │
  ┌────────────────────┐        │  5. Archive to   │──── Trigger ────▶ Lambda
  │  Every 5 min       │───────▶│     S3           │                   (downstream analytics)
  │  (OTA status check)│        └──────────────────┘
  └────────────────────┘
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Giải thích |
|---|---|---|
| **Simplicity** | ✅ Cao | Không cần streaming infrastructure |
| **Cost** | ✅ Rất thấp | Chỉ tính tiền lúc chạy (~minutes/giờ) |
| **Timeliness** | ❌ Thấp | Dữ liệu cũ tối đa = schedule interval (1 giờ, 1 ngày) |
| **Memory** | ⚠️ Chú ý | Aggregating large datasets cần memory lớn; Lambda max 10GB |
| **Idempotency** | ✅ Dễ | Scheduled tasks có thể re-run an toàn nếu thiết kế đúng |
| **Execution limit** | ⚠️ Chú ý | Lambda 15-min limit; nếu aggregation lớn hơn, dùng Fargate |

**Khi dùng Pattern này:**
- Reporting và analytics không cần real-time
- Billing aggregation, fleet health digest
- Dữ liệu cần batch export sang data warehouse (S3 → Athena, BigQuery)
- Cost-sensitive: chỉ muốn pay khi cần tính toán

---

### Pattern 4: Hybrid Serverless-Streaming

**Mô tả:** Kết hợp managed streaming service (Kinesis, Event Hubs) để buffer high-volume telemetry với Lambda làm stream consumer. Lambda sử dụng **event source mapping** để xử lý batches từ stream.

**Topology:**

```
  Devices (10K–1M)
  ┌───────────────┐
  │ High-frequency│──MQTT──▶┌─────────────────┐    ┌──────────────────────────┐
  │ Telemetry     │         │  IoT Core        │───▶│  Kinesis Data Streams    │
  │ (1 msg/sec)   │         │  Rules Engine    │    │  (buffer + ordering)     │
  └───────────────┘         └─────────────────┘    │                          │
                                                   │  Shard 1: device 1–10K  │
  ┌───────────────┐                                │  Shard 2: device 10K–20K│
  │ High-frequency│──MQTT──▶┌─────────────────┐    │  Shard N: ...           │
  │ Telemetry     │         │  IoT Core        │───▶│                          │
  │ (10 msg/sec)  │         └─────────────────┘    └──────────┬───────────────┘
  └───────────────┘                                           │ Event Source
                                                              │ Mapping
                                                   ┌──────────▼───────────────┐
                                                   │  Lambda: stream-consumer  │
                                                   │                           │
                                                   │  Batch size: 100–1000     │
                                                   │  Tumbling window: 30s     │
                                                   │  Bisect on error: enabled │
                                                   │                           │
                                                   │  1. Deserialize batch     │
                                                   │  2. Validate records      │
                                                   │  3. Aggregate metrics     │
                                                   │  4. Bulk write to DB      │
                                                   └──────────┬────────────────┘
                                                              │
                                             ┌────────────────┼────────────────┐
                                             ▼                ▼                ▼
                                      TimescaleDB /       S3 Data Lake    Anomaly
                                      InfluxDB            (Parquet)       Alert SNS
                                      (time-series)
```

**Trade-off Analysis:**

| Dimension | Đánh giá | Giải thích |
|---|---|---|
| **Throughput** | ✅ Cao | Kinesis 1MB/s/shard; Lambda parallelization factor ×10 |
| **Ordering** | ✅ Đảm bảo | Kinesis đảm bảo order trong partition/shard |
| **Latency** | ⚠️ Trung bình | Polling interval + batch window thêm 100ms–30s |
| **Cost** | ⚠️ Trung bình | Kinesis + Lambda combined; tốn hơn pure serverless |
| **Complexity** | ⚠️ Tăng | Cần hiểu shard management, parallelization factor |
| **Backpressure** | ✅ Tốt | Kinesis buffer hấp thụ spike; Lambda không bị overwhelmed |

**Khi dùng Pattern này:**
- 10,000–1,000,000 devices với high-frequency telemetry
- Cần ordered processing (event ordering quan trọng)
- Lambda pure (Pattern 1) bắt đầu throttle hoặc quá đắt
- Vẫn muốn elastic scale, không muốn quản lý Kafka cluster

---

### So sánh 4 Patterns

| Tiêu chí | Pattern 1: Event-Triggered | Pattern 2: Lifecycle Orchestration | Pattern 3: Scheduled Batch | Pattern 4: Hybrid Streaming |
|---|---|---|---|---|
| **Tải phù hợp** | Sporadic (< 1K/s) | Very sporadic | Scheduled | High-volume (> 10K/s) |
| **Latency** | 200ms–2s | Nhiều giây – giờ | Phút – giờ | Giây (batch) |
| **Stateful?** | Không (ngoại trừ external DB) | Có (Step Functions) | Không | Không |
| **Cost (idle)** | $0 | $0 | $0 | Kinesis: ~$10/shard/tháng |
| **Operability** | Rất thấp | Thấp–trung bình | Rất thấp | Trung bình |
| **Vendor lock-in** | Trung bình | Cao | Thấp | Trung bình |

---

## 6. So sánh Cloud Provider

### 6.1 AWS Serverless IoT Stack

AWS có **stack IoT serverless trưởng thành nhất** (2025) nhờ IoT Core Rules Engine — cho phép filter/route MQTT message tại broker mà không cần code.

| Thành phần | Service AWS | Đặc điểm nổi bật |
|---|---|---|
| **IoT Broker** | AWS IoT Core | 500M+ device connections; MQTT 3.1.1/5.0; WebSocket |
| **Rules Engine** | IoT Core Rules Engine | SQL syntax; 15+ action types; không cần Lambda cho routing đơn giản |
| **FaaS** | AWS Lambda | 15 phút max; 10GB memory; 1,000+ languages via custom runtime |
| **Workflow** | AWS Step Functions | Visual workflow; Standard (history) + Express (volume); 1-year max |
| **Streaming** | Amazon Kinesis | 1MB/s/shard; 7-day retention; 1–365 day enhanced fan-out |
| **NoSQL** | Amazon DynamoDB | On-demand capacity; DAX cache; PITR backup |
| **Notifications** | Amazon SNS | 1M subscribers; email, SMS, HTTP, Lambda, SQS fan-out |
| **API Gateway** | Amazon API Gateway | REST/HTTP/WebSocket; 29s timeout; throttling per stage |
| **Secrets** | AWS Secrets Manager | Auto-rotation; audit trail; $0.40/secret/month |
| **Observability** | CloudWatch + X-Ray | Metrics, logs, distributed traces; Service Map |

**Điểm khác biệt — IoT Rules Engine SQL:**

```sql
-- Chỉ invoke Lambda khi nhiệt độ > 90°C trên sensor công nghiệp
-- Rules Engine thực hiện filter ngay tại broker — không tốn Lambda invocation
SELECT *, topic(2) as deviceId, timestamp() as receivedAt
FROM 'factory/+/sensors/temperature'
WHERE temperature > 90 AND sensorType = 'industrial'
```

*(Nguồn: [AWS IoT Rules SQL Reference](https://docs.aws.amazon.com/iot/latest/developerguide/iot-sql-reference.html))*

### 6.2 Azure Serverless IoT Stack

| Thành phần | Service Azure | Đặc điểm nổi bật |
|---|---|---|
| **IoT Broker** | Azure IoT Hub | MQTT/AMQP/HTTP; Device Twin; Cloud-to-Device messaging |
| **Event Routing** | IoT Hub Message Routing | Route theo property; endpoint: Event Hubs, Service Bus, Storage |
| **FaaS** | Azure Functions (Consumption) | 10 phút max (Consumption); unlimited (Premium/Dedicated) |
| **Workflow** | Durable Functions | Orchestrator/Activity pattern; Fan-out/Fan-in; Eternal orchestration |
| **Streaming** | Azure Event Hubs | Kafka-compatible; 1–7 day retention; Capture to ADLS |
| **NoSQL** | Azure Cosmos DB | Multi-region; 5 consistency levels; serverless capacity mode |
| **Notifications** | Azure Event Grid | 1M events/s; CloudEvents schema; 24h retry |
| **API** | Azure API Management | Developer portal; policy engine; caching |
| **Secrets** | Azure Key Vault | HSM-backed; Managed Identities integration |
| **Observability** | Application Insights + Monitor | Distributed tracing; Smart Detection alerts |

**Điểm khác biệt — Durable Functions cho IoT OTA:**

Durable Functions cho phép viết stateful orchestration bằng code thông thường (C#, Python, JS) thay vì JSON state machine — linh hoạt hơn Step Functions với developer experience tốt hơn:

```python
# Durable Functions orchestrator cho OTA rollout
@app.orchestration_trigger(context_name="context")
def ota_rollout_orchestrator(context):
    devices = yield context.call_activity('get_target_devices', input_)
    # Fan-out: update tất cả devices song song
    tasks = [context.call_activity('update_device', d) for d in devices[:100]]
    results = yield context.task_all(tasks)
    # Wait for health check
    yield context.create_timer(context.current_utc_datetime + timedelta(minutes=30))
    return yield context.call_activity('verify_rollout', results)
```

### 6.3 GCP Serverless IoT Stack

| Thành phần | Service GCP | Đặc điểm nổi bật |
|---|---|---|
| **IoT Broker** | Cloud IoT Core (deprecated 2023) → Pub/Sub trực tiếp | ⚠️ GCP khai tử IoT Core 2023; cần dùng bên thứ 3 (HiveMQ, EMQX) hoặc trực tiếp Pub/Sub |
| **FaaS** | Cloud Functions (Gen2) | 60 phút max; Cloud Run backend |
| **Workflow** | Cloud Workflows | YAML/JSON; không có Gen1 Step Functions equivalent |
| **Streaming** | Cloud Pub/Sub + Dataflow | Pub/Sub: unlimited throughput; Dataflow: Apache Beam managed |
| **NoSQL** | Cloud Firestore / Bigtable | Firestore: serverless; Bigtable: managed, high-throughput |
| **API** | Cloud Endpoints / API Gateway | gRPC + HTTP/JSON transcoding |
| **Observability** | Cloud Trace + Cloud Monitoring | OpenTelemetry native support |

**Lưu ý quan trọng về GCP IoT:** Google Cloud IoT Core đã bị **khai tử tháng 8/2023**. Các project IoT trên GCP hiện cần sử dụng Pub/Sub làm MQTT broker qua bên thứ ba, hoặc migrate sang AWS/Azure. Đây là rủi ro vendor lock-in nghiêm trọng.

### 6.4 Bảng So sánh Tổng hợp

| Tiêu chí | AWS | Azure | GCP |
|---|---|---|---|
| **IoT Broker tích hợp** | ✅ IoT Core (native) | ✅ IoT Hub (native) | ❌ IoT Core đã khai tử |
| **Rules Engine** | ✅ SQL-based, 15+ actions | ⚠️ Message routing (đơn giản hơn) | ❌ Không có native |
| **FaaS max duration** | 15 phút | 10 phút (Consump.) / unlimited (Premium) | 60 phút |
| **Default concurrency** | 1,000/region | 200 (Consumption) | 3,000/region |
| **Cold start mitigation** | SnapStart (Java), Provisioned Concurrency | Premium plan (always-on), Flex Consumption | Min instances |
| **Workflow orchestration** | Step Functions (visual, JSON) | Durable Functions (code-first) | Cloud Workflows (YAML) |
| **Streaming** | Kinesis | Event Hubs (Kafka-compatible) | Pub/Sub + Dataflow |
| **Serverless NoSQL** | DynamoDB (on-demand) | Cosmos DB (serverless mode) | Firestore |
| **IoT market maturity** | ★★★★★ Nhất thị trường | ★★★★☆ Tốt cho enterprise | ★★★☆☆ Mất thế mạnh IoT |
| **Vendor lock-in risk** | Cao (Rules Engine, Step Functions) | Cao (IoT Hub, Durable Functions) | Trung bình (nhưng GCP IoT không còn) |
| **Recommended for** | Default choice cho IoT serverless | Enterprise đã dùng Azure/Microsoft stack | Chỉ nếu đã có GCP data platform |

---

## 7. Phân tích Trade-offs Sâu

### 7.1 Mô hình Chi phí (Cost Model)

#### Cấu trúc Chi phí Lambda (AWS)

```
Chi phí Lambda = Chi phí Request + Chi phí Compute

Chi phí Request:  $0.20 / 1,000,000 invocations
Chi phí Compute:  $0.0000166667 / GB-second (x86)
                  $0.0000133334 / GB-second (Graviton/ARM)

Free tier:        400,000 GB-seconds/tháng
                  1,000,000 invocations/tháng
```

#### Ví dụ Tính toán: Fleet 10,000 Devices

**Kịch bản:** 10,000 sensors, 1 message/phút, Lambda 128MB, 100ms execution

```
Invocations/tháng = 10,000 × 60 × 24 × 30 = 432,000,000

Chi phí request = 432M × $0.20/1M         = $86.40
GB-seconds      = 432M × 0.1s × 0.128GB   = 5,529,600 GB-s
Chi phí compute = 5,529,600 × $0.0000167  = $92.34

TỔNG LAMBDA:     ~$178.74/tháng
```

**So sánh với ECS Fargate (container):**

```
ECS Fargate (1 vCPU, 2GB RAM) để xử lý 72,000 msg/giờ:
Chi phí Fargate = $0.04048/vCPU-hour × 720h + $0.004445/GB-hour × 2 × 720h
                = $29.15 + $6.40
                = ~$35.55/tháng (x1 task)

→ Tại workload này: Fargate ($35.55) rẻ hơn Lambda ($178.74) khoảng 5×
```

**Break-even Point:**

```
Lambda đắt hơn Fargate khi: sustained RPS > ~100 request/giây
                            (tùy Lambda duration và memory)

Cụ thể cho ví dụ trên: break-even ≈ 72,000 msg/giờ ≈ 20 msg/giây

Dưới 20 msg/giây  → Lambda rẻ hơn (hoặc $0 khi idle)
Trên 20 msg/giây  → Fargate/ECS rẻ hơn
```

**Quy tắc thực hành:**

| Workload | Serverless hay Container? | Lý do |
|---|---|---|
| < 100K events/ngày | ✅ Serverless | Cost gần $0; overhead không đáng kể |
| 100K–10M events/ngày | ⚠️ Phân tích | Phụ thuộc duration và memory; tính cụ thể |
| > 10M events/ngày sustained | ❌ Container | Break-even đã vượt qua; container rẻ hơn 2–10× |
| Spike: 0 → 1M events/giờ | ✅ Serverless | Container không kịp scale; serverless xử lý tự nhiên |

*(Nguồn: [AWS Lambda Pricing](https://aws.amazon.com/lambda/pricing/), [AWS Fargate Pricing](https://aws.amazon.com/fargate/pricing/))*

### 7.2 Cold Start — Phân tích và Giải pháp Giảm thiểu

#### Nguyên nhân Cold Start

```
Cold start xảy ra khi:
1. Function lần đầu được invoke (no existing execution env)
2. Lambda cần thêm concurrent executions (scale out)
3. Execution environment bị recycle sau ~15 phút idle
4. Function code hoặc config vừa được deploy
5. Execution environment bị recycle để cân bằng tải
```

#### Chiến lược Giảm thiểu

| Chiến lược | Hiệu quả | Chi phí | Phù hợp cho IoT |
|---|---|---|---|
| **Provisioned Concurrency** | ★★★★★ Loại bỏ cold start hoàn toàn | $0.015/GB-hour provisioned | ✅ Alert paths yêu cầu < 200ms P99 |
| **Lambda SnapStart (Java)** | ★★★★☆ Giảm cold start 10× | Không thêm phí | ✅ Java runtimes; giảm từ 3s xuống 200ms |
| **Dùng Python/Node/Go** | ★★★☆☆ Cold start ngắn hơn | Không | ✅ Lựa chọn runtime phù hợp |
| **Giảm dependency size** | ★★★☆☆ Init phase nhanh hơn | Không | ✅ Loại bỏ unused packages; tree-shaking |
| **Warm-up ping (EventBridge)** | ★★☆☆☆ Duy trì warm; không hoàn hảo | Thấp | ⚠️ Không đáng tin cậy; Lambda có thể vẫn recycle |
| **Tăng Memory** | ★★☆☆☆ CPU tăng → init nhanh hơn | Tăng compute cost | ⚠️ Ít hiệu quả hơn SnapStart |

**Quyết định:**
- Nếu cold start là vấn đề → dùng **Provisioned Concurrency** cho critical paths
- Nếu Provisioned Concurrency quá đắt → đánh giá lại xem serverless có phải đúng choice

### 7.3 Quản lý Trạng thái (State Management)

Serverless là **stateless by design** — mỗi invocation không biết invocation trước đó. Đây là một trong những thách thức lớn nhất cho IoT.

**4 Pattern để Work Around Stateless Constraint:**

**Pattern A: External State Store**
```
Lambda → DynamoDB (device state table)
       ← Read state at start of each invocation
       → Write updated state at end
```
Phù hợp: device shadow, alert deduplication, session tracking
Nhược điểm: thêm latency (~1–5ms/read) và chi phí DynamoDB

**Pattern B: Device Shadow / Device Twin**
```
AWS IoT Core Device Shadow (được managed bởi IoT Core)
Azure IoT Hub Device Twin (được managed bởi IoT Hub)
```
Phù hợp: trạng thái thiết bị (desired vs. reported state)
Ưu điểm: không cần quản lý; built-in conflict resolution; offline sync

**Pattern C: Durable Orchestration (Step Functions / Durable Functions)**
```
Step Functions State Machine → lưu execution state managed
Durable Functions → lưu orchestration state trong storage account
```
Phù hợp: multi-step workflows có state (OTA, provisioning)
Nhược điểm: chỉ dùng cho orchestration, không dùng cho per-message state

**Pattern D: Streaming Aggregation Windows**
```
Lambda event source mapping với tumbling window = 30s
→ Lambda nhận batch 30s của events
→ Tính aggregate trong memory của batch đó
→ Không cần truy vấn external state cho window aggregation
```
Phù hợp: sliding/tumbling window metrics (avg temp 30s, max pressure 1min)
Nhược điểm: window size cố định; cross-window state vẫn cần external store

### 7.4 Vendor Lock-in — Ma trận Rủi ro

| Thành phần | Mức độ Lock-in | Lý do | Mitigation |
|---|---|---|---|
| **Lambda code (Python/Node)** | Thấp | Code có thể chạy trên bất kỳ Python runtime | Giữ business logic trong pure Python/Node modules |
| **IoT Core / IoT Hub** | Cao | Device certificates, MQTT topics, shadow format vendor-specific | Abstraction layer tại device firmware |
| **Step Functions / Durable Functions** | Rất cao | ASL JSON / C# API hoàn toàn vendor-specific | Cân nhắc Temporal.io (open-source alternative) |
| **DynamoDB / Cosmos DB** | Trung bình | API vendor-specific; data có thể export | Repository pattern; abstract DB access |
| **IoT Rules Engine SQL** | Cao | SQL dialect vendor-specific | Document tất cả rules; test coverage |
| **Invocation triggers** | Trung bình | Event schema khác nhau | Adapter layer trong Lambda handler |

**Chiến lược giảm lock-in (Hexagonal Architecture):**

```
┌─────────────────────────────────────────────────────┐
│              Lambda Handler (thin adapter)           │
│  - Parse vendor-specific event format               │
│  - Call domain core                                 │
│  - Map domain output to vendor-specific response    │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│              Domain Core (vendor-neutral)            │
│  - Pure Python/TypeScript/Go                        │
│  - No AWS/Azure SDK imports                         │
│  - Testable without cloud                           │
└────────────────────────┬────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────┐
│              Repository / Port Adapters              │
│  - DeviceRepository (interface)                     │
│    ├── DynamoDBDeviceRepository (AWS impl)          │
│    └── CosmosDBDeviceRepository (Azure impl)        │
└─────────────────────────────────────────────────────┘
```

---

## 8. Bảo mật

### 8.1 IAM Least-Privilege cho IoT Lambda

Mỗi Lambda function cần **một IAM role riêng** với chỉ những permission tối thiểu:

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:PutItem",
        "dynamodb:GetItem"
      ],
      "Resource": "arn:aws:dynamodb:us-east-1:*:table/DeviceTelemetry"
    },
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:us-east-1:*:device-alerts"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

**Anti-pattern nguy hiểm:** Dùng `AWSLambdaFullAccess` hoặc `AdministratorAccess` — một function bị compromise có thể xóa toàn bộ hạ tầng.

### 8.2 Bảo mật Device-to-Cloud

| Tầng | Cơ chế | Thực hành tốt nhất |
|---|---|---|
| **Transport** | TLS 1.2+ bắt buộc | Disable TLS 1.0/1.1 tại IoT broker |
| **Authentication** | mTLS (client certificate) | Mỗi device có certificate riêng; không dùng shared credentials |
| **Authorization** | IoT Policy per device/group | Policy granular: device chỉ có thể publish lên topic của chính nó |
| **Certificate lifecycle** | Auto-rotation | Certificate expiry monitoring; OTA cert renewal |
| **Device identity** | Hardware Security Module (HSM) | Lưu private key trong HSM; không expose trong firmware |

**IoT Policy mẫu (AWS):**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:*:*:topic/devices/${iot:ClientId}/telemetry"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Subscribe",
      "Resource": "arn:aws:iot:*:*:topicfilter/devices/${iot:ClientId}/commands"
    }
  ]
}
```

Device chỉ được publish lên topic của **chính nó** — không thể impersonate device khác.

### 8.3 Bảo mật Function Code

| Rủi ro | Giải pháp |
|---|---|
| **Dependency vulnerabilities** | Dependency scanning (Snyk, Dependabot); lock file; regular updates |
| **Secret trong code** | Tuyệt đối không hardcode; dùng Secrets Manager / Key Vault |
| **Malformed device input** | Input validation tại đầu handler; reject trước khi xử lý |
| **Injection via device payload** | Sanitize tất cả string input trước khi dùng trong SQL/NoSQL query |
| **DLQ monitoring** | Alert ngay khi có message vào DLQ — có thể là tấn công hoặc malformed data |

### 8.4 Bảo mật Shared Infrastructure

Serverless chạy trên **multi-tenant infrastructure**. AWS/Azure thực hiện isolation qua:
- Hardware virtualization (Firecracker microVM cho Lambda)
- Mỗi execution environment là một microVM riêng biệt
- Memory không được chia sẻ giữa customers

Rủi ro thực tế: **Side-channel attacks** qua shared CPU cache — rất khó exploit, nhưng tồn tại về lý thuyết. Với IoT data bình thường, rủi ro này chấp nhận được.

*(Nguồn: [AWS Lambda Security Overview](https://docs.aws.amazon.com/lambda/latest/dg/lambda-security.html), [Firecracker paper — NSDI 2020](https://www.usenix.org/conference/nsdi20/presentation/agache))*

---

## 9. Observability

### 9.1 Ba Pillars of Observability trong Serverless IoT

| Pillar | AWS | Azure | Mục đích |
|---|---|---|---|
| **Metrics** | CloudWatch Metrics | Azure Monitor Metrics | Health, performance, capacity |
| **Logs** | CloudWatch Logs | Log Analytics Workspace | Debug, audit, forensics |
| **Traces** | AWS X-Ray | Application Insights | Latency breakdown, dependency map |

### 9.2 Key Metrics Cho IoT Lambda

| Metric | Ngưỡng Alert | Ý nghĩa |
|---|---|---|
| **Invocations** | Baseline ±3σ | Volume bất thường (DDoS, fleet issue) |
| **Duration P99** | > 5,000ms | Cold start hoặc downstream timeout |
| **Error rate** | > 1% | Function crash; malformed payload |
| **Throttles** | > 0 | Concurrency limit đã đạt; cần tăng limit |
| **ConcurrentExecutions** | > 80% of limit | Sắp hit concurrency limit |
| **Dead Letter Queue** | > 0 messages | Xử lý thất bại; cần investigation ngay |
| **Iterator age (Kinesis)** | > 60,000ms | Lambda không xử lý kịp stream; lag tăng |

### 9.3 Distributed Tracing qua IoT Pipeline

Để trace một event từ device đến alert, cần **correlation ID** được truyền qua toàn bộ pipeline:

```
Device Message → IoT Core → Lambda(process) → DynamoDB → Lambda(alert) → SNS
     │                           │                              │
     └──────── correlationId ────┴──────────────────────────────┘
               = deviceId + timestamp (unique per message)
```

**Structured logging pattern cho IoT Lambda:**

```python
import json
import logging
logger = logging.getLogger()
logger.setLevel(logging.INFO)

def handler(event, context):
    # Extract correlation context
    device_id = event.get('deviceId', 'unknown')
    message_id = event.get('messageId', context.aws_request_id)

    logger.info(json.dumps({
        'level': 'INFO',
        'event': 'telemetry_received',
        'deviceId': device_id,
        'messageId': message_id,
        'temperature': event.get('temperature'),
        'lambdaRequestId': context.aws_request_id,
        'coldStart': _is_cold_start  # global flag set at init
    }))
```

Logs có structured JSON format có thể được query bằng CloudWatch Logs Insights:

```sql
-- Query: tất cả cold start trong 1 giờ qua
fields @timestamp, deviceId, lambdaRequestId
| filter coldStart = true
| sort @timestamp desc
| limit 100
```

---

## 10. Triển khai Thực tế

### Case Study 1: iRobot — Smart Home Device Fleet trên AWS IoT Core + Lambda

**Bối cảnh:**
iRobot (Roomba, Braava) quản lý hàng chục triệu robot hút bụi và lau sàn kết nối cloud. Devices gửi telemetry về trạng thái (pin, bản đồ, lịch hẹn) và nhận lệnh điều khiển.

**Kiến trúc serverless:**
```
Roomba/Braava → MQTT/TLS → AWS IoT Core → Rules Engine → Lambda (event processing)
                                                        → DynamoDB (device state)
                                                        → SNS (notifications)
                                                        → S3 (map storage)
```

**Kết quả được công bố:**
- Quản lý hàng chục triệu connected devices không cần cluster management
- Team nhỏ vận hành fleet quy mô enterprise nhờ managed services
- Auto-scale trong suốt ngày ra mắt sản phẩm mà không cần pre-provision

**Bài học architecture:**
- IoT Core Rules Engine giảm đáng kể số Lambda invocations bằng cách filter tại broker
- Device Shadow giải quyết state synchronization khi device offline/reconnect
- Per-function IAM roles quan trọng: mỗi Lambda chỉ access đúng resource cần thiết

*(Nguồn: [AWS re:Invent 2018 — iRobot IoT at Scale](https://www.youtube.com/watch?v=YjdtjWWXzg0), [AWS IoT blog](https://aws.amazon.com/blogs/iot/))*

---

### Case Study 2: Volkswagen Automotive Cloud — Vehicle Telemetry với Azure IoT Hub + Functions

**Bối cảnh:**
Volkswagen Group vận hành Volkswagen Automotive Cloud (VAC) phục vụ hàng triệu xe kết nối (Volkswagen, Audi, SEAT, Skoda). Xe gửi telemetry về vị trí, diagnostics, và driving behavior; cloud gửi lệnh (unlock, software update, navigation).

**Kiến trúc serverless:**
```
Connected Vehicles (MQTT/HTTPS)
    → Azure IoT Hub (device management, D2C/C2D)
    → Event Hubs (high-volume telemetry buffer)
    → Azure Functions (event processing, enrichment)
    → Cosmos DB (vehicle state, trip data)
    → Event Grid (notification fan-out)
    → Azure Functions (alert processing, push notification)
```

**Scale:**
- Hàng triệu xe kết nối đồng thời
- Peak: hàng triệu kết nối sau giờ cao điểm buổi chiều (commute end)
- Telemetry volume: hàng tỷ events/ngày

**Kết quả:**
- Azure IoT Hub + Functions scale tự động qua peak traffic
- Durable Functions cho OTA software update đảm bảo rollback an toàn nếu update thất bại
- Cosmos DB multi-region đảm bảo data availability theo yêu cầu GDPR (data residency EU)

**Bài học architecture:**
- Event Hubs làm buffer giữa IoT Hub và Functions — hấp thụ spike kết nối
- Durable Functions phù hợp cho OTA rollout: branching logic, retry, compensation
- Multi-region Cosmos DB là yêu cầu bắt buộc cho GDPR compliance

*(Nguồn: [Microsoft Customer Story — Volkswagen](https://customers.microsoft.com/en-us/story/1346010428222360527-volkswagen-azure-automotive), [Azure IoT blog](https://techcommunity.microsoft.com/t5/internet-of-things-blog/))*

---

### Case Study 3: Rolls-Royce — Jet Engine Health Monitoring với AWS Serverless

**Bối cảnh:**
Rolls-Royce vận hành dịch vụ TotalCare: giám sát liên tục sức khỏe động cơ phản lực trên hàng nghìn máy bay thương mại toàn cầu. Mỗi động cơ gửi hàng nghìn điểm dữ liệu mỗi giây trong chuyến bay.

**Kiến trúc serverless:**
```
Aircraft Engines (ACARS/satellite link)
    → Ingestion Layer (AWS IoT Core + Kinesis)
    → Lambda Stream Consumer (anomaly detection, data enrichment)
    → DynamoDB (engine state, maintenance schedule)
    → Step Functions (maintenance workflow: alert → engineer assignment → part order)
    → Lambda (notification) → SNS (airline operations, maintenance teams)
```

**Thách thức và giải pháp:**
- **Bursty data:** Data chỉ gửi trong chuyến bay (~8–16 giờ/ngày); serverless xử lý pattern này tự nhiên hơn container luôn bật
- **High stakes:** ML anomaly detection cần độ chính xác cao → Lambda chỉ làm data pipeline; ML inference trên SageMaker dedicated endpoint
- **Compliance:** Lưu trữ aviation maintenance records 30 năm → S3 + Glacier lifecycle policy

**Kết quả được công bố:**
- Giảm chi phí unplanned engine maintenance thông qua predictive alerts
- Serverless pipeline xử lý được volume khi nhiều chuyến bay cất cánh đồng thời

**Bài học architecture:**
- Serverless phù hợp cho **data ingestion và pipeline** — không phải ML inference (vẫn cần dedicated compute)
- Step Functions cho maintenance workflow có audit trail quan trọng cho aviation compliance
- Lambda không thay thế được toàn bộ hệ thống — chỉ là một lớp trong kiến trúc lớn hơn

*(Nguồn: [AWS re:Invent 2019 — Rolls-Royce](https://aws.amazon.com/solutions/case-studies/rolls-royce/), [Rolls-Royce Digital](https://www.rolls-royce.com/media/our-stories/innovation/2019/rolls-royce-and-aws.aspx))*

---

### Tổng kết Case Studies

| Công ty | Scale | Pattern chính | Kết quả nổi bật |
|---|---|---|---|
| **iRobot** | Hàng chục triệu robots | Event-Triggered (Pattern 1) | Team nhỏ vận hành fleet enterprise |
| **Volkswagen VAC** | Hàng triệu xe | Hybrid Streaming (Pattern 4) + Orchestration (Pattern 2) | GDPR compliance; auto-scale qua peak |
| **Rolls-Royce** | Nghìn động cơ | Hybrid Streaming + Orchestration | Predictive maintenance với audit trail |

---

## 11. Khi nào Dùng / Khi nào Không Dùng

> Đây là mục quan trọng nhất của chapter trong style *Fundamentals of Software Architecture*.
> Một architecture không phải "tốt" hay "xấu" — nó phù hợp hay không phù hợp với context cụ thể.

### 11.1 Khi nào Dùng Serverless IoT ✅

**1. Workload sporadic và không đoán trước được**
- Ví dụ: cảm biến chất lượng không khí gửi alert khi vượt ngưỡng
- Tiêu chí: < 1,000 events/giây trong 90% thời gian

**2. Team nhỏ không có DevOps chuyên dụng**
- Ví dụ: startup IoT 2–5 người cần ship nhanh
- Tiêu chí: Team < 5 người; không có dedicated infrastructure engineer

**3. Workload có spike khó dự đoán**
- Ví dụ: Fleet thiết bị thông minh nhà ở reconnect đồng loạt sau mất điện
- Tiêu chí: Ratio peak/baseline > 10×; container pre-provisioning không hiệu quả

**4. Latency requirement > 500ms (non-critical paths)**
- Ví dụ: Gửi push notification khi nhiệt độ nhà vượt ngưỡng
- Tiêu chí: User nhận thông báo trong vài giây là đủ

**5. Multi-step device lifecycle workflows**
- Ví dụ: Provisioning, OTA update, decommissioning
- Tiêu chí: Workflow có branching, compensation, và audit trail requirement

**6. Budget-sensitive với traffic thấp đến trung bình**
- Ví dụ: Smart agriculture sensor mạng gửi 1 reading/giờ
- Tiêu chí: < 10M events/ngày; muốn zero cost khi idle

---

### 11.2 Khi nào KHÔNG Dùng Serverless IoT ❌

**1. Latency < 200ms là yêu cầu cứng**
- Ví dụ: Safety shutdown cho robot công nghiệp; command-and-control thời gian thực
- Lý do: Cold start 100ms–3s không thể đảm bảo; dùng Edge-Cloud Hybrid thay

**2. Continuous high-throughput telemetry**
- Ví dụ: 100,000 devices × 1 msg/giây = 100,000 msg/giây sustained
- Lý do: Chi phí Lambda 5–10× so với Kinesis + ECS tại volume này

**3. Offline operation bắt buộc**
- Ví dụ: Remote oil pump trong vùng không có kết nối internet
- Lý do: Serverless là pure-cloud; không có local processing; dùng Edge-Cloud Hybrid

**4. In-memory stateful computation**
- Ví dụ: Sliding window anomaly detection trên 30 giây cuối của vibration data
- Lý do: Lambda không giữ state giữa invocations; cần Kinesis Data Analytics hoặc Flink

**5. ML inference trên large models**
- Ví dụ: Computer vision trên video stream từ factory cameras
- Lý do: Lambda max 10GB memory; GPU không hỗ trợ; cold start quá dài cho model load

**"Serverless breaks down when" — Bảng tóm tắt:**

| Ngưỡng | Lambda behavior | Recommendation |
|---|---|---|
| > 100 req/s sustained | 5–10× đắt hơn container | ECS Fargate / Kubernetes |
| < 200ms latency P99 | Cold start vi phạm SLA | Provisioned Concurrency hoặc containers |
| > 15 phút execution | Timeout | Fargate tasks hoặc EC2 |
| > 1,000 concurrent (AWS default) | Throttling | Request limit increase; hoặc containers |
| Offline operation needed | Zero capability | Edge-Cloud Hybrid (K3s + Greengrass) |
| In-memory state | Không khả thi | Kinesis Data Analytics, Flink, Spark |

---

## 12. Framework Quyết định

### 12.1 Six Questions Framework

Trả lời 6 câu hỏi theo thứ tự. Câu đầu tiên cho kết quả **Không** sẽ loại trừ Serverless:

```
Q1: Workload có cần offline operation không?
    → Có: Serverless không phù hợp → Edge-Cloud Hybrid
    → Không: Tiếp tục

Q2: Latency requirement có < 200ms (P99) không?
    → Có: Serverless không phù hợp (trừ khi Provisioned Concurrency OK về cost)
    → Không: Tiếp tục

Q3: Sustained throughput có > 100 req/s không?
    → Có: Tính break-even (§7.1); có thể container rẻ hơn
    → Không: Tiếp tục

Q4: Computation có cần in-memory state (sliding windows, ML inference)?
    → Có: Serverless không phù hợp → Kinesis Analytics / containers
    → Không: Tiếp tục

Q5: Team size có ≥ 5 với dedicated DevOps không?
    → Không (team nhỏ): Serverless càng phù hợp
    → Có (team lớn): Cả serverless và containers đều khả thi; chọn theo cost/latency

Q6: Budget có cho phép Provisioned Concurrency nếu cần?
    → Có: Serverless hoàn toàn khả thi
    → Không: Đánh giá cold start impact; nếu không chấp nhận được → containers
```

### 12.2 Decision Flowchart

```
                         START
                           │
                    Cần offline operation?
                   ┌───YES────────────────────────────────────────────────┐
                   │                                                      │
                   NO                                                     ▼
                   │                                           Edge-Cloud Hybrid
                   ▼                                                      │
            Latency P99 < 200ms?                                         │
           ┌───YES─────────────────────────────────────────────┐         │
           │                                                   │         │
           NO                                              Provisioned   │
           │                                               Concurrency   │
           ▼                                               cost OK?      │
    Sustained > 100 req/s?                                     │         │
   ┌───YES────────────────────────────────────────┐         YES│  NO─────┤
   │                                              │            │         │
   NO                                             ▼            ▼         │
   │                                      Break-even     Serverless      │
   ▼                                      analysis         với PC        │
  In-memory state required?               ┌─────┤                       │
 ┌───YES────────────────────────┐      Cheap│  Expensive                │
 │                              │      enough│      │                   │
 NO                             ▼            ▼      ▼                   │
 │                     Kinesis Analytics  Serverless  Containers        │
 ▼                     Flink/Spark                                      │
SERVERLESS ✅                                                            │
(Pattern 1, 2, 3, or 4                                                  │
 depending on workload)                                                  │
                              ◄──────────────────────────────────────────┘
```

### 12.3 Decision Matrix theo Use Case IoT Điển hình

| Use Case | Recommended Pattern | Lý do |
|---|---|---|
| **Smart home alerts** (< 10K devices) | Serverless Pattern 1 | Sporadic; team nhỏ; latency OK |
| **Industrial safety shutoff** | Edge-Cloud Hybrid | < 10ms; offline; không thể cold start |
| **Vehicle telemetry** (hàng triệu xe) | Serverless Pattern 4 (Hybrid Streaming) | High volume; ordered; elastic |
| **Device provisioning** | Serverless Pattern 2 (Orchestration) | Multi-step; sporadic; audit trail |
| **Predictive maintenance** | Serverless (ingestion) + ML Platform (inference) | Pipeline = serverless; inference = dedicated |
| **Smart city monitoring** (10M+ sensors) | Microservices + EDA + Serverless (mix) | Quá lớn cho pure serverless |
| **Agriculture sensors** (1 msg/giờ) | Serverless Pattern 3 (Scheduled Batch) | Rất thưa; $0 khi idle; đơn giản |
| **Factory video analytics** | Edge-Cloud Hybrid + dedicated GPU | Video stream = serverless không phù hợp |

### 12.4 Migration Path: Khi Serverless Không Còn Đủ

IoT systems thường bắt đầu với serverless và migrate khi scale tăng:

```
Stage 1: Startup/MVP         Stage 2: Growth              Stage 3: Enterprise
┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
│  Pure Serverless │ ──→     │  Serverless +    │ ──→     │  Microservices   │
│                  │ pain    │  Streaming       │  pain   │  + EDA           │
│  Pattern 1, 2, 3 │ signal  │  (Pattern 4)     │  signal │  + Serverless    │
│  < 1K devices    │         │  1K–100K devices │         │  (alerting only) │
└──────────────────┘         └──────────────────┘         └──────────────────┘

Migration triggers:
  → Pattern 1 sang Pattern 4: Lambda throttle errors bắt đầu xuất hiện
  → Pattern 4 sang Microservices+EDA: Break-even cost exceeded; team > 5 người
  → Thêm Edge: Latency requirement < 100ms hoặc offline operation cần thiết
```

---

## 13. Tài liệu Tham khảo

### Sách

| Tác giả | Tựa sách | Nhà xuất bản | Năm | Phần liên quan |
|---|---|---|---|---|
| Richards, M. & Ford, N. | *Fundamentals of Software Architecture* | O'Reilly | 2020 | Ch.17: Serverless Architecture; Ch.4: Architecture Characteristics |
| Fowler, M. | *Patterns of Enterprise Application Architecture* | Addison-Wesley | 2002 | Stateless service patterns |
| Newman, S. | *Building Microservices* (2nd ed.) | O'Reilly | 2021 | Service decomposition, event-driven patterns |
| Richardson, C. | *Microservices Patterns* | Manning | 2018 | Saga pattern (relevant cho Device Lifecycle Orchestration) |

### Cloud Provider Documentation

| Nguồn | URL | Phần liên quan |
|---|---|---|
| AWS Lambda Developer Guide | [docs.aws.amazon.com/lambda](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) | Concurrency, cold start, event source mapping |
| AWS IoT Core Developer Guide | [docs.aws.amazon.com/iot](https://docs.aws.amazon.com/iot/latest/developerguide/) | Rules Engine SQL, Device Shadow |
| AWS Lambda Quotas | [Lambda limits](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-limits.html) | Concurrency, execution, memory limits |
| Azure Functions Documentation | [learn.microsoft.com/azure/azure-functions](https://learn.microsoft.com/en-us/azure/azure-functions/) | Durable Functions, consumption plan |
| Azure IoT Hub | [learn.microsoft.com/azure/iot-hub](https://learn.microsoft.com/en-us/azure/iot-hub/) | Device Twin, message routing |

### Research Papers và Whitepapers

| Tác giả | Tựa | Venue | Năm |
|---|---|---|---|
| Agache, A. et al. | *Firecracker: Lightweight Virtualization for Serverless Applications* | NSDI 2020 | 2020 |
| CNCF WG Serverless | *CNCF Serverless Whitepaper v2* | CNCF | 2019 |
| Wang, L. et al. | *Peeking Behind the Curtains of Serverless Platforms* | USENIX ATC | 2018 |
| Eismann, S. et al. | *Serverless Applications: Why, When, and How?* | IEEE Software | 2021 |
| Shilkov, M. | *AWS Lambda Cold Start Performance* | mikhail.io | 2020–2024 |

### Industry Reports

| Nguồn | Tựa | Năm |
|---|---|---|
| Datadog | *State of Serverless 2024* | 2024 |
| CNCF | *Annual Survey 2024* | 2024 |
| Gartner | *Magic Quadrant for Cloud IoT Platforms* | 2023 |

### Case Study Sources

| Công ty | Nguồn |
|---|---|
| iRobot | [AWS re:Invent 2018 — IOT207](https://www.youtube.com/watch?v=YjdtjWWXzg0) |
| Volkswagen | [Microsoft Customer Stories](https://customers.microsoft.com/en-us/story/1346010428222360527-volkswagen-azure-automotive) |
| Rolls-Royce | [AWS Case Study](https://aws.amazon.com/solutions/case-studies/rolls-royce/) |
