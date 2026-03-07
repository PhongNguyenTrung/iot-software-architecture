# 📋 BÁO CÁO RÚT GỌN: KIẾN TRÚC PHẦN MỀM TRONG IoT

> **Chủ đề**: Software Architecture for Internet of Things (IoT)
> **Ngày**: 28/02/2026
> **Tác giả**: Phong Nguyen Trung
> **Lưu ý**: Phiên bản rút gọn — tập trung 4 chủ đề cốt lõi. Xem `report.md` để có phiên bản đầy đủ.

---

## Mục lục

1. [Giới thiệu tổng quan về IoT](#1-giới-thiệu-tổng-quan-về-iot)
2. [Đặc tính kiến trúc trong IoT](#2-đặc-tính-kiến-trúc-trong-iot)
3. [Các mẫu kiến trúc phần mềm + Đánh đổi](#3-các-mẫu-kiến-trúc-phần-mềm--đánh-đổi)
4. [Nghiên cứu tình huống thực tế](#4-nghiên-cứu-tình-huống-thực-tế)
5. [Kết luận](#5-kết-luận)

---

## 1. Giới thiệu tổng quan về IoT

### 1.1 Định nghĩa

**Internet of Things (IoT)** — Internet vạn vật — là mạng lưới các thiết bị vật lý được tích hợp cảm biến, phần mềm và khả năng kết nối mạng để thu thập, trao đổi và xử lý dữ liệu mà không cần sự can thiệp trực tiếp của con người.

### 1.2 Quy mô và tầm quan trọng

| Chỉ số | Giá trị |
|---|---|
| Số thiết bị IoT kết nối (2026) | ~20 tỷ thiết bị |
| Dự báo đến 2030 | 75+ tỷ thiết bị |
| Quy mô thị trường IoT (2030) | ~1.5 nghìn tỷ USD |
| Tốc độ tăng trưởng hàng năm (CAGR) | ~25% |

### 1.3 Các lĩnh vực ứng dụng chính

- **Công nghiệp 4.0 (IIoT)**: Giám sát máy móc, bảo trì dự đoán, tối ưu dây chuyền sản xuất
- **Nhà thông minh (Smart Home)**: Điều khiển ánh sáng, nhiệt độ, an ninh, thiết bị gia dụng
- **Thành phố thông minh (Smart City)**: Quản lý giao thông, năng lượng, rác thải, chiếu sáng
- **Y tế (Healthcare IoT)**: Giám sát bệnh nhân từ xa, thiết bị đeo theo dõi sức khỏe
- **Nông nghiệp thông minh**: Giám sát đất, tưới tiêu tự động, theo dõi gia súc
- **Logistics & Vận tải**: Theo dõi tài sản, quản lý đội xe, chuỗi cung ứng lạnh

### 1.4 Tại sao kiến trúc phần mềm quan trọng trong IoT

IoT khác biệt so với phần mềm truyền thống ở nhiều khía cạnh tạo ra thách thức kiến trúc đặc thù:

| Thách thức | Mô tả |
|---|---|
| **Tính không đồng nhất** | Hàng trăm loại thiết bị khác nhau về phần cứng, OS, giao thức |
| **Quy mô khổng lồ** | Hàng triệu thiết bị gửi dữ liệu liên tục |
| **Kết nối không ổn định** | Mạng không dây thường xuyên gián đoạn, độ trễ cao |
| **Yêu cầu thời gian thực** | Nhiều ứng dụng cần phản hồi trong mili-giây |
| **Tài nguyên hạn chế** | Thiết bị có CPU yếu, RAM ít (KB), pin giới hạn |
| **Bề mặt tấn công lớn** | Mỗi thiết bị là một điểm xâm nhập tiềm năng |
| **Vòng đời dài** | Thiết bị hoạt động 10–15 năm, cần cập nhật liên tục |

```
Kiến trúc kém × Số lượng thiết bị = Nợ kỹ thuật cấp số nhân
```

---

## 2. Đặc tính kiến trúc trong IoT

### 2.1 Khái niệm đặc tính kiến trúc

Trong thiết kế phần mềm, chúng ta phân biệt hai loại yêu cầu:

- **Yêu cầu chức năng:** Hệ thống *làm gì*? — "Thu thập nhiệt độ mỗi 30 giây"
- **Đặc tính kiến trúc:** Hệ thống phải *giỏi điều gì*? — "Xử lý 1 triệu kết nối đồng thời"

**Đặc tính kiến trúc** (còn gọi là *thuộc tính chất lượng* hoặc *yêu cầu phi chức năng*) là các thuộc tính đo lường được của hệ thống, buộc kiến trúc sư phải đưa ra quyết định cấu trúc cụ thể.

Quy trình thiết kế kiến trúc đúng đắn:

```
Yêu cầu nghiệp vụ IoT
         ↓
Đặc tính kiến trúc  ←── (Phần 2)
         ↓
Lựa chọn mẫu kiến trúc  ←── (Phần 3)
         ↓
Triển khai công nghệ
```

### 2.2 Tám đặc tính cốt lõi của IoT

| # | Đặc tính kiến trúc | Định nghĩa ngắn gọn | Góc độ IoT đặc thù | Chỉ số đo lường |
|---|---|---|---|---|
| 1 | **Khả năng mở rộng (Scalability)** | Xử lý tăng trưởng về thiết bị và dữ liệu | Hai chiều độc lập: số kết nối VÀ thông lượng dữ liệu | Thiết bị đồng thời, sự kiện/giây |
| 2 | **Độ tin cậy (Reliability)** | Dịch vụ đúng như thiết kế kể cả khi mất mạng | Thiết bị phải tự chủ offline — không dừng vì đám mây có sự cố | Uptime %, MTBF, tỷ lệ giao hàng tin nhắn |
| 3 | **Hiệu năng (Performance)** | Độ trễ và thông lượng đáp ứng yêu cầu | Deadline cứng vật lý — phần mềm ghép với quy trình thế giới thực | Độ trễ end-to-end, sự kiện/giây |
| 4 | **Bảo mật (Security)** | Bảo vệ khỏi truy cập trái phép | Bề mặt tấn công vật lý — không thể thêm bảo mật sau | CVE response time, thời gian xoay vòng chứng chỉ |
| 5 | **Khả năng bảo trì (Maintainability)** | Sửa đổi và cập nhật dễ dàng | OTA update cho thiết bị 10+ năm không thể tiếp cận vật lý | Thời gian OTA, tỷ lệ rollback thành công |
| 6 | **Khả năng tương tác (Interoperability)** | Giao tiếp với thiết bị và giao thức khác nhau | Không ứng dụng web nào phải nói MQTT + BLE + LoRaWAN + Modbus cùng lúc | Số giao thức hỗ trợ, thời gian tích hợp |
| 7 | **Khả năng quan sát (Observability)** | Suy ra trạng thái nội bộ từ đầu ra bên ngoài | Không thể SSH vào cảm biến — phải được thiết kế sẵn từ đầu | MTTD, MTTR, tỷ lệ cảnh báo giả |
| 8 | **Hiệu quả năng lượng (Energy Efficiency)** | Hoàn thành chức năng với điện năng tối thiểu | Không có analog trong phần mềm enterprise — pin AA phải dùng nhiều năm | Tuổi thọ pin, mW trung bình |

### 2.3 Đánh đổi giữa các đặc tính

Trong thực tế, không thể tối ưu tất cả đặc tính đồng thời. Kiến trúc sư phải thực hiện **đánh đổi có chủ ý**:

| Đặc tính A | ↔ | Đặc tính B | Xung đột cụ thể | Chiến lược giải quyết |
|---|---|---|---|---|
| **Bảo mật** (TLS/mTLS) | | **Hiệu năng** | TLS thêm 50–150ms overhead | Hardware crypto (AES-NI), TLS session resumption |
| **Bảo mật** (AES-256) | | **Năng lượng** | Mã hóa tăng chu kỳ CPU | Crypto coprocessor (TPM, ATECC608A) |
| **Độ tin cậy** (QoS 2) | | **Hiệu năng** (thông lượng) | QoS 2 cần 4-message handshake | QoS 1 cho telemetry, QoS 2 cho lệnh quan trọng |
| **Quan sát** (log liên tục) | | **Năng lượng** | Ghi log cần truyền radio | Batch log; gửi theo cửa sổ TX lên lịch |
| **Mở rộng** (microservices) | | **Bảo trì** (độ phức tạp) | Nhiều service = nhiều deployment | Đầu tư Kubernetes, GitOps ngay từ đầu |

> *"Kiến trúc tốt không phải tối đa hóa tất cả đặc tính — mà là thực hiện đánh đổi có chủ ý, phù hợp với ưu tiên nghiệp vụ."*

### 2.4 Đặc tính dẫn dắt lựa chọn mẫu kiến trúc

Xác định 2–3 đặc tính ưu tiên hàng đầu và dùng bảng này để chọn mẫu phù hợp:

| Đặc tính ưu tiên cao | Mẫu kiến trúc khởi điểm | Lý do |
|---|---|---|
| **Hiệu năng** < 100ms | Edge-Cloud Hybrid | Đám mây không thể nằm trong đường xử lý tới hạn |
| **Mở rộng** > 100K thiết bị | Microservices + EDA | Mở rộng ngang độc lập; broker hiệu năng cao |
| **Độ tin cậy** (tự chủ offline) | Edge-Cloud Hybrid | Rules engine cục bộ hoạt động không cần đám mây |
| **Năng lượng** (chạy pin) | EDA + MQTT QoS thấp | Thời gian thức tối thiểu; không polling |
| **Tương tác** (tích hợp đa giao thức) | EDA + gateway bridge | Event bus tách biệt producer đa giao thức với consumer |

---

## 3. Các mẫu kiến trúc phần mềm + Đánh đổi

### 3.1 Kiến trúc phân lớp (Layered Architecture)

**Mô tả**: Tổ chức hệ thống thành các lớp ngang, mỗi lớp có trách nhiệm riêng và giao tiếp theo chiều dọc với lớp liền kề.

```
┌──────────────────────┐
│   Trình bày (UI)     │
├──────────────────────┤
│   Logic nghiệp vụ    │
├──────────────────────┤
│   Truy cập dữ liệu   │
├──────────────────────┤
│   Hạ tầng            │
└──────────────────────┘
```

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Độ phức tạp** | Đơn giản, dễ hiểu, triển khai nhanh | "Quả bóng bùn lớn" khi hệ thống tăng trưởng | ✅ Lý tưởng cho POC; ❌ sụp đổ khi vượt ~1K thiết bị |
| **Khớp nối** | Phân tách trách nhiệm trong một lớp | Liên kết chặt giữa các lớp — thay đổi lan truyền | ❌ Thêm loại cảm biến mới đòi sửa đồng thời nhiều lớp |
| **Khả năng mở rộng** | Dễ suy luận ban đầu | Không thể mở rộng độc lập từng lớp | ❌ Nếu ingestion là nút thắt, phải scale *toàn bộ* ứng dụng |
| **Phù hợp đội ngũ** | Rào cản gia nhập thấp | Không phù hợp nhiều nhóm phát triển song song | ✅ Đúng cho 1–3 người; ❌ xung đột merge cho nhóm lớn |

**Khi nào sụp đổ trong IoT:** Số thiết bị vượt ~1,000; yêu cầu độ trễ < 100ms; nhiều nhóm làm việc song song.

**Phù hợp**: POC, prototype, đội nhỏ < 3 người, < 100 thiết bị.

---

### 3.2 Kiến trúc Microservices

**Mô tả**: Phân tách hệ thống thành các dịch vụ nhỏ, độc lập, mỗi dịch vụ sở hữu một khả năng domain cụ thể và database riêng.

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Quản lý    │  │  Thu nhận    │  │   Cảnh báo   │
│   Thiết bị   │  │  Telemetry   │  │   Service    │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       └─────────────────┼─────────────────┘
                   Message Broker (Kafka)
       ┌─────────────────┼─────────────────┐
┌──────┴───────┐  ┌──────┴───────┐  ┌──────┴───────┐
│  Cập nhật    │  │  Phân tích   │  │  Quản lý     │
│  OTA         │  │  Analytics   │  │  Người dùng  │
└──────────────┘  └──────────────┘  └──────────────┘
```

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Khả năng mở rộng** | Mở rộng từng dịch vụ độc lập | Nhiều hạ tầng hơn cần quản lý | ✅ Telemetry Ingestion scale đến 10M msg/s; OTA ở mức tối thiểu |
| **Cô lập lỗi** | Lỗi một dịch vụ không lan truyền | Cần circuit breakers và timeout | ✅ Analytics crash không ảnh hưởng kết nối thiết bị |
| **Vận hành** | Mỗi dịch vụ độc lập triển khai | Kubernetes + service mesh + observability tốn kém | ❌ Nhóm nhỏ dành 40–60% công sức vận hành thay vì tính năng |
| **Độ trễ** | Dịch vụ tối ưu cho mục đích riêng | Network hop cộng thêm 10–30ms mỗi bước | ❌ Đường xử lý lệnh qua 3 dịch vụ có thể không đạt < 100ms |

**Khi nào sụp đổ:** Nhóm < 5 kỹ sư; sản phẩm giai đoạn sớm; đường xử lý đòi hỏi < 10ms.

**Phù hợp**: Nền tảng IoT lớn (10K+ thiết bị), tổ chức đa nhóm.

---

### 3.3 Kiến trúc hướng sự kiện (Event-Driven Architecture — EDA)

**Mô tả**: Các thành phần giao tiếp bằng cách phát và tiêu thụ sự kiện qua event broker trung tâm. IoT vốn dĩ là hướng sự kiện — cảm biến liên tục tạo ra sự kiện.

```
Cảm biến ──publish──▶ [Event Broker] ──deliver──▶ Consumers
                      (Kafka/MQTT/RabbitMQ)       ├── Cảnh báo
                                                  ├── Lưu trữ
                                                  └── Phân tích
```

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Khớp nối** | Producer và consumer hoàn toàn tách biệt | Hợp đồng ngầm qua event schema — thay đổi schema làm vỡ consumer | ✅ Thêm ML consumer mới mà không sửa firmware cảm biến |
| **Khả năng mở rộng** | Kafka xử lý hàng triệu sự kiện/giây | Broker là điểm lỗi duy nhất nếu không cluster | ✅ Kafka hấp thụ tự nhiên đỉnh telemetry IoT |
| **Tính nhất quán** | Event log cho phép replay và phục hồi | Eventual consistency — consumer có thể lag 50–500ms | ❌ Dashboard hiển thị trạng thái cũ; ✅ chấp nhận được cho giám sát |
| **Debug** | Event log cung cấp forensics đầy đủ | Trace lỗi qua chuỗi async phức tạp | ❌ Cần distributed tracing để phân tích chuỗi sự kiện |

**Khi nào sụp đổ:** Đường lệnh điều khiển cần phản hồi tức thì; dự án nhỏ (broker là overhead thừa).

**Phù hợp**: Hệ thống giám sát và cảnh báo thời gian thực, thông lượng cao (triệu sự kiện/giây).

---

### 3.4 Kiến trúc Serverless

**Mô tả**: Logic ứng dụng chạy trong các hàm phi trạng thái, kích hoạt bởi sự kiện, được quản lý hoàn toàn bởi nhà cung cấp đám mây.

```
[Thiết bị IoT] → [IoT Hub] → trigger → [Lambda/Function] → [Database]
                                │
                         if bất thường
                                ↓
                         [Lambda] → [SNS/Email thông báo]
```

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Vận hành** | Không quản lý server — không cần DevOps | Vendor lock-in; di chuyển đòi viết lại | ✅ Startup 2 người chạy cảnh báo cho 10K thiết bị |
| **Chi phí** | Trả theo lần gọi — gần bằng 0 khi không có traffic | Ở tải liên tục, đắt hơn container 5–10× | ✅ Cảnh báo thưa thớt ❌ Ingestion liên tục 1M sự kiện/ngày |
| **Độ trễ** | Warm function: < 10ms | Cold start: 100ms–2s | ❌ Không phù hợp cho yêu cầu < 100ms |
| **Trạng thái** | Thiết kế stateless rõ ràng | Mỗi lần gọi phải lấy lại context từ DB | ❌ Theo dõi trạng thái thiết bị cần external state store |

**Khi nào sụp đổ:** Telemetry liên tục tần suất cao; đường xử lý yêu cầu < 200ms; quy trình stateful phức tạp.

**Phù hợp**: Pipeline cảnh báo, biến đổi dữ liệu, workload không đoán trước, đội nhỏ.

> **Phân tích chuyên sâu:** Xem [docs/07-serverless-iot.md](docs/07-serverless-iot.md) — bao gồm phân tích chi phí thực tế theo quy mô fleet, so sánh AWS/Azure/GCP, 4 mẫu kiến trúc chi tiết (alerting, telemetry, provisioning, OTA), cold start, bảo mật và quan sát, với nguồn trích dẫn uy tín.

---

### 3.5 Kiến trúc lai Biên-Đám mây (Edge-Cloud Hybrid)

**Mô tả**: Phân phối xử lý giữa edge (gần thiết bị) và cloud. Đây là **mẫu kiến trúc chủ đạo cho IoT doanh nghiệp** từ 2025. Thay vì gửi toàn bộ dữ liệu thô lên đám mây, hệ thống xử lý dữ liệu nhạy cảm về thời gian ngay tại edge, chỉ đồng bộ dữ liệu tổng hợp và trạng thái quan trọng lên cloud.

```
┌──────────────────────────── ĐÁM MÂY ─────────────────────────────┐
│  Huấn luyện ML  │  Lưu trữ dài hạn  │  Quản lý fleet  │  APIs   │
│  Digital Twin   │  Phân tích BI      │  OTA Updates    │ Dashboard│
└──────────────────────────────┬────────────────────────────────────┘
                               │ Đồng bộ tổng hợp / lệnh điều khiển
                               │ (MQTT, AMQP, HTTPS — mã hoá TLS 1.3)
┌──────────────────────────────┼────────────────────────────────────┐
│                        BIÊN (EDGE)                                │
│  Rules Engine │  Local Time-Series DB  │  AI Inference Model     │
│  Protocol Gateway │  Store-and-Forward Buffer │  OPC-UA Server   │
└──────────────────────────────┬────────────────────────────────────┘
                               │ Fieldbus / OPC-UA / MQTT
              ┌────────────────┼────────────────┐
         [Cảm biến]       [Camera]          [Actuator]
              TẦNG THIẾT BỊ (Device Tier)
```

#### Mô hình đồng bộ hóa

| Mô hình | Mô tả | Trường hợp sử dụng |
|---|---|---|
| **Store-and-Forward** | Đệm dữ liệu cục bộ, gửi khi có mạng | Kết nối không ổn định (mạng di động, vệ tinh) |
| **Eventual Consistency** | Biên và đám mây hội tụ theo thời gian | Telemetry không khẩn cấp |
| **Edge-First** | Biên ra quyết định, đám mây học từ đó | Hệ thống an toàn thời gian thực (IIoT) |
| **Cloud-First** | Đám mây ra quyết định, biên thực thi | Điều phối toàn fleet |
| **Bi-directional Sync** | Cả biên và đám mây đều có thể sửa đổi trạng thái | Device twin / device shadow |

> **Nguồn:** AWS IoT Greengrass — [Store and forward messages](https://docs.aws.amazon.com/greengrass/v2/developerguide/store-and-forward.html); Azure IoT Hub — [Device twins](https://learn.microsoft.com/en-us/azure/iot-hub/iot-hub-devguide-device-twins)

#### Nền tảng triển khai phổ biến (2025)

| Nền tảng | Nhà cung cấp | Khả năng cốt lõi | Nguồn |
|---|---|---|---|
| **AWS IoT Greengrass v2** | Amazon Web Services | Lambda tại edge, local MQTT broker, shadow sync, component model | [AWS Docs](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html) |
| **Azure IoT Edge** | Microsoft | Docker containers tại edge, offline operation, IoT Hub integration, AI deployment | [MS Docs](https://learn.microsoft.com/en-us/azure/iot-edge/about-iot-edge) |
| **Google Distributed Cloud Edge** | Google Cloud | Kubernetes tại edge, Vertex AI Edge deployment, Anthos clusters | [GCP Docs](https://cloud.google.com/distributed-cloud/edge/latest/docs/overview) |
| **Eclipse Kura** | Eclipse Foundation | OSGi-based open-source edge framework, protocol bridging (OPC-UA, Modbus) | [Eclipse Kura](https://eclipse.dev/kura/) |
| **K3s + Rancher** | SUSE/Rancher | Lightweight Kubernetes tại edge node, phù hợp IIoT on-premise | [K3s.io](https://k3s.io/) |

#### Hệ sinh thái mã nguồn mở (CNCF & LF Edge)

Ngoài các nền tảng thương mại, cộng đồng mã nguồn mở cung cấp các framework được quản trị bởi CNCF và Linux Foundation:

| Dự án | Tổ chức | Đặc điểm nổi bật | Nguồn |
|---|---|---|---|
| **KubeEdge** | CNCF Graduated (9/2024) | Kubernetes-native edge, chỉ 70MB RAM, tự chủ offline hoàn toàn, CloudHub + EdgeController + Edged | [kubeedge.io](https://kubeedge.io/) · [CNCF](https://www.cncf.io/projects/kubeedge/) |
| **Project EVE** | LF Edge Impact Stage (2024) | Universal Linux OS cho edge, multi-node resilience, hỗ trợ bare-metal & VM | [LF Press](https://www.linuxfoundation.org/press/project-eve-hits-maturity-within-lf-edge-releases-multi-node-support-for-enhanced-edge-computing-resilience) |
| **EdgeX Foundry** | LF Edge | Microservices framework cho edge IoT, protocol-agnostic, REST/MQTT APIs | [lfedge.org](https://lfedge.org/our-projects/) |
| **Eclipse Kura v5.6** | Eclipse Foundation | OSGi-based gateway, Wires visual programming, OPC-UA/Modbus bridge, container deployment | [eclipse-kura.github.io](https://eclipse-kura.github.io/kura/docs-release-5.6/) |

> **KubeEdge vs K3s:** K3s là Kubernetes nhỏ gọn cho các node có đủ tài nguyên (≥ 512MB RAM). KubeEdge phù hợp hơn cho thiết bị cực hạn chế tài nguyên và cần tự chủ hoàn toàn khi mất kết nối đám mây.

#### Tiêu chuẩn & Đặc tả quốc tế

Edge-Cloud Hybrid trong IoT doanh nghiệp phải tuân thủ các tiêu chuẩn sau:

| Tiêu chuẩn | Tổ chức | Nội dung liên quan | Nguồn |
|---|---|---|---|
| **NIST SP 800-213** | NIST (Mỹ) | Hướng dẫn bảo mật thiết bị IoT — định danh, cập nhật an toàn, bảo vệ dữ liệu cho edge nodes | [nvlpubs.nist.gov](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-213.pdf) (2022) |
| **ETSI GS MEC 003 v3.1.1** | ETSI (châu Âu) | Kiến trúc tham chiếu Multi-access Edge Computing — thành phần chức năng, điểm tham chiếu, MEC services | [etsi.org](https://www.etsi.org/deliver/etsi_gs/MEC/001_099/003/03.01.01_60/gs_mec003v030101p.pdf) (2022) |
| **ETSI White Paper #59** | ETSI | MEC tích hợp IoT — oneM2M APIs, ETSI MEC APIs, edge service placement | [etsi.org WP59](https://www.etsi.org/images/files/ETSIWhitePapers/ETSI-WP59-Enabling-Multi-access-Edge-Computing-in-iot.pdf) (2020) |
| **OPC-UA / IEC 62541** | OPC Foundation | Giao thức M2M công nghiệp chuẩn cho gateway biên — semantic data model, publish/subscribe | [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) |

#### AI tại biên (Edge AI)

Một trong những lợi thế cạnh tranh lớn nhất của Edge-Cloud Hybrid là khả năng chạy **suy diễn ML (inference) ngay tại thiết bị biên** mà không cần kết nối đám mây:

| Phần cứng | Hiệu năng Inference | Trường hợp sử dụng điển hình |
|---|---|---|
| **NVIDIA Jetson Orin** | 275 TOPS | Phát hiện đối tượng YOLO 30fps, phân tích video an ninh |
| **Google Coral TPU** | 4 TOPS | Phân loại hình ảnh nhúng, thiết bị tiêu dùng |
| **Intel OpenVINO** | Tùy CPU/VPU | Tối ưu mô hình cho edge Intel (NUC, Industrial PC) |
| **Qualcomm AI Hub** | Tùy SoC | Mobile/embedded inference, smartphone và camera thông minh |

> **Nguồn:** NVIDIA — [Jetson Orin Product Brief](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/); Google — [Coral Edge TPU](https://coral.ai/docs/edgetpu/faq/)

Cập nhật mô hình AI sau khi triển khai yêu cầu **OTA model delivery** — pipeline riêng biệt để đẩy trọng số mô hình mới đến hàng nghìn node biên một cách an toàn và nguyên tử.

**Phân tích đánh đổi chi tiết:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Độ trễ** | Xử lý cục bộ — < 1ms on-device, < 10ms tại edge gateway | Thêm một tầng kiến trúc phải quản lý và quan sát | ✅ Ngắt khẩn cấp an toàn nhà máy không thể chờ round-trip đám mây (50–200ms) |
| **Độ tin cậy** | Tiếp tục hoạt động khi mất kết nối đám mây | Đồng bộ sau khi kết nối lại phức tạp (conflict resolution, thứ tự, deduplication) | ✅ Nút lưới điện thông minh quản lý tải trong khi WAN bị ngắt 4 giờ |
| **Băng thông** | Tổng hợp tại biên — gửi 1–5% dữ liệu thô lên cloud | Phần cứng edge (NVIDIA Jetson, PC công nghiệp) tốn $500–$5,000+/site | ✅ Nhà máy 500 camera: xử lý video tại biên tiết kiệm 95% băng thông đám mây |
| **Chủ quyền dữ liệu** | Dữ liệu nhạy cảm không rời cơ sở vật chất | Residency đạt được qua cấu hình, không phải kiến trúc — cần kỷ luật vận hành | ✅ GDPR/HIPAA: dữ liệu bệnh nhân xử lý tại bệnh viện, không lên cloud |
| **Bảo mật** | Phân đoạn mạng giới hạn bán kính tấn công khi bị xâm phạm | Bảo mật vật lý phần cứng edge là trách nhiệm nhà vận hành | ❌ Kẻ tấn công với quyền truy cập vật lý có thể trích xuất thông tin xác thực đám mây từ edge gateway |
| **AI tại biên** | Suy diễn ML không cần round-trip đám mây | Cập nhật mô hình yêu cầu OTA đến hàng nghìn node | ✅ YOLO object detection trên Jetson: 30fps cục bộ. ❌ Deploy mô hình mới đến 1,000 node mất nhiều giờ |
| **Độ phức tạp** | Công cụ phù hợp cho yêu cầu IoT doanh nghiệp phức tạp | Mẫu phức tạp nhất — cần chuyên môn distributed systems, edge runtime (K3s, Greengrass), sync protocols | ❌ Nhóm mới với IoT không nên bắt đầu từ đây — đường cong học tập trì hoãn ra mắt hàng tháng |

#### Tích hợp 5G và MEC (Multi-access Edge Computing)

5G là yếu tố tăng tốc quan trọng cho Edge-Cloud Hybrid. **MEC (Multi-access Edge Computing)** theo chuẩn ETSI đưa tài nguyên tính toán xuống sát trạm gốc 5G (radio access network), loại bỏ hầu hết độ trễ WAN:

| Mô hình kết nối | Độ trễ điển hình | Phù hợp |
|---|---|---|
| Thiết bị → Cloud (4G/LTE) | 50–150ms | Telemetry không khẩn cấp |
| Thiết bị → Edge node on-premise | 1–10ms | IIoT safety, real-time control |
| Thiết bị → MEC node (5G base station) | < 5ms | Connected vehicles, AR/VR công nghiệp |
| Thiết bị → On-device inference | < 1ms | Safety shutoff, haptic feedback |

**Ứng dụng MEC-IoT điển hình:**
- **Nhà máy thông minh**: 5G private network + MEC xử lý telemetry CNC machines tại local, không phụ thuộc WAN
- **Thành phố thông minh**: Phân tích camera giao thông tại MEC node của nhà mạng, quyết định đèn tín hiệu < 5ms
- **Xe kết nối (V2X)**: Phát hiện va chạm và phối hợp xe tại MEC, độ trễ không thể đạt được qua cloud

> **Nguồn:** ETSI — [ETSI MEC Technology Page](https://www.etsi.org/technologies/multi-access-edge-computing); ETSI MEC Phase 4 (2024) với Linux Foundation collaboration cho open APIs; MDPI 2024 — [Integrating MEC into Open 5G Core](https://www.mdpi.com/2673-4001/5/2/22)

**Khi nào sụp đổ:**
- Nhóm thiếu kinh nghiệm hệ thống phân tán — lỗi đồng bộ hóa rất tinh vi và có thể thảm khốc (thực thi lệnh hai lần, mất cập nhật)
- Dự án hạn chế CapEx — phần cứng edge mỗi site là chi phí upfront đáng kể
- Dữ liệu không quan trọng mà đám mây có thể xử lý với chi phí hợp lý

**Phù hợp**: IIoT với yêu cầu an toàn, hệ thống cần hoạt động offline, xử lý video/ảnh, latency < 10ms, chủ quyền dữ liệu (GDPR/HIPAA).

#### Triển khai thực tế đã được công bố

| Tổ chức | Mô tả | Nguồn |
|---|---|---|
| **Rolls-Royce** | ECU preprocessing trên 4,500+ động cơ phản lực, MQTT qua vệ tinh đến Azure Databricks | [Microsoft Customer Story](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks) · [RTInsights](https://www.rtinsights.com/rolls-royce-jet-engine-maintenance-iot/) |
| **John Deere** | Edge AI trên 1.5M+ máy nông nghiệp, quyết định gieo hạt từng cây tại edge (<100ms) | [Databricks Blog 2021](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html) · [Harvard D3](https://d3.harvard.edu/platform-digit/submission/farm-to-data-table-john-deere-and-data-in-precision-agriculture/) |
| **BMW Group** | AWS IoT Greengrass tại 30+ nhà máy toàn cầu, kiểm soát chất lượng bằng computer vision | [AWS Case Study](https://aws.amazon.com/solutions/case-studies/bmw-group-iot/) |
| **Siemens MindSphere** | MindConnect edge runtime (IoT2050) + cloud analytics, 200+ giao thức công nghiệp | [AWS re:Invent MFG202](https://d1.awsstatic.com/events/reinvent/2019/Building_on_AWS_The_architecture_of_the_Siemens_MindSphere_platform_MFG202.pdf) · [Siemens Whitepaper](https://www.plm.automation.siemens.com/media/global/en/Siemens-MindSphere-Whitepaper-69993_tcm27-29087.pdf) |
| **GE Predix** | Edge analytics cho 400+ cơ sở sản xuất, Modbus/OPC-UA/DDS qua Predix Machine | [GE Predix Whitepaper](https://ecosystems4innovating.wordpress.com/wp-content/uploads/2016/11/predix-the-platform-for-the-industrial-internet-whitepaper.pdf) · [MIT Sloan Review](https://sloanreview.mit.edu/case-study/ge-big-bet-on-data-and-analytics/) |

#### Nghiên cứu học thuật

> **Peer-reviewed research xác nhận các đánh đổi kiến trúc trên:**
>
> - *"Edge Computing and Cloud Computing for Internet of Things: A Review"* — MDPI Machines Journal, 2024. Phân tích toàn diện các đánh đổi latency/scalability/privacy trong kiến trúc hybrid IoT. [mdpi.com/2227-9709/11/4/71](https://www.mdpi.com/2227-9709/11/4/71)
>
> - *"Edge Computing for IoT — Comprehensive Survey"* — arXiv, 2024. Khảo sát các mẫu triển khai edge-cloud cho IoT và phân loại workload theo đặc tính thời gian. [arxiv.org/html/2402.13056v1](https://arxiv.org/html/2402.13056v1)
>
> - *"On the Edge of the Deployment: A Survey on Multi-access Edge Computing"* — ACM Computing Surveys, 2023. Phân tích chuyên sâu về kiến trúc MEC, service placement, và orchestration. [dl.acm.org/doi/10.1145/3529758](https://dl.acm.org/doi/10.1145/3529758)
>
> - *"Integrating IoT and 6G: Applications of Edge Intelligence"* — IEEE Computer Society, 2025. Xu hướng kết hợp edge AI và 6G cho thế hệ IoT tiếp theo. [computer.org](https://www.computer.org/csdl/journal/sc/2025/04/11074426/28eYP7r5TUI)

#### Tài liệu tham khảo

**Tài liệu nhà cung cấp (Vendor Documentation)**
- AWS — *IoT Greengrass v2 Developer Guide* (2024): https://docs.aws.amazon.com/greengrass/v2/developerguide/
- AWS — *IoT Greengrass Nucleus Lite for constrained devices* (2024): https://aws.amazon.com/blogs/iot/aws-iot-greengrass-nucleus-lite-revolutionizing-edge-computing-on-resource-constrained-devices/
- Microsoft — *Azure IoT Edge documentation* (2024): https://learn.microsoft.com/en-us/azure/iot-edge/
- Microsoft — *Azure IoT Edge Runtime architecture* (2024): https://learn.microsoft.com/en-us/azure/iot-edge/iot-edge-runtime
- Microsoft — *Azure Industrial IoT / OPC-UA Platform*: https://azure.github.io/Industrial-IoT/
- Google Cloud — *Distributed Cloud Edge overview* (2024): https://cloud.google.com/distributed-cloud/edge/latest/docs/overview
- Google Cloud — *Distributed Cloud Edge use cases* (2024): https://cloud.google.com/blog/products/infrastructure-modernization/google-distributed-cloud-edge-appliance-use-cases/

**Mã nguồn mở (Open-Source Frameworks)**
- KubeEdge — *CNCF Graduated Project* (2024): https://kubeedge.io/ · https://www.cncf.io/projects/kubeedge/
- LF Edge — *Project EVE Impact Stage* (2024): https://www.linuxfoundation.org/press/project-eve-hits-maturity-within-lf-edge-releases-multi-node-support-for-enhanced-edge-computing-resilience
- LF Edge — *EdgeX Foundry & all LF Edge projects*: https://lfedge.org/our-projects/
- Eclipse Foundation — *Eclipse Kura v5.6 Documentation*: https://eclipse-kura.github.io/kura/docs-release-5.6/
- K3s — *Lightweight Kubernetes for edge*: https://k3s.io/

**Tiêu chuẩn quốc tế (International Standards)**
- NIST — *SP 800-213: IoT Device Cybersecurity Guidance* (2022): https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-213.pdf
- ETSI — *GS MEC 003 v3.1.1: MEC Reference Architecture* (2022): https://www.etsi.org/deliver/etsi_gs/MEC/001_099/003/03.01.01_60/gs_mec003v030101p.pdf
- ETSI — *White Paper #59: Enabling MEC in IoT* (2020): https://www.etsi.org/images/files/ETSIWhitePapers/ETSI-WP59-Enabling-Multi-access-Edge-Computing-in-iot.pdf
- ETSI — *MEC Technology Page (Phase 4, 2024)*: https://www.etsi.org/technologies/multi-access-edge-computing

**Phần cứng Edge AI**
- NVIDIA — *Jetson Orin Product Brief*: https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/jetson-orin/
- Google — *Coral Edge TPU*: https://coral.ai/docs/edgetpu/faq/

**Nghiên cứu học thuật (Academic Research)**
- MDPI Machines Journal — *Edge Computing and Cloud Computing for IoT: A Review* (2024): https://www.mdpi.com/2227-9709/11/4/71
- arXiv — *Edge Computing for IoT: Comprehensive Survey* (2024): https://arxiv.org/html/2402.13056v1
- ACM Computing Surveys — *Survey on Multi-access Edge Computing* (2023): https://dl.acm.org/doi/10.1145/3529758
- IEEE Computer Society — *Integrating IoT and 6G: Edge Intelligence* (2025): https://www.computer.org/csdl/journal/sc/2025/04/11074426/28eYP7r5TUI
- MDPI — *Integrating MEC into Open 5G Core* (2024): https://www.mdpi.com/2673-4001/5/2/22

**Nghiên cứu tình huống (Industry Case Studies)**
- Microsoft Customer Story — *Rolls-Royce + Azure Databricks* (2023): https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks
- Rolls-Royce — *IntelligentEngine Press Release* (2018): https://www.rolls-royce.com/media/press-releases/2018/06-02-2018-rr-intelligentengine-driven-by-data.aspx
- Databricks Blog — *John Deere Precision Agriculture* (2021): https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html
- Harvard D3 — *John Deere and Data in Precision Agriculture*: https://d3.harvard.edu/platform-digit/submission/farm-to-data-table-john-deere-and-data-in-precision-agriculture/
- AWS re:Invent 2019 MFG202 — *Siemens MindSphere on AWS*: https://d1.awsstatic.com/events/reinvent/2019/Building_on_AWS_The_architecture_of_the_Siemens_MindSphere_platform_MFG202.pdf
- GE Predix Whitepaper — *Platform for the Industrial Internet*: https://ecosystems4innovating.wordpress.com/wp-content/uploads/2016/11/predix-the-platform-for-the-industrial-internet-whitepaper.pdf

---

### 3.6 Bảng so sánh toàn diện 5 mẫu

Đánh giá tương đối (Thấp / Trung bình / Cao / Rất cao) cho môi trường IoT sản xuất:

| Chiều | Layered | Microservices | EDA | Serverless | Edge-Cloud Hybrid |
|---|---|---|---|---|---|
| **Độ phức tạp ban đầu** | Thấp | Cao | Trung bình | Thấp | Rất cao |
| **Độ phức tạp vận hành** | Thấp | Rất cao | Cao | Thấp | Rất cao |
| **Khả năng mở rộng** | Thấp | Cao | Rất cao | Rất cao | Cao |
| **Độ trễ thấp nhất** | Trung bình | Trung bình | Thấp | Thay đổi* | Rất thấp |
| **Khả năng offline** | Không | Không | Không | Không | Đầy đủ |
| **Hiệu quả băng thông** | Không | Không | Không | Không | Rất cao |
| **Quy mô nhóm cần thiết** | 1–3 người | 5+ người | 3–5 người | 1–3 người | 5+ người |
| **Chi phí ban đầu** | Thấp | Trung bình | Trung bình | Rất thấp | Cao (HW biên) |

\* Serverless: thấp với warm function (< 10ms), cao với cold start (100ms–2s)

### 3.7 Hướng dẫn chọn mẫu theo kịch bản

| Kịch bản | Mẫu được khuyến nghị | Lý do chính |
|---|---|---|
| **POC / Prototype** (< 100 thiết bị) | Layered | Đơn giản, triển khai nhanh, ít rủi ro |
| **Startup nhỏ** (100–1K thiết bị, đội 2–4 người) | Layered + Serverless alerts | Chi phí thấp, không cần DevOps |
| **Tăng trưởng** (1K–100K, đội 5–10 người) | Microservices + EDA | Mở rộng độc lập, phân tách domain |
| **Doanh nghiệp** (100K+ thiết bị, yêu cầu realtime) | Edge-Cloud Hybrid + Microservices + EDA | Offline resilience, latency < 100ms, quy mô lớn |
| **Thiết bị công nghiệp / IIoT** (safety-critical) | Edge-First + Microservices | Tự chủ cục bộ, không phụ thuộc WAN cho quyết định an toàn |

**Con đường tiến hóa điển hình:**

```
Giai đoạn 1: POC     Giai đoạn 2: Tăng trưởng   Giai đoạn 3: Quy mô   Giai đoạn 4: Doanh nghiệp
 Layered          →   Microservices           →   + EDA             →   + Edge-Cloud Hybrid
 < 1K thiết bị        + Serverless                100K+ thiết bị        Doanh nghiệp lớn
                       1K–100K thiết bị
```

---

## 4. Nghiên cứu tình huống thực tế

> Các tình huống dưới đây là các dự án **thực tế từ các công ty lớn**, có tài liệu công khai từ AWS re:Invent, Microsoft Customer Stories, Databricks Blog và học thuật Harvard.

### 4.1 Rolls-Royce — IntelligentEngine

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Rolls-Royce Holdings plc |
| **Lĩnh vực** | Hàng không dân dụng / IIoT |
| **Quy mô** | 4,500+ động cơ máy bay giám sát liên tục; hàng nghìn cảm biến/động cơ |
| **Mô hình kinh doanh** | TotalCare "Power by the Hour" — hãng bay trả theo giờ bay |
| **Kiến trúc** | Edge-Cloud Hybrid + Digital Twin |
| **Cloud** | Microsoft Azure (IoT Backend + Databricks ML) |
| **Giao thức** | MQTT qua liên kết vệ tinh/mặt đất |
| **Đặc tính chủ đạo** | Độ tin cậy, Hiệu năng, Khả năng quan sát |

**Quyết định kiến trúc quan trọng:**
- **Xử lý biên tại ECU**: Tiền xử lý tại chỗ giảm khối lượng truyền và phát hiện bất thường tức thì
- **Lớp chuẩn hóa dữ liệu**: Hàng trăm biến thể động cơ → 1 định dạng thống nhất cho analytics
- **Azure Databricks + ML tự học**: Dự đoán hao mòn linh kiện và tuổi thọ còn lại của từng động cơ
- **Digital Twin thời gian thực**: Bản sao số đồng bộ liên tục — mô phỏng "what-if" trước khi can thiệp bảo trì

**Kết quả** *(nguồn: [Microsoft Customer Story](https://www.microsoft.com/en/customers/story/23201-rolls-royce-azure-databricks))*:
- Tiết kiệm **hàng triệu USD** nhờ tránh được bảo trì ngoài kế hoạch
- Giám sát thời gian thực **4,500+ động cơ** trên toàn cầu đội bay thương mại
- Mô hình TotalCare khả thi: chỉ có thể cam kết SLA khi IoT dự đoán được chính xác

**Bài học kiến trúc:** *Mô hình kinh doanh và kiến trúc gắn liền nhau* — dịch vụ "Power by the Hour" không thể tồn tại nếu không có khả năng giám sát dự đoán liên tục. Kiến trúc IoT không phải là tính năng, mà **là sản phẩm**.

---

### 4.2 John Deere — Nền tảng Nông nghiệp Chính xác IoT

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Deere & Company (John Deere) |
| **Lĩnh vực** | Nông nghiệp / Thiết bị kết nối |
| **Quy mô** | 1.5 triệu máy kết nối, 500 triệu mẫu Anh được quản lý |
| **Tần suất** | Telemetry mỗi **5 giây** từ mỗi máy |
| **Tăng trưởng dữ liệu** | Khối lượng dữ liệu tăng gấp đôi/gấp ba mỗi năm |
| **Kiến trúc** | Multi-Cloud Hybrid + Edge + Microservices |
| **Cloud** | AWS (chính: DynamoDB, Kinesis, OpenSearch, S3) + Azure Arc + Data Center riêng |
| **Đặc tính chủ đạo** | Khả năng mở rộng, Tính tương tác, Hiệu năng |

**Quyết định kiến trúc quan trọng:**
- **Multi-cloud có chủ đích**: AWS cho IoT/analytics; Azure cho vận hành nhà máy sản xuất — các workload khác nhau cần nền tảng tối ưu khác nhau
- **Kinesis cho streaming**: Xử lý dữ liệu thời gian thực ở quy mô tăng trưởng hàm mũ
- **AI cấp độ từng cây trồng**: Mô hình Databricks xác định lượng hạt giống/phân bón tối ưu cho từng cây — hàng nghìn cây/giây/máy
- **Telemetry 5 giây**: Cân bằng giữa dữ liệu có giá trị hành động và chi phí băng thông mạng di động

**Kết quả** *(nguồn: [Databricks Blog](https://www.databricks.com/blog/2021/07/09/down-to-the-individual-grain-how-john-deere-uses-industrial-ai-to-increase-crop-yields-through-precision-agriculture.html))*:
- **Nông nghiệp chính xác cấp cây trồng**: AI xác định xử lý tối ưu cho từng cây — chuyển đổi từ quy mô cánh đồng sang cây trồng
- Giảm lãng phí phân bón, thuốc trừ sâu, hạt giống qua công nghệ tỷ lệ biến thiên
- Kiến trúc được thiết kế để xử lý **dữ liệu tăng gấp đôi mỗi năm** mà không cần tái kiến trúc

**Bài học kiến trúc:** *Thiết kế cho tăng trưởng 10× ngay từ đầu* — dữ liệu tăng hàm mũ, không tuyến tính. Multi-cloud không phải để tránh lock-in mà để **chọn nền tảng phù hợp nhất cho từng loại workload**.

---

### 4.3 Siemens MindSphere / Insights Hub — Nền tảng IIoT PaaS

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Siemens AG |
| **Lĩnh vực** | Nền tảng IoT công nghiệp (PaaS) |
| **Đặc điểm** | Multi-tenant — phục vụ hàng nghìn khách hàng công nghiệp toàn cầu |
| **Kiến trúc** | Microservices + EDA + Hybrid Cloud |
| **Cloud** | AWS (hosted) |
| **Streaming** | Amazon Kinesis (managed) HOẶC Apache Kafka (self-managed) |
| **Kết nối thiết bị** | MindConnect Elements — 200+ giao thức công nghiệp (OPC UA, Modbus, S7, MQTT) |
| **Đặc tính chủ đạo** | Khả năng mở rộng, Tính tương tác, Bảo trì |

**Quyết định kiến trúc quan trọng:**
- **"Đi theo sáng tạo của AWS"**: Chọn dịch vụ managed thay vì tự quản lý — giảm chi phí vận hành, tự động nhận cải tiến hạ tầng
- **Cung cấp cả Kinesis lẫn Kafka**: Khách hàng có yêu cầu quản trị khác nhau — managed (đơn giản) hoặc self-hosted (kiểm soát)
- **Microservices lỏng lẻo**: Mỗi chức năng (quản lý tài sản, identity, analytics, time-series) scale và cập nhật độc lập
- **MindConnect Elements**: Adapter phần cứng agnostic — kết nối bất kỳ thiết bị công nghiệp nào với nền tảng

**Kết quả — Triển khai Rittal Blue e+** *(nguồn: [AWS re:Invent 2019 MFG202](https://d1.awsstatic.com/events/reinvent/2019/Building_on_AWS_The_architecture_of_the_Siemens_MindSphere_platform_MFG202.pdf))*:
- Giảm **75%** tiêu thụ năng lượng của hệ thống làm mát kết nối mạng
- Giảm **75%** lượng carbon thải ra
- ROI dưới **3 tháng** từ bảo trì dự đoán
- **1,700+ đối tác** và ứng dụng trong hệ sinh thái sau 15+ năm phát triển

**Bài học kiến trúc:** *Tính tương tác giao thức mới là hào kinh tế thực sự* — khả năng kết nối 200+ giao thức công nghiệp khó sao chép hơn nhiều so với stack analytics trên đám mây. **Cung cấp lựa chọn** managed vs self-managed streaming cho phép đáp ứng yêu cầu quản trị khác nhau mà không phân nhánh kiến trúc.

---

### 4.4 Bài học kiến trúc từ thực tế

| Công ty | Đặc tính chủ đạo | Mẫu kiến trúc | Quyết định quan trọng nhất | Bài học |
|---|---|---|---|---|
| **Rolls-Royce** | Độ tin cậy, Quan sát | Edge-Cloud Hybrid + Digital Twin | MQTT qua vệ tinh; Azure Databricks | Kiến trúc IoT **là** sản phẩm, không phải tính năng |
| **John Deere** | Mở rộng, Tương tác | Multi-Cloud Hybrid + Microservices | Multi-cloud có chủ đích theo workload | Thiết kế cho tăng trưởng 10× từ ngày đầu |
| **Siemens** | Tương tác, Mở rộng | Microservices + EDA + Hybrid Cloud | 200+ giao thức qua MindConnect | Protocol interoperability = hào kinh tế thực sự |

**Ba nguyên tắc chung từ 3 case studies:**

1. **Đặc tính quyết định mẫu**: Mỗi công ty có 2–3 đặc tính ưu tiên rõ ràng — đặc tính đó dẫn trực tiếp đến lựa chọn kiến trúc
2. **Kết hợp mẫu trong thực tế**: Không có hệ thống nào chỉ dùng một mẫu — tất cả đều là hybrid của nhiều mẫu
3. **Kiến trúc tốt = mô hình kinh doanh mới**: Các dịch vụ như "Power by the Hour" (Rolls-Royce) hoặc nông nghiệp cấp cây trồng (John Deere) không thể tồn tại nếu không có kiến trúc IoT phù hợp

---

## 5. Kết luận

### Năm điểm cốt lõi

1. **IoT đặt ra thách thức kiến trúc khác biệt** — quy mô, tính không đồng nhất, tài nguyên hạn chế, vòng đời dài, kết nối không ổn định và bề mặt tấn công vật lý không tồn tại trong phần mềm web truyền thống.

2. **Xác định đặc tính kiến trúc trước khi chọn mẫu** — 8 đặc tính cốt lõi (Scalability, Reliability, Performance, Security, Maintainability, Interoperability, Observability, Energy Efficiency) là cầu nối giữa yêu cầu nghiệp vụ và lựa chọn cấu trúc.

3. **Không có mẫu kiến trúc nào phù hợp tất cả** — mỗi mẫu có điểm mạnh và điểm sụp đổ riêng trong bối cảnh IoT cụ thể; hệ thống thực tế luôn là hybrid.

4. **Thực tế xác nhận lý thuyết** — Rolls-Royce, John Deere và Siemens đều chứng minh rằng việc xác định đặc tính ưu tiên rõ ràng dẫn đến lựa chọn kiến trúc tốt và kết quả kinh doanh cụ thể.

5. **Tiến hóa theo nhu cầu, không thiết kế quá mức từ đầu** — bắt đầu với Layered/Serverless đơn giản, di chuyển sang Microservices + EDA khi đau đớn của mẫu hiện tại vượt chi phí di chuyển.

### Tài liệu tham khảo thêm

Để đọc sâu hơn, xem thư mục [docs/](docs/) trong repository này:
- [docs/06-architecture-characteristics.md](docs/06-architecture-characteristics.md) — Chi tiết 8 đặc tính kiến trúc
- [docs/02-patterns.md](docs/02-patterns.md) — Phân tích đánh đổi đầy đủ cho 5 mẫu kiến trúc
- [docs/05-case-studies.md](docs/05-case-studies.md) — Case studies Rolls-Royce, John Deere, Siemens phiên bản tiếng Anh
- [report.md](report.md) — Phiên bản báo cáo đầy đủ (12 phần, bao gồm protocols, security, edge computing)

---

> *Báo cáo rút gọn được biên soạn cho Seminar "Software Architecture for IoT" — Tháng 02/2026*
