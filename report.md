# 📋 BÁO CÁO CHI TIẾT: KIẾN TRÚC PHẦN MỀM TRONG IoT

> **Chủ đề**: Software Architecture for Internet of Things (IoT)
> **Ngày**: 28/02/2026
> **Tác giả**: Phong Nguyen Trung

---

## Mục lục

1. [Giới thiệu tổng quan về IoT](#1-giới-thiệu-tổng-quan-về-iot)
2. [Tại sao kiến trúc phần mềm quan trọng trong IoT](#2-tại-sao-kiến-trúc-phần-mềm-quan-trọng-trong-iot)
3. [Kiến trúc tham chiếu IoT 4 lớp](#3-kiến-trúc-tham-chiếu-iot-4-lớp)
4. [Các mẫu kiến trúc phần mềm cho IoT](#4-các-mẫu-kiến-trúc-phần-mềm-cho-iot)
5. [Giao thức truyền thông trong IoT](#5-giao-thức-truyền-thông-trong-iot)
6. [Bảo mật trong kiến trúc IoT](#6-bảo-mật-trong-kiến-trúc-iot)
7. [Điện toán biên (Edge Computing)](#7-điện-toán-biên-edge-computing)
8. [Xu hướng công nghệ mới](#8-xu-hướng-công-nghệ-mới)
9. [Nghiên cứu tình huống thực tế](#9-nghiên-cứu-tình-huống-thực-tế)
10. [Kết luận và khuyến nghị](#10-kết-luận-và-khuyến-nghị)
11. [Tài liệu tham khảo](#11-tài-liệu-tham-khảo)

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

---

## 2. Tại sao kiến trúc phần mềm quan trọng trong IoT

### 2.1 Các thách thức đặc thù của IoT

IoT khác biệt so với phần mềm truyền thống ở nhiều khía cạnh:

| Thách thức | Mô tả |
|---|---|
| **Tính không đồng nhất** | Hàng trăm loại thiết bị khác nhau về phần cứng, OS, giao thức |
| **Quy mô khổng lồ** | Hàng triệu thiết bị gửi dữ liệu liên tục |
| **Kết nối không ổn định** | Mạng không dây thường xuyên gián đoạn, độ trễ cao |
| **Yêu cầu thời gian thực** | Nhiều ứng dụng cần phản hồi trong mili-giây |
| **Tài nguyên hạn chế** | Thiết bị có CPU yếu, RAM ít (KB), pin giới hạn |
| **Bề mặt tấn công lớn** | Mỗi thiết bị là một điểm xâm nhập tiềm năng |
| **Vòng đời dài** | Thiết bị hoạt động 10-15 năm, cần cập nhật liên tục |

### 2.2 Hậu quả của kiến trúc kém

```
Kiến trúc kém × Số lượng thiết bị = Nợ kỹ thuật cấp số nhân
```

- **Không thể mở rộng** khi số thiết bị tăng từ 100 → 10,000 → 1,000,000
- **Sự cố lan truyền** — lỗi một thành phần ảnh hưởng toàn hệ thống
- **Chi phí vận hành tăng** theo thời gian thay vì giảm
- **Không thể cập nhật** thiết bị đã triển khai ngoài thực địa
- **Vi phạm bảo mật** do thiếu thiết kế phân tầng

### 2.3 Mục tiêu của kiến trúc IoT tốt

1. **Khả năng mở rộng (Scalability)**: Xử lý từ hàng trăm đến hàng triệu thiết bị
2. **Độ tin cậy (Reliability)**: Hoạt động liên tục 24/7 với khả năng tự phục hồi
3. **Bảo mật (Security)**: Bảo vệ dữ liệu từ thiết bị đến đám mây
4. **Khả năng tương tác (Interoperability)**: Tích hợp đa giao thức, đa nhà cung cấp
5. **Khả năng bảo trì (Maintainability)**: Dễ cập nhật, sửa lỗi, mở rộng tính năng

---

## 3. Kiến trúc tham chiếu IoT 4 lớp

### 3.1 Tổng quan mô hình

```
┌─────────────────────────────────────────────────────┐
│              LỚP ỨNG DỤNG (Application)             │
│  Dashboard │ Phân tích │ Logic nghiệp vụ │ API      │
├─────────────────────────────────────────────────────┤
│          LỚP XỬ LÝ BIÊN (Edge/Processing)           │
│  Gateway │ Edge Compute │ Lưu trữ cục bộ │ Rules    │
├─────────────────────────────────────────────────────┤
│              LỚP MẠNG (Network)                     │
│  WiFi │ 5G │ LoRaWAN │ BLE │ MQTT │ CoAP │ HTTP     │
├─────────────────────────────────────────────────────┤
│           LỚP CẢM NHẬN (Perception)                 │
│  Cảm biến │ Bộ truyền động │ Firmware │ MCU         │
└─────────────────────────────────────────────────────┘
```

### 3.2 Lớp Cảm nhận (Perception Layer)

**Vai trò**: Giao diện vật lý giữa thế giới số và thế giới thực.

#### Thành phần chính

| Thành phần | Vai trò | Ví dụ |
|---|---|---|
| **Cảm biến** | Đo lường hiện tượng vật lý | Nhiệt độ (DS18B20), độ ẩm (DHT22), GPS, gia tốc kế |
| **Bộ truyền động** | Thực hiện hành động vật lý | Motor, relay, van điện từ, màn hình LED |
| **Vi điều khiển** | Chạy firmware nhúng | ESP32, STM32, Arduino, Raspberry Pi Pico |
| **Hệ điều hành nhúng** | Quản lý tài nguyên thiết bị | FreeRTOS, Zephyr RTOS, Mbed OS |

#### Các yếu tố thiết kế quan trọng

- **Quản lý năng lượng**: Sử dụng chế độ ngủ sâu (deep sleep), chu kỳ hoạt động (duty cycling)
- **Tài nguyên hạn chế**: CPU (MHz), RAM (KB) — chọn giao thức nhẹ, tối ưu firmware
- **Bảo mật phần cứng**: TPM/HSM cho lưu trữ khóa an toàn, secure boot
- **Cập nhật OTA**: Thiết kế firmware hỗ trợ cập nhật qua mạng từ ngày đầu
- **Môi trường vật lý**: Chịu nhiệt, ẩm, rung, chuẩn IP rating phù hợp

#### Ví dụ: Giám sát nhiệt độ công nghiệp

```
[Cảm biến DS18B20] → [ESP32 MCU] → [WiFi/MQTT] → Gateway
                       │
                       ├── Đọc nhiệt độ mỗi 30 giây
                       ├── Áp dụng bộ lọc Kalman giảm nhiễu
                       ├── Publish MQTT: factory/zone1/temp
                       └── Deep sleep giữa các lần đọc
```

### 3.3 Lớp Mạng (Network Layer)

**Vai trò**: Xử lý truyền thông giữa thiết bị, gateway và dịch vụ đám mây.

#### Công nghệ kết nối

| Công nghệ | Phạm vi | Băng thông | Năng lượng | Phù hợp cho |
|---|---|---|---|---|
| **BLE 5.0** | 100m | 2 Mbps | Rất thấp | Thiết bị đeo, cận tiếp xúc |
| **Zigbee** | 100m (mesh) | 250 kbps | Thấp | Nhà thông minh, chiếu sáng |
| **WiFi 6** | 100m | 9.6 Gbps | Cao | Băng thông cao trong nhà |
| **LoRaWAN** | 15 km | 50 kbps | Rất thấp | Nông nghiệp, tiện ích |
| **NB-IoT** | 10 km | 250 kbps | Thấp | Theo dõi tài sản, đo lường |
| **5G (mMTC)** | 1 km | 10 Gbps | Trung bình | Xe tự lái, AR/VR |
| **Vệ tinh** | Toàn cầu | Khác nhau | Cao | Hàng hải, khai thác mỏ |

### 3.4 Lớp Xử lý Biên (Edge/Processing Layer)

**Vai trò**: Xử lý dữ liệu gần nguồn phát, giảm độ trễ, tiết kiệm băng thông, hỗ trợ hoạt động offline.

#### Thành phần

| Thành phần | Vai trò | Công nghệ |
|---|---|---|
| **IoT Gateway** | Chuyển đổi giao thức, quản lý thiết bị | AWS Greengrass, Azure IoT Edge |
| **Edge Server** | Tính toán và lưu trữ cục bộ | K3s, Docker, NVIDIA Jetson |
| **Stream Processor** | Xử lý dữ liệu thời gian thực | Apache Kafka, Node-RED |
| **CSDL cục bộ** | Lưu trữ chuỗi thời gian | InfluxDB, SQLite |
| **Rules Engine** | Ra quyết định tự động | Node-RED, custom engine |

#### Các mô hình xử lý biên

**1. Lọc và tổng hợp dữ liệu**
```
Dữ liệu thô (1000 bản ghi/giây)
  → Biên: Trung bình theo cửa sổ 1 phút
  → Đám mây: 1 bản ghi/phút (giảm 99.9%)
```

**2. Suy luận AI tại biên**
```
Camera stream (30 fps)
  → Biên: Chạy mô hình YOLO phát hiện đối tượng
  → Đám mây: Chỉ gửi sự kiện phát hiện (bất thường, cảnh báo)
```

**3. Store-and-Forward (Lưu và chuyển tiếp)**
```
Dữ liệu cảm biến khi mất mạng
  → Biên: Đệm trong SQLite cục bộ
  → Mạng phục hồi: Phát lại dữ liệu theo thứ tự thời gian
```

**4. Tự động hóa cục bộ**
```
Nhiệt độ đọc 95°C (ngưỡng: 90°C)
  → Biên: Kích hoạt hệ thống làm mát NGAY LẬP TỨC
  → Đám mây: Ghi log sự kiện (không khẩn cấp)
```

### 3.5 Lớp Ứng dụng (Application Layer)

**Vai trò**: Chuyển đổi dữ liệu IoT thô thành giá trị kinh doanh.

#### Pipeline dữ liệu điển hình

```
Thu thập → Xử lý → Lưu trữ → Phân tích → Trực quan hóa
   │          │         │          │            │
  MQTT     Stream    InfluxDB    ML/AI      Grafana
  HTTP    Processing  MongoDB   Rules      Mobile App
  AMQP     Batch     PostgreSQL Reports     APIs
```

#### Thành phần chính

| Thành phần | Vai trò | Công nghệ |
|---|---|---|
| Message Ingestion | Thu nhận dữ liệu thiết bị | AWS IoT Core, Azure IoT Hub, Kafka |
| Stream Processing | Xử lý thời gian thực | Apache Flink, Spark Streaming |
| Time-Series DB | Lưu telemetry | InfluxDB, TimescaleDB |
| ML Platform | Phân tích dự đoán | TensorFlow, PyTorch, SageMaker |
| Visualization | Dashboard và báo cáo | Grafana, Power BI, Kibana |

---

## 4. Các mẫu kiến trúc phần mềm cho IoT

### 4.1 Kiến trúc phân lớp (Layered Architecture)

**Mô tả**: Tổ chức hệ thống thành các lớp ngang, mỗi lớp có trách nhiệm riêng.

```
┌──────────────────────┐
│   Trình bày (UI)     │
├──────────────────────┤
│   Logic nghiệp vụ   │
├──────────────────────┤
│   Truy cập dữ liệu  │
├──────────────────────┤
│   Hạ tầng            │
└──────────────────────┘
```

| Ưu điểm | Nhược điểm |
|---|---|
| Đơn giản, dễ hiểu | Khó mở rộng độc lập |
| Phân tách trách nhiệm rõ ràng | Liên kết chặt giữa các lớp |
| Phù hợp dự án nhỏ (< 100 thiết bị) | Thay đổi lan truyền qua các lớp |

### 4.2 Kiến trúc Microservices

**Mô tả**: Phân tách hệ thống thành các dịch vụ nhỏ, độc lập, mỗi dịch vụ sở hữu một khả năng domain cụ thể.

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

#### Quyết định thiết kế chính

1. **Ranh giới dịch vụ**: Theo domain IoT (quản lý thiết bị, telemetry, analytics, OTA)
2. **Giao tiếp**: Bất đồng bộ (Kafka, MQTT) cho telemetry; đồng bộ (REST, gRPC) cho truy vấn
3. **Dữ liệu**: Mỗi dịch vụ sở hữu CSDL riêng — không chia sẻ database
4. **Triển khai**: Docker + Kubernetes cho containerization và orchestration

| Ưu điểm | Nhược điểm |
|---|---|
| Mở rộng độc lập từng dịch vụ | Phức tạp vận hành |
| Đa dạng công nghệ | Thách thức hệ thống phân tán |
| Cô lập lỗi | Cần DevOps trưởng thành |
| Phát triển song song | Chi phí giao tiếp giữa dịch vụ |

**Phù hợp**: Nền tảng IoT lớn (10K+ thiết bị), tổ chức đa nhóm.

### 4.3 Kiến trúc hướng sự kiện (Event-Driven Architecture — EDA)

**Mô tả**: Các thành phần giao tiếp bằng cách phát và tiêu thụ sự kiện. IoT vốn dĩ là hướng sự kiện — cảm biến liên tục tạo ra sự kiện.

```
Cảm biến ──publish──▶ [Event Broker] ──deliver──▶ Consumers
                      (Kafka/MQTT/RabbitMQ)       ├── Cảnh báo
                                                  ├── Lưu trữ
                                                  └── Phân tích
```

#### Các mẫu EDA trong IoT

| Mẫu | Mô tả | Ứng dụng |
|---|---|---|
| **Event Notification** | Thông báo nhẹ về sự kiện xảy ra | Cảnh báo vượt ngưỡng |
| **Event-Carried State** | Sự kiện mang đầy đủ trạng thái | Giảm callback, telemetry |
| **Event Sourcing** | Lưu tất cả sự kiện dưới dạng log bất biến | Audit trail, replay |
| **CQRS** | Tách đường ghi (commands) và đọc (queries) | Tối ưu ingestion + truy vấn |

| Ưu điểm | Nhược điểm |
|---|---|
| Phù hợp tự nhiên với IoT | Tính nhất quán cuối cùng |
| Liên kết lỏng lẻo | Khó debug (bất đồng bộ) |
| Dễ thêm consumer mới | Thứ tự message phức tạp |
| Khả năng mở rộng xuất sắc | Cần broker mạnh mẽ |

### 4.4 Kiến trúc Serverless

**Mô tả**: Logic ứng dụng chạy trong các hàm phi trạng thái, kích hoạt bởi sự kiện, được quản lý bởi nhà cung cấp đám mây.

```
[Thiết bị IoT] → [IoT Hub] → trigger → [Lambda/Function] → [DynamoDB]
                                │
                         if bất thường
                                ▼
                          [Lambda] → [SNS/Email thông báo]
```

| Ưu điểm | Nhược điểm |
|---|---|
| Không quản lý server | Cold start (100ms–2s) |
| Tự động scale đến 0 và đến hàng triệu | Giới hạn thời gian thực thi |
| Chỉ trả tiền khi sử dụng | Vendor lock-in |
| Tích hợp HA sẵn | Khó debug và test |

**Phù hợp**: Pipeline cảnh báo, biến đổi dữ liệu, workload không dự đoán được.

### 4.5 Kiến trúc lai Biên-Đám mây (Edge-Cloud Hybrid)

**Mô tả**: Phân phối xử lý giữa edge (gần thiết bị) và cloud. Đây là **mẫu kiến trúc chủ đạo cho IoT doanh nghiệp** từ 2025.

```
┌────────────── ĐÁM MÂY ──────────────┐
│  Huấn luyện ML │ Lưu trữ dài hạn    │
│  Quản lý fleet │ Dashboard & APIs    │
└──────────────────┬───────────────────┘
                   │ Đồng bộ
┌──────────────────┼───────────────────┐
│             BIÊN (EDGE)              │
│  Rules Engine │ Local DB │ AI Model  │
│  Gateway │ Protocol Bridge           │
└──────────────────┬───────────────────┘
                   │
        ┌──────────┼──────────┐
    [Cảm biến] [Camera] [Actuator]
```

#### Mô hình đồng bộ hóa

| Mô hình | Mô tả | Trường hợp sử dụng |
|---|---|---|
| **Store-and-Forward** | Đệm dữ liệu cục bộ, gửi khi có mạng | Kết nối không ổn định |
| **Eventual Consistency** | Biên và đám mây hội tụ theo thời gian | Telemetry không khẩn cấp |
| **Edge-First** | Biên ra quyết định, đám mây học từ đó | Hệ thống an toàn thời gian thực |
| **Bi-directional Sync** | Cả biên và đám mây đều có thể sửa đổi | Device twin/shadow |

### 4.6 Hướng dẫn chọn mẫu kiến trúc

```
                    Quy mô nhỏ           Quy mô lớn
                ┌──────────────────┬──────────────────┐
  Yêu cầu      │   Phân lớp +     │   Edge-Cloud     │
  độ trễ thấp   │   Edge Processing│   Hybrid +       │
                │                  │   Microservices   │
                ├──────────────────┼──────────────────┤
  Chấp nhận     │   Phân lớp       │   Microservices  │
  độ trễ cao    │   hoặc Serverless│   + EDA          │
                └──────────────────┴──────────────────┘
```

#### Câu hỏi quyết định

1. **Bao nhiêu thiết bị?** < 100 → Phân lớp; 100–10K → Microservices; > 10K → Microservices + EDA
2. **Yêu cầu độ trễ?** < 100ms → Cần Edge; > 1s → Cloud-only khả thi
3. **Kết nối ổn định?** Không → Edge-Cloud Hybrid với store-and-forward
4. **Quy mô đội ngũ?** Nhỏ → Serverless/Phân lớp; Lớn → Microservices
5. **Ngân sách?** Hạn chế → Serverless; Doanh nghiệp → Hybrid

---

## 5. Giao thức truyền thông trong IoT

### 5.1 MQTT — Tiêu chuẩn IoT

**Message Queuing Telemetry Transport** — giao thức Publish/Subscribe qua TCP, được thiết kế cho băng thông thấp và mạng không ổn định.

#### Kiến trúc MQTT
```
[Publisher/Cảm biến] ──publish("sensor/temp")──▶ [MQTT Broker]
                                                      │
[Subscriber/Dashboard] ◀──deliver("sensor/temp")──────┘
```

#### Đặc tính chính

| Đặc tính | Mô tả |
|---|---|
| **Publish/Subscribe** | Tách biệt producer và consumer |
| **QoS 0** | Gửi tối đa một lần (fire-and-forget) |
| **QoS 1** | Gửi ít nhất một lần (có thể trùng lặp) |
| **QoS 2** | Gửi chính xác một lần (đảm bảo cao nhất) |
| **Retained Message** | Broker lưu message cuối cho subscriber mới |
| **Last Will (LWT)** | Tự động publish khi client mất kết nối bất ngờ |
| **Topic phân cấp** | `factory/floor1/zone3/temperature` |
| **Wildcards** | `+` (một cấp), `#` (đa cấp) |

#### Thiết kế Topic

```
{tổ_chức}/{site}/{zone}/{loại_thiết_bị}/{id_thiết_bị}/{phép_đo}

# Ví dụ:
nha_may/phan_xuong_A/day_chuyen_1/robot/R-001/nhiet_do
nha_may/phan_xuong_A/day_chuyen_1/robot/R-001/trang_thai

# Topic lệnh (downlink):
cmd/{device_id}/cap_nhat_firmware
cmd/{device_id}/cau_hinh
```

#### Broker phổ biến
- **Mosquitto**: Mã nguồn mở, phù hợp phát triển
- **HiveMQ**: Thương mại, doanh nghiệp quy mô lớn
- **EMQX**: Mã nguồn mở, hiệu suất cao, clustering
- **AWS IoT Core / Azure IoT Hub**: Dịch vụ quản lý

### 5.2 Bảng so sánh giao thức

| Tiêu chí | MQTT | CoAP | AMQP | HTTP | WebSocket |
|---|---|---|---|---|---|
| **Tầng vận chuyển** | TCP | UDP | TCP | TCP | TCP |
| **Mô hình** | Pub/Sub | Req/Res | Pub/Sub+Queue | Req/Res | Hai chiều |
| **Header** | 2 bytes | 4 bytes | 8+ bytes | 100+ bytes | 2-14 bytes |
| **Năng lượng** | Thấp | Rất thấp | Trung bình | Cao | Trung bình |
| **Băng thông** | Rất thấp | Rất thấp | Trung bình | Cao | Thấp |
| **QoS** | 3 mức | Confirmable | Transaction | Không | Không |
| **Hỗ trợ offline** | Có | Không | Có | Không | Không |
| **Hỗ trợ trình duyệt** | Qua WebSocket | Không | Không | Native | Native |

### 5.3 Cây quyết định chọn giao thức

```
Thiết bị cực kỳ hạn chế (< 10KB RAM)?        → CoAP
Cần pub/sub + gửi tin cậy?                    → MQTT
Cần routing phức tạp + tích hợp doanh nghiệp? → AMQP
Dashboard web / UI thời gian thực?             → WebSocket
API quản lý thiết bị?                          → HTTP/REST
Mặc định?                                     → MQTT
```

### 5.4 Kiến trúc đa giao thức

Trong thực tế, hệ thống IoT thường sử dụng **nhiều giao thức** tại các điểm khác nhau:

```
Cảm biến(BLE) → Gateway(CoAP→MQTT) → MQTT Broker → Cloud
                                          ├── AMQP → ERP
                                          ├── HTTP API → Dashboard
                                          └── WebSocket → UI thời gian thực
```

---

## 6. Bảo mật trong kiến trúc IoT

### 6.1 Thách thức bảo mật IoT

- **Quy mô**: Hàng triệu thiết bị = hàng triệu điểm xâm nhập tiềm năng
- **Đa dạng**: Phần cứng, OS, firmware khác nhau
- **Hạn chế**: Thiết bị không thể chạy phần mềm bảo mật truyền thống
- **Tuổi thọ**: Thiết bị 10+ năm cần được cập nhật liên tục
- **Phơi bày vật lý**: Thiết bị ở môi trường công cộng

#### Sự cố bảo mật IoT nổi bật

| Sự cố | Tác động |
|---|---|
| **Mirai Botnet (2016)** | 600K+ thiết bị IoT bị chiếm dùng cho DDoS quy mô lớn |
| **Casino Fish Tank** | Hacker xâm nhập qua nhiệt kế bể cá thông minh để đánh cắp dữ liệu |
| **Stuxnet** | Tấn công SCADA công nghiệp vào máy ly tâm hạt nhân |

### 6.2 Bảo mật theo từng lớp

| Lớp | Mối đe dọa | Biện pháp giảm thiểu |
|---|---|---|
| **Cảm nhận** | Can thiệp vật lý, sao chép thiết bị | Secure boot, HSM, TPM |
| **Mạng** | Nghe lén, MITM, replay | TLS/DTLS, certificate pinning, mTLS |
| **Biên** | Truy cập trái phép, malware | Firewall, container isolation, signed firmware |
| **Ứng dụng** | Rò rỉ dữ liệu, injection, DDoS | OAuth 2.0, rate limiting, WAF, mã hóa |

### 6.3 Kiến trúc Zero Trust cho IoT

**Nguyên tắc: Không bao giờ tin tưởng, luôn xác minh — kể cả thiết bị nội bộ.**

1. **Danh tính thiết bị duy nhất**: Mỗi thiết bị có chứng chỉ X.509 hoặc token riêng
2. **Mutual TLS (mTLS)**: Cả thiết bị VÀ server đều xác thực lẫn nhau
3. **Kiểm soát truy cập cấp topic MQTT**:
   ```
   Thiết bị sensor-42:
     PUBLISH: factory/zone1/sensor-42/#     ✅
     SUBSCRIBE: cmd/sensor-42/#             ✅
     PUBLISH: factory/zone1/sensor-99/#     ❌
   ```
4. **Giám sát liên tục**: Phát hiện bất thường trong hành vi thiết bị
5. **Cập nhật OTA an toàn**: Ký mã (code signing), rollback, triển khai từng bước

### 6.4 Quyền riêng tư dữ liệu

| Quy định | Khu vực | Yêu cầu chính |
|---|---|---|
| **GDPR** | EU | Tối thiểu hóa dữ liệu, quyền xóa, đồng ý |
| **CCPA** | California | Quyền người tiêu dùng, từ chối bán dữ liệu |
| **HIPAA** | Y tế Mỹ | Bảo vệ PHI, audit trail |
| **Luật ATTT VN** | Việt Nam | Bảo vệ dữ liệu cá nhân, báo cáo sự cố |

#### Nguyên tắc Privacy-by-Design
1. **Tối thiểu hóa dữ liệu**: Chỉ thu thập những gì cần thiết
2. **Xử lý tại biên**: Giữ dữ liệu nhạy cảm tại chỗ
3. **Ẩn danh hóa**: Loại bỏ PII trước khi gửi lên đám mây
4. **Mã hóa**: End-to-end cho luồng dữ liệu nhạy cảm
5. **Chính sách lưu trữ**: Tự động xóa dữ liệu sau thời hạn

---

## 7. Điện toán biên (Edge Computing)

### 7.1 Tại sao Edge Computing là thiết yếu

Edge computing đã chuyển từ khái niệm bổ sung thành **thành phần cốt lõi** của kiến trúc IoT hiện đại.

| Lợi ích | Mô tả |
|---|---|
| **Giảm độ trễ** | Xử lý tại chỗ thay vì round-trip đến cloud (ms thay vì 100ms+) |
| **Tiết kiệm băng thông** | Xử lý 80% dữ liệu cục bộ, gửi 20% lên cloud |
| **Hoạt động offline** | Tiếp tục hoạt động khi mất kết nối internet |
| **Tuân thủ dữ liệu** | Giữ dữ liệu trong phạm vi địa lý (GDPR, luật pháp VN) |
| **Khả năng mở rộng** | Phân tán tải xử lý thay vì tập trung tại cloud |

### 7.2 Công nghệ Edge Computing

| Công nghệ | Loại | Mô tả |
|---|---|---|
| **AWS Greengrass** | Nền tảng | Chạy Lambda functions tại edge |
| **Azure IoT Edge** | Nền tảng | Container-based edge modules |
| **K3s** | Orchestration | Kubernetes nhẹ cho edge |
| **KubeEdge** | Orchestration | Kubernetes mở rộng cho edge |
| **NVIDIA Jetson** | Phần cứng | GPU-enabled edge AI |
| **Node-RED** | Low-code | Flow-based programming cho IoT |

### 7.3 AI tại biên (Edge AI)

```
Mô hình truyền thống:
  Thiết bị → Dữ liệu thô → Cloud → AI → Kết quả
  (độ trễ cao, tốn băng thông, phụ thuộc mạng)

Mô hình Edge AI:
  Thiết bị → Dữ liệu → Edge AI → Kết quả tức thì
                          │
                     Chỉ gửi metadata lên Cloud
  (độ trễ thấp, tiết kiệm, hoạt động offline)
```

#### Công nghệ Edge AI
- **TinyML**: Mô hình ML trên vi điều khiển (TensorFlow Lite Micro)
- **NVIDIA TensorRT**: Tối ưu inference trên GPU edge
- **ONNX Runtime**: Chạy đa nền tảng
- **OpenVINO**: Intel edge AI toolkit

---

## 8. Xu hướng công nghệ mới

### 8.1 AIoT (AI + IoT)

AI tích hợp trực tiếp vào hệ thống IoT, biến thiết bị từ **nguồn dữ liệu thụ động** thành **tác nhân thông minh tự chủ**.

- **Suy luận trên thiết bị**: Mô hình ML chạy trên MCU nhúng
- **Học liên kết (Federated Learning)**: Huấn luyện mô hình phân tán không cần tập trung dữ liệu
- **Ra quyết định tự chủ**: Thiết bị hoạt động không cần kết nối cloud

### 8.2 Digital Twins (Bản sao số)

- Bản sao ảo thời gian thực của tài sản vật lý
- Đồng bộ liên tục qua luồng dữ liệu IoT
- Mô phỏng kịch bản "what-if" trước khi thay đổi thực tế
- **Công cụ**: Azure Digital Twins, AWS IoT TwinMaker, NVIDIA Omniverse

### 8.3 5G + IoT

| Tính năng 5G | Ứng dụng IoT |
|---|---|
| **URLLC** (< 1ms latency) | Phẫu thuật từ xa, xe tự lái |
| **mMTC** (1M thiết bị/km²) | Triển khai cảm biến quy mô lớn |
| **Network Slicing** | Mạng ảo chuyên dụng cho mỗi ứng dụng IoT |

### 8.4 IoT bền vững (Green IoT)

- Thu hoạch năng lượng (solar, thermal, kinetic) cho cảm biến không pin
- Tối ưu duty cycling và chế độ ngủ
- Carbon-aware computing: dịch chuyển workload đến vùng năng lượng sạch
- Thiết kế thiết bị có tuổi thọ 10+ năm

### 8.5 Matter Protocol

- Tiêu chuẩn thống nhất cho nhà thông minh
- Hỗ trợ bởi Apple, Google, Amazon, Samsung
- Xây dựng trên Thread (mesh) và WiFi
- Một thiết bị hoạt động với tất cả hệ sinh thái

### 8.6 IoT vệ tinh

- Kết nối trực tiếp thiết bị-vệ tinh cho tài sản ở xa
- Ứng dụng: hàng hải, nông nghiệp, dầu khí, lâm nghiệp
- Nhà cung cấp: Swarm (SpaceX), Astrocast, Myriota

---

## 9. Nghiên cứu tình huống thực tế

### 9.1 Nhà máy thông minh (Industry 4.0)

| Yếu tố | Chi tiết |
|---|---|
| **Bối cảnh** | Nhà máy sản xuất với 500+ cảm biến |
| **Kiến trúc** | Edge-Cloud Hybrid + EDA + Microservices |
| **Giao thức** | MQTT (QoS 1) cho telemetry |
| **Edge** | K3s trên NVIDIA Jetson — AI phát hiện lỗi |
| **CSDL** | InfluxDB cho chuỗi thời gian |
| **Dashboard** | Grafana thời gian thực |

**Kết quả**:
- Giảm **30% thời gian ngừng máy** nhờ bảo trì dự đoán
- Tiết kiệm **$2M/năm** từ phát hiện lỗi sớm
- **99.9%** độ tin cậy gửi dữ liệu qua MQTT QoS 1

### 9.2 Quản lý giao thông thành phố thông minh

| Yếu tố | Chi tiết |
|---|---|
| **Bối cảnh** | Tối ưu giao thông toàn thành phố |
| **Kiến trúc** | Edge-First + Event-Driven |
| **Edge AI** | NVIDIA Jetson + TensorRT — xử lý video tại nút giao |
| **Giao thức** | MQTT cho sự kiện, HTTP cho cấu hình |

**Kết quả**:
- Cải thiện **20% lưu lượng giao thông**
- Giảm **15% khí thải** tại các nút giao được giám sát
- **Tuân thủ quyền riêng tư**: Video xử lý cục bộ, chỉ gửi metadata

### 9.3 Y tế kết nối

| Yếu tố | Chi tiết |
|---|---|
| **Bối cảnh** | Giám sát bệnh nhân liên tục bằng thiết bị đeo |
| **Kiến trúc** | Serverless + Event-Driven |
| **Bảo mật** | mTLS + AES-256 + tuân thủ HIPAA |
| **Cảnh báo** | AWS Lambda (serverless) |
| **Tích hợp EHR** | AMQP + HL7 FHIR |

**Kết quả**:
- Phản ứng khẩn cấp **nhanh hơn 40%**
- Giám sát **liên tục** thay cho kiểm tra thủ công 4 giờ/lần
- **Tuân thủ HIPAA** hoàn toàn với audit trail

---

## 10. Kết luận và khuyến nghị

### 10.1 Tóm tắt các điểm chính

1. **Kiến trúc phân lớp** cung cấp phân tách trách nhiệm rõ ràng — hiểu vai trò của từng lớp
2. **Chọn mẫu kiến trúc dựa trên yêu cầu**, không phải xu hướng — sử dụng framework chọn lựa
3. **MQTT là tiêu chuẩn thực tế** cho IoT messaging — bắt đầu từ đây trừ khi có ràng buộc đặc biệt
4. **Edge computing là thiết yếu**, không phải tùy chọn — xử lý cục bộ, đồng bộ toàn cục
5. **Bảo mật phải được tích hợp từ đầu** — Zero Trust từ thiết bị đến đám mây
6. **AIoT và Digital Twins** đang định hình lại khả năng của IoT
7. **Không có mẫu nào phù hợp tất cả** — kết hợp các mẫu cho nhu cầu cụ thể

### 10.2 Khuyến nghị cho tổ chức

| Quy mô | Khuyến nghị kiến trúc |
|---|---|
| **Startup / POC** | Phân lớp + MQTT + Serverless |
| **SME (100–1K thiết bị)** | Microservices + MQTT + Edge gateway |
| **Doanh nghiệp (10K+)** | Edge-Cloud Hybrid + Microservices + EDA |
| **IoT công nghiệp** | Edge-First + AI at Edge + MQTT + Digital Twins |

### 10.3 Lộ trình triển khai đề xuất

```
Giai đoạn 1 (0-3 tháng): Foundation
  → Chọn giao thức (MQTT), thiết lập broker
  → Kết nối 10-50 thiết bị pilot
  → Dashboard cơ bản (Grafana)

Giai đoạn 2 (3-6 tháng): Scale
  → Triển khai Edge gateway
  → Microservices cho backend
  → Bảo mật: mTLS, RBAC

Giai đoạn 3 (6-12 tháng): Intelligence
  → Edge AI cho suy luận cục bộ
  → Analytics pipeline
  → Digital Twin cho tài sản quan trọng

Giai đoạn 4 (12+ tháng): Optimize
  → Tối ưu chi phí và hiệu suất
  → Mở rộng đến hàng nghìn thiết bị
  → Tích hợp AIoT nâng cao
```

---

## 11. Tài liệu tham khảo

### Sách
- *Designing IoT Solutions with Microsoft Azure* (Packt Publishing)
- *IoT and Edge Computing for Architects* (Packt Publishing)
- *Building the Web of Things* (Manning Publications)
- *MQTT Essentials* (HiveMQ)

### Tham chiếu kiến trúc
- [AWS IoT Lens — Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/iot-lens/welcome.html)
- [Azure IoT Reference Architecture](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/iot)
- [Google Cloud IoT Architecture](https://cloud.google.com/architecture/connected-devices)

### Tiêu chuẩn và đặc tả
- [MQTT v5.0 Specification (OASIS)](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html)
- [CoAP RFC 7252 (IETF)](https://tools.ietf.org/html/rfc7252)
- [NIST Cyber-Physical Systems Framework](https://www.nist.gov/el/cyber-physical-systems)

### Cộng đồng và tài nguyên trực tuyến
- [Eclipse IoT Working Group](https://iot.eclipse.org/)
- [HiveMQ MQTT Guides](https://www.hivemq.com/mqtt/)
- [IoT For All](https://www.iotforall.com/)
- [MQTT.org](https://mqtt.org/)

---

> *Báo cáo được biên soạn cho Seminar "Software Architecture for IoT" — Tháng 02/2026*
