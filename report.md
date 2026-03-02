# 📋 BÁO CÁO CHI TIẾT: KIẾN TRÚC PHẦN MỀM TRONG IoT

> **Chủ đề**: Software Architecture for Internet of Things (IoT)
> **Ngày**: 28/02/2026
> **Tác giả**: Phong Nguyen Trung

---

## Mục lục

1. [Giới thiệu tổng quan về IoT](#1-giới-thiệu-tổng-quan-về-iot)
2. [Tại sao kiến trúc phần mềm quan trọng trong IoT](#2-tại-sao-kiến-trúc-phần-mềm-quan-trọng-trong-iot)
3. [Đặc tính kiến trúc (Architecture Characteristics) trong IoT](#3-đặc-tính-kiến-trúc-architecture-characteristics-trong-iot)
4. [Kiến trúc tham chiếu IoT 4 lớp](#4-kiến-trúc-tham-chiếu-iot-4-lớp)
5. [Các mẫu kiến trúc phần mềm cho IoT](#5-các-mẫu-kiến-trúc-phần-mềm-cho-iot)
6. [Giao thức truyền thông trong IoT](#6-giao-thức-truyền-thông-trong-iot)
7. [Bảo mật trong kiến trúc IoT](#7-bảo-mật-trong-kiến-trúc-iot)
8. [Điện toán biên (Edge Computing)](#8-điện-toán-biên-edge-computing)
9. [Xu hướng công nghệ mới](#9-xu-hướng-công-nghệ-mới)
10. [Nghiên cứu tình huống thực tế](#10-nghiên-cứu-tình-huống-thực-tế)
11. [Kết luận và khuyến nghị](#11-kết-luận-và-khuyến-nghị)
12. [Tài liệu tham khảo](#12-tài-liệu-tham-khảo)

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

Kiến trúc IoT tốt cần đạt 8 **đặc tính kiến trúc** (architecture characteristics) cốt lõi — mỗi đặc tính đặt ra yêu cầu cụ thể về cấu trúc hệ thống và ảnh hưởng trực tiếp đến lựa chọn mẫu kiến trúc:

| # | Đặc tính kiến trúc | Mục tiêu tóm tắt |
|---|---|---|
| 1 | **Khả năng mở rộng (Scalability)** | Xử lý từ hàng trăm đến hàng triệu thiết bị |
| 2 | **Độ tin cậy (Reliability)** | Hoạt động liên tục 24/7, kể cả khi mất kết nối đám mây |
| 3 | **Hiệu năng (Performance)** | Đáp ứng yêu cầu độ trễ và thông lượng thực tế |
| 4 | **Bảo mật (Security)** | Bảo vệ dữ liệu từ thiết bị đến đám mây |
| 5 | **Khả năng bảo trì (Maintainability)** | Cập nhật OTA cho thiết bị có vòng đời 10+ năm |
| 6 | **Khả năng tương tác (Interoperability)** | Tích hợp đa giao thức, đa nhà cung cấp |
| 7 | **Khả năng quan sát (Observability)** | Giám sát hàng triệu thiết bị không thể tiếp cận vật lý |
| 8 | **Hiệu quả năng lượng (Energy Efficiency)** | Thiết bị pin hoạt động nhiều năm |

> Phần 3 sẽ phân tích chi tiết từng đặc tính này — cách đo lường, đánh đổi, và mối liên hệ với lựa chọn mẫu kiến trúc.

---

## 3. Đặc tính kiến trúc (Architecture Characteristics) trong IoT

### 3.1 Khái niệm đặc tính kiến trúc

Trong thiết kế phần mềm, chúng ta phân biệt hai loại yêu cầu:

- **Yêu cầu chức năng (Functional Requirements):** Hệ thống *làm gì*? — "Thu thập nhiệt độ mỗi 30 giây", "Cảnh báo khi áp suất vượt ngưỡng".
- **Đặc tính kiến trúc (Architecture Characteristics):** Hệ thống phải *giỏi điều gì*? — "Xử lý 1 triệu kết nối đồng thời", "Phản hồi trong vòng 10ms".

**Đặc tính kiến trúc** (còn gọi là *thuộc tính chất lượng* hoặc *yêu cầu phi chức năng*) là các thuộc tính đo lường được của hệ thống, buộc kiến trúc sư phải đưa ra quyết định cấu trúc cụ thể. Chúng không mô tả tính năng — chúng mô tả *mức độ phù hợp* của hệ thống.

Quy trình thiết kế kiến trúc đúng đắn:

```
Yêu cầu nghiệp vụ IoT
         ↓
Đặc tính kiến trúc  ←── Phần 3 (phần này)
         ↓
Lựa chọn mẫu kiến trúc  ←── Phần 5
         ↓
Triển khai công nghệ
```

**Tại sao đặc tính IoT khác với phần mềm truyền thống?**

Hệ thống IoT vận hành đồng thời trên 3 tầng: thiết bị vật lý, điện toán biên, và đám mây. Mỗi tầng tạo ra áp lực đặc tính riêng mà ứng dụng web không có:

| Tầng | Áp lực đặc tính đặc thù |
|---|---|
| Thiết bị | Ràng buộc năng lượng, bề mặt tấn công vật lý, vòng đời 10+ năm |
| Biên | Hoạt động offline, tự động cục bộ, phần cứng hạn chế tài nguyên |
| Đám mây | Số lượng kết nối khổng lồ, định dạng dữ liệu không đồng nhất, quản lý đội thiết bị |

---

### 3.2 Các đặc tính kiến trúc cốt lõi của IoT

#### 3.2.1 Khả năng mở rộng (Scalability)

**Định nghĩa:** Khả năng hệ thống xử lý sự tăng trưởng về số thiết bị, khối lượng dữ liệu và thao tác quản lý mà không cần tái kiến trúc.

**Điểm khác biệt với web:** Web mở rộng một chiều — số người dùng đồng thời. IoT mở rộng hai chiều độc lập:
- **Mở rộng kết nối:** 100 thiết bị → 100,000 → 10 triệu kết nối MQTT đồng thời
- **Mở rộng thông lượng:** 1 đọc/thiết bị/phút → 100 đọc/thiết bị/giây (chênh lệch 7 bậc độ lớn)
- **Mở rộng quản lý:** Cập nhật OTA, đẩy cấu hình, giám sát cho đội thiết bị ngày càng lớn

| Quy mô | Thiết bị | Sự kiện/giây | Kiến trúc gợi ý |
|---|---|---|---|
| POC | < 1,000 | < 1,000 | Layered monolith |
| SME | 1,000–100,000 | 1,000–100,000 | Microservices + MQTT broker |
| Enterprise | 100K–10M | 100K–1M+ | Microservices + EDA |
| Industrial | 10M+ | 10M+ | Edge-Cloud Hybrid + broker phân tán |

> **Mẫu kiến trúc liên quan:** Đây là lý do chính để chuyển từ Layered sang **Event-Driven Architecture** và **Microservices** (Phần 5).

---

#### 3.2.2 Độ tin cậy và khả dụng (Reliability & Availability)

**Định nghĩa:** Xác suất hệ thống cung cấp dịch vụ đúng như thiết kế — kể cả ở tầng thiết bị khi mất kết nối mạng hoặc đám mây.

**Hai chiều độ tin cậy trong IoT:**
- **Khả dụng nền tảng:** Uptime của dịch vụ cloud (99.9% = 8.7 giờ downtime/năm; 99.99% = 52 phút/năm)
- **Tự chủ thiết bị:** Thiết bị phải tiếp tục chức năng chính khi bị ngắt kết nối đám mây

Dây chuyền lắp ráp nhà máy không thể dừng vì AWS có sự cố 12 phút. Màn hình bệnh nhân phải cảnh báo cục bộ dù mạng di động không khả dụng.

| Cơ chế | Mô tả | Tác động độ tin cậy |
|---|---|---|
| MQTT QoS 0 | Gửi tối đa 1 lần | Có thể mất tin nhắn |
| MQTT QoS 1 | Gửi ít nhất 1 lần | Đảm bảo giao hàng, có thể trùng lặp |
| MQTT QoS 2 | Đúng 1 lần | Đảm bảo cao nhất, chi phí cao nhất |
| MQTT LWT | Last Will Testament | Phát hiện thiết bị mất kết nối đáng tin cậy |
| Store-and-Forward | Đệm tin nhắn khi mất kết nối | Không mất dữ liệu khi gián đoạn kết nối |

> **Mẫu kiến trúc liên quan:** Yêu cầu tự chủ offline là lý do chính chọn **Edge-Cloud Hybrid** — node biên duy trì rules engine cục bộ và tiếp tục hoạt động khi không có đám mây.

---

#### 3.2.3 Hiệu năng — Độ trễ và thông lượng (Performance)

**Định nghĩa:** Độ trễ là thời gian từ sự kiện đến phản hồi. Thông lượng là khối lượng sự kiện hệ thống xử lý được trong một đơn vị thời gian.

**Tại sao IoT có deadline cứng:** Trang web tải 3 giây chỉ là khó chịu. Hệ thống làm lạnh mất 3 giây phản hồi khi quá nhiệt có thể phá hủy thiết bị. IoT ghép phần mềm với quy trình vật lý có yêu cầu thời gian riêng.

**Mô hình 5 tầng độ trễ IoT:**

| Tầng độ trễ | Mục tiêu | Vị trí xử lý | Ví dụ |
|---|---|---|---|
| Thời gian thực cứng | < 1 ms | Firmware thiết bị | Vòng điều khiển PID motor |
| Thời gian thực mềm | < 10 ms | Edge gateway | Ngắt khẩn cấp an toàn |
| Gần thời gian thực | < 100 ms | Edge compute | Cảnh báo phát hiện bất thường |
| Tương tác | < 1 giây | Cloud | Làm mới dashboard |
| Batch/Analytics | Phút–giờ | Cloud | Huấn luyện mô hình ML |

> **Mẫu kiến trúc liên quan:** Yêu cầu độ trễ < 100ms là yếu tố phân biệt quan trọng nhất giữa **Edge-Cloud Hybrid** và kiến trúc cloud thuần túy.

---

#### 3.2.4 Bảo mật (Security)

**Định nghĩa:** Mức độ hệ thống bảo vệ thông tin và tài nguyên khỏi truy cập, sửa đổi hoặc tiết lộ trái phép — kể cả ở tầng thiết bị vật lý.

**Tại sao bảo mật IoT là đặc tính cấu trúc:** Trong phần mềm web, kiểm soát bảo mật có thể được thêm vào sau khi xây dựng xong. Trong IoT, điều này không thể. Các quyết định tại thời điểm sản xuất thiết bị (secure boot, danh tính phần cứng, lưu trữ mã hóa) không thể thay đổi ngoài thực địa.

Bề mặt tấn công vật lý: Bất kỳ thiết bị nào cũng có thể bị tháo rời, dump firmware và nhân bản.

**Bảo mật tương tác với mọi đặc tính khác:**
- Bảo mật ↔ Hiệu năng: TLS thêm 50–150ms độ trễ handshake
- Bảo mật ↔ Năng lượng: AES-256 tăng chu kỳ CPU và tiêu thụ điện
- Bảo mật ↔ Bảo trì: Xoay vòng chứng chỉ phải được tự động hóa ở quy mô lớn

Chi tiết xem tại [docs/04-security.md](docs/04-security.md).

---

#### 3.2.5 Khả năng bảo trì (Maintainability)

**Định nghĩa:** Sự dễ dàng trong việc sửa đổi, cập nhật, gỡ lỗi và mở rộng hệ thống — kể cả thiết bị vật lý — trong suốt vòng đời hoạt động.

**Tại sao bảo trì IoT là yêu cầu sống còn:** Phần mềm web có thể cập nhật bằng cách deploy code server mới trong vài giây. Thiết bị IoT có thể được triển khai trên sàn nhà máy, đồng ruộng hẻo lánh, hạ tầng nhúng, hoặc cấy ghép vào bệnh nhân. Chúng không thể tiếp cận vật lý.

Thiết bị xuất xưởng năm 2026 chạy pin 5 năm phải nhận bản vá bảo mật đến năm 2031.

| Chiều bảo trì | Cơ chế |
|---|---|
| Cập nhật firmware | OTA qua phân vùng A/B, delta update, staged rollout |
| Cấu hình | Device twin/shadow (desired state vs reported state) |
| Giám sát | Heartbeat message, LWT, telemetry định kỳ |
| Tương thích ngược | API versioning, lớp trừu tượng giao thức |
| Vòng đời thiết bị | Device registry với state machine |

> **Mẫu kiến trúc liên quan:** Cập nhật OTA quy mô lớn cần **OTA Update Service** riêng biệt (Microservices) và hạ tầng phân phối biên (Edge-Cloud Hybrid).

---

#### 3.2.6 Khả năng tương tác (Interoperability)

**Định nghĩa:** Khả năng hệ thống giao tiếp và trao đổi dữ liệu có ý nghĩa với thiết bị, giao thức, tiêu chuẩn và hệ thống bên ngoài không đồng nhất.

**Tại sao IoT phức tạp hơn:** Không ứng dụng web nào phải đồng thời nói HTTP, MQTT, CoAP, BLE, Zigbee, LoRaWAN, Modbus và OPC-UA. IoT thừa hưởng sự đa dạng giao thức của thế giới vật lý.

**Ba cấp độ tương tác:**

- **Giao thức:** BLE → MQTT, CoAP → MQTT, Modbus → MQTT, AMQP → ERP
- **Mô hình dữ liệu:** JSON, CBOR, Protocol Buffers, định dạng nhị phân độc quyền
- **Tiêu chuẩn ngành:** Matter (nhà thông minh), OPC-UA (công nghiệp), HL7 FHIR (y tế), oneM2M

> **Mẫu kiến trúc liên quan:** Tương tác thúc đẩy **Event-Driven Architecture** với event bus trung tâm (tách biệt producer đa giao thức với consumer) và lớp gateway trong **Edge-Cloud Hybrid** (thực hiện chuyển đổi giao thức trước khi sự kiện đến đám mây).

---

#### 3.2.7 Khả năng quan sát (Observability)

**Định nghĩa:** Mức độ trạng thái nội bộ của hệ thống có thể được suy ra từ đầu ra bên ngoài — cho phép vận hành viên hiểu, chẩn đoán và dự đoán hành vi hệ thống mà không cần tiếp cận vật lý thiết bị.

**Tại sao IoT khó quan sát hơn web:** Không thể SSH vào cảm biến nhiệt độ. Không thể thêm câu lệnh debug vào firmware đang chạy ngoài thực địa. Nếu thiết bị hoạt động sai, chỉ có dữ liệu nó chọn gửi để chẩn đoán. Khả năng quan sát phải được **thiết kế sẵn từ đầu**.

**Ba trụ cột quan sát trong IoT:**

| Trụ cột | Triển khai trong IoT |
|---|---|
| **Metrics** | Sức khỏe thiết bị (pin, RSSI, uptime), thông lượng nền tảng, chỉ số nghiệp vụ |
| **Logs** | Log gateway biên, log service ứng dụng, log kiểm toán bảo mật — cấu trúc JSON, tương quan theo device ID |
| **Traces** | Đường dẫn end-to-end từ thiết bị publish → broker → ingestion → storage → dashboard |

**Patterns quan sát đặc thù IoT:**
- **Heartbeat message:** Thiết bị publish tin nhắn sức khỏe theo lịch; thiếu heartbeat kích hoạt cảnh báo
- **Device shadow/twin:** Chế độ xem đám mây về trạng thái được báo cáo gần nhất của mỗi thiết bị
- **LWT (Last Will Testament):** Broker publish thông báo "thiết bị offline" khi kết nối bị ngắt đột ngột

> **Mẫu kiến trúc liên quan:** Quan sát cao cấp ưu tiên **Event-Driven Architecture** (mỗi sự kiện là đơn vị có cấu trúc, có thể trace) và **Microservices** (mỗi service phát metrics và logs độc lập).

---

#### 3.2.8 Hiệu quả năng lượng (Energy Efficiency)

**Định nghĩa:** Khả năng hệ thống — đặc biệt ở tầng thiết bị — hoàn thành chức năng trong khi tiêu thụ điện năng tối thiểu, cho phép hoạt động bằng pin trong nhiều tháng hoặc nhiều năm.

**Tại sao không có analog trong phần mềm enterprise:** Không server web nào chạy bằng hai pin AA. Năng lượng là ràng buộc kiến trúc cơ bản cho mọi hệ thống IoT có thiết bị chạy pin.

**Mô hình ngân sách điện:**
```
Tuổi thọ pin (giờ) = Dung lượng pin (mAh) / Dòng điện trung bình (mA)

Pin AA: 2,500 mAh
TX tích cực (WiFi): 250 mA → 10 giờ
TX tích cực (LoRaWAN): 20 mA → 125 giờ
Ngủ sâu: 0.01 mA → 25,000 giờ (2.8 năm)
```

**Với duty cycling** (TX 100ms mỗi 30 giây):
- Dòng trung bình ≈ 0.84 mA → Tuổi thọ pin ≈ **124 ngày**

| Mọi quyết định kiến trúc ảnh hưởng đến năng lượng |
|---|
| Mức QoS: QoS 2 (4 tin nhắn/giao dịch) vs QoS 0 (1 tin nhắn) — 4× thời gian radio |
| Định dạng payload: JSON (văn bản, dài) vs CBOR (nhị phân, ngắn) |
| Tần suất gửi: mỗi 1 giây vs mỗi 1 phút — ảnh hưởng 60× đến tuổi thọ pin |
| Tổng hợp tại biên: gateway tổng hợp 60 đọc và gửi 1 — kéo dài pin thiết bị 60× |

> **Mẫu kiến trúc liên quan:** Ràng buộc năng lượng thường quyết định lựa chọn giao thức (LoRaWAN thay WiFi), mẫu kiến trúc (Edge-Cloud Hybrid thay cloud thuần túy — ít round-trip hơn mỗi thiết bị), và định dạng dữ liệu (CBOR/Protobuf thay JSON).

---

### 3.3 Đánh đổi giữa các đặc tính

Trong thực tế, không thể tối ưu tất cả đặc tính đồng thời. Kiến trúc sư phần mềm phải ưu tiên và chấp nhận **đánh đổi có chủ ý**:

| Đặc tính A | ↔ | Đặc tính B | Xung đột | Chiến lược giải quyết |
|---|---|---|---|---|
| **Bảo mật** (TLS/mTLS) | | **Hiệu năng** (độ trễ) | TLS thêm 50–150ms overhead | Hardware crypto (ESP32 AES-NI), TLS session resumption |
| **Bảo mật** (AES-256) | | **Năng lượng** | Mã hóa tăng chu kỳ CPU | Dùng crypto coprocessor (TPM, ATECC608A) |
| **Độ tin cậy** (QoS 2) | | **Hiệu năng** (thông lượng) | QoS 2 cần 4-message handshake | Dùng QoS 1 cho telemetry, QoS 2 cho lệnh quan trọng |
| **Quan sát** (ghi log liên tục) | | **Năng lượng** | Ghi log cần truyền radio | Batch log; gửi theo cửa sổ TX lên lịch |
| **Mở rộng** (microservices) | | **Bảo trì** (độ phức tạp vận hành) | Nhiều service hơn = nhiều deployment | Đầu tư vào Kubernetes, GitOps ngay từ đầu |
| **Năng lượng** (ngủ sâu) | | **Quan sát** (giám sát thời gian thực) | Thiết bị ngủ không thể gửi metrics | Wake-on-interrupt cho sự kiện quan trọng; beacon định kỳ |

> "Kiến trúc tốt không phải tối đa hóa tất cả đặc tính — mà là thực hiện đánh đổi có chủ ý, phù hợp với ưu tiên nghiệp vụ."

---

### 3.4 Đặc tính kiến trúc dẫn dắt lựa chọn mẫu

Đây là cầu nối giữa phần này và Phần 5 (Mẫu kiến trúc). Xác định 2–3 đặc tính ưu tiên hàng đầu và dùng bảng này để thu hẹp không gian mẫu phù hợp:

| Đặc tính ưu tiên cao | Mẫu kiến trúc khởi điểm | Lý do |
|---|---|---|
| **Hiệu năng** < 100ms | Edge-Cloud Hybrid | Đám mây không thể nằm trong đường xử lý tới hạn |
| **Mở rộng** > 100K thiết bị | Microservices + EDA | Mở rộng ngang độc lập; broker hiệu năng cao |
| **Độ tin cậy** (tự chủ offline) | Edge-Cloud Hybrid | Rules engine cục bộ hoạt động không cần đám mây |
| **Năng lượng** (chạy pin) | EDA + MQTT QoS thấp | Thời gian thức tối thiểu mỗi sự kiện; không polling |
| **Bảo mật** (ngành có quy định) | Bất kỳ mẫu nào + Zero Trust overlay | Bảo mật xuyên suốt; mTLS, PKI, topic ACL |
| **Bảo trì** (đội thiết bị lớn, vòng đời dài) | Microservices + Edge | Giao OTA qua biên; service độc lập để deploy |
| **Tương tác** (tích hợp enterprise) | EDA + AMQP bridge | Event bus tách biệt producer đa giao thức với consumer |
| **Quan sát** (giám sát quan trọng) | EDA + Microservices | Sự kiện có cấu trúc hỗ trợ distributed tracing |

**Ví dụ thực tế:** Nhà máy thông minh trong Phần 10 có yêu cầu cao ở tất cả đặc tính — đây là lý do kết hợp cả 4 mẫu: Edge-Cloud Hybrid + Microservices + EDA + Zero Trust.

---

### 3.5 Kết nối với các phần tiếp theo

| Bước tiếp theo | Phần | Nội dung |
|---|---|---|
| Cấu trúc hệ thống theo lớp | **Phần 4** | Kiến trúc tham chiếu IoT 4 lớp |
| Lựa chọn mẫu theo đặc tính | **Phần 5** | Các mẫu kiến trúc và ma trận lựa chọn |
| Đặc tính bảo mật chi tiết | **Phần 7** | Kiến trúc Zero Trust và mô hình mối đe dọa |
| Đặc tính trong thực tiễn | **Phần 10** | Nghiên cứu tình huống — xem đặc tính nào dẫn dắt quyết định |

---

## 4. Kiến trúc tham chiếu IoT 4 lớp

### 4.1 Tổng quan mô hình

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

### 4.2 Lớp Cảm nhận (Perception Layer)

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

### 4.3 Lớp Mạng (Network Layer)

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

### 4.4 Lớp Xử lý Biên (Edge/Processing Layer)

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

### 4.5 Lớp Ứng dụng (Application Layer)

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

## 5. Các mẫu kiến trúc phần mềm cho IoT

### 5.1 Kiến trúc phân lớp (Layered Architecture)

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

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Độ phức tạp** | Đơn giản, dễ hiểu và triển khai | "Quả bóng bùn lớn" khi hệ thống tăng trưởng | ✅ Lý tưởng cho POC; ❌ sụp đổ khi thiết bị vượt ~1K |
| **Khớp nối** | Phân tách trách nhiệm trong một lớp | Liên kết chặt *giữa* các lớp — thay đổi lan truyền | ❌ Thêm loại cảm biến mới đòi hỏi sửa đổi Perception, Network và Application đồng thời |
| **Khả năng mở rộng** | Dễ suy luận | Không thể mở rộng độc lập từng lớp | ❌ Nếu ingestion là nút thắt cổ chai, phải scale *toàn bộ* ứng dụng |
| **Đội ngũ** | Rào cản gia nhập thấp | Không phù hợp cho nhiều nhóm phát triển song song | ✅ Đúng cho 1–3 người; ❌ gây xung đột merge cho nhóm lớn hơn |

**Khi nào Layered sụp đổ trong IoT:** Số thiết bị vượt ~1,000; yêu cầu độ trễ < 100ms; nhiều nhóm cần làm việc song song.

### 5.2 Kiến trúc Microservices

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

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Khả năng mở rộng** | Mở rộng từng dịch vụ độc lập | Nhiều hạ tầng hơn cần quản lý | ✅ Telemetry Ingestion có thể scale lên 10M msg/s trong khi OTA Service ở mức tối thiểu |
| **Cô lập lỗi** | Lỗi một dịch vụ không lan truyền | Cần circuit breakers và timeout giữa dịch vụ | ✅ Analytics crash không ảnh hưởng kết nối thiết bị và cảnh báo |
| **Vận hành** | Mỗi dịch vụ độc lập triển khai | Kubernetes + service mesh + observability tốn kém | ❌ Nhóm nhỏ dành 40–60% công sức cho vận hành thay vì tính năng |
| **Độ trễ** | Dịch vụ tối ưu cho mục đích riêng | Network hop giữa dịch vụ cộng thêm 10–30ms mỗi bước | ❌ Đường xử lý lệnh qua 3 dịch vụ có thể không đáp ứng yêu cầu < 100ms |

**Khi nào Microservices sụp đổ:** Nhóm < 5 kỹ sư; sản phẩm giai đoạn sớm (ranh giới service sai gây tái cấu trúc tốn kém); đường xử lý đòi hỏi < 10ms.

**Phù hợp**: Nền tảng IoT lớn (10K+ thiết bị), tổ chức đa nhóm.

### 5.3 Kiến trúc hướng sự kiện (Event-Driven Architecture — EDA)

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

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Khớp nối** | Producer và consumer hoàn toàn tách biệt | Hợp đồng ngầm định qua event schema — thay đổi schema làm vỡ consumer ngầm | ✅ Thêm ML consumer mới mà không sửa firmware cảm biến |
| **Khả năng mở rộng** | Kafka xử lý hàng triệu sự kiện/giây | Broker là điểm lỗi duy nhất nếu không cluster | ✅ Kafka hấp thụ tự nhiên đỉnh telemetry IoT |
| **Tính nhất quán** | Event log cho phép replay và phục hồi | Tính nhất quán cuối cùng — consumer có thể lag 50–500ms | ❌ Dashboard có thể hiển thị trạng thái cũ; ✅ chấp nhận được cho giám sát, không phải cho điều khiển |
| **Debug** | Event log cung cấp forensics đầy đủ | Trace lỗi qua chuỗi async phức tạp — cần distributed tracing | ❌ Cảnh báo bị bỏ sót có thể do producer, broker hoặc consumer |

**Khi nào EDA sụp đổ:** Đường lệnh điều khiển cần phản hồi tức thì; dự án nhỏ (broker là overhead thừa); cần read-your-own-writes consistency.

### 5.4 Kiến trúc Serverless

**Mô tả**: Logic ứng dụng chạy trong các hàm phi trạng thái, kích hoạt bởi sự kiện, được quản lý bởi nhà cung cấp đám mây.

```
[Thiết bị IoT] → [IoT Hub] → trigger → [Lambda/Function] → [DynamoDB]
                                │
                         if bất thường
                                ▼
                          [Lambda] → [SNS/Email thông báo]
```

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Vận hành** | Không quản lý server | Vendor lock-in; di chuyển khỏi Lambda đòi viết lại | ✅ Startup 2 người chạy cảnh báo cho 10K thiết bị không cần DevOps |
| **Chi phí** | Trả theo lần gọi — gần bằng 0 khi không có traffic | Ở khối lượng liên tục cao, chi phí theo lần gọi cao hơn container | ✅ Xử lý cảnh báo thưa thớt: chi phí gần 0 khi không có cảnh báo. ❌ Ingestion liên tục 1M sự kiện/ngày: đắt hơn 5–10× so với container |
| **Độ trễ** | Hàm warm: < 10ms | Cold start: 100ms–2s khi hàm không được gọi gần đây | ❌ Không chấp nhận được cho yêu cầu < 100ms. ✅ Chấp nhận được cho thông báo cảnh báo |
| **Trạng thái** | Thiết kế stateless rõ ràng hơn | Mỗi lần gọi phải lấy lại context từ DB — thêm độ trễ | ❌ Theo dõi trạng thái thiết bị cần external state store (DynamoDB, Redis) |

**Khi nào Serverless sụp đổ:** Telemetry liên tục tần suất cao (chi phí cao hơn container 3–10×); đường xử lý yêu cầu < 200ms; quy trình stateful phức tạp.

**Phù hợp**: Pipeline cảnh báo, biến đổi dữ liệu, workload không dự đoán được.

### 5.5 Kiến trúc lai Biên-Đám mây (Edge-Cloud Hybrid)

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

**Phân tích đánh đổi:**

| Chiều đánh đổi | Ưu điểm | Nhược điểm | Bối cảnh IoT |
|---|---|---|---|
| **Độ trễ** | Xử lý cục bộ — < 10ms tại edge gateway | Thêm một tầng kiến trúc phải quản lý | ✅ Ngắt khẩn cấp an toàn nhà máy không thể chờ round-trip đám mây (50–200ms) |
| **Độ tin cậy** | Tiếp tục hoạt động khi mất kết nối đám mây | Đồng bộ hóa sau khi kết nối lại phức tạp (xung đột, thứ tự, trùng lặp) | ✅ Nút lưới điện thông minh quản lý tải trong khi WAN bị ngắt |
| **Băng thông** | Tổng hợp và lọc tại biên — gửi 1–5% dữ liệu thô lên đám mây | Phần cứng edge (NVIDIA Jetson, PC công nghiệp) tốn $500–$5,000+/site | ✅ Nhà máy 500 camera: xử lý video tại biên tiết kiệm 95% băng thông đám mây |
| **Bảo mật** | Phân đoạn mạng giới hạn bán kính vụ nổ khi bị tấn công | Bảo mật vật lý phần cứng edge là trách nhiệm của nhà vận hành | ❌ Kẻ tấn công có quyền truy cập vật lý vào edge gateway có thể trích xuất thông tin xác thực đám mây |
| **Độ phức tạp** | Công cụ phù hợp cho yêu cầu IoT phức tạp | Mẫu phức tạp nhất — cần chuyên môn hệ thống phân tán, edge runtime, sync protocol | ❌ Nhóm mới với IoT không nên bắt đầu từ đây — đường cong học tập trì hoãn ra mắt hàng tháng |

**Khi nào Edge-Cloud Hybrid sụp đổ:** Nhóm thiếu kinh nghiệm hệ thống phân tán; dự án hạn chế CapEx; dữ liệu không quan trọng mà đám mây có thể xử lý ở chi phí hợp lý.

---

### 5.7 So sánh đánh đổi toàn diện

#### Ma trận đánh đổi so sánh 5 mẫu

Đánh giá tương đối (Thấp / Trung bình / Cao / Rất cao) cho môi trường IoT sản xuất:

| Chiều | Layered | Microservices | EDA | Serverless | Edge-Cloud Hybrid |
|---|---|---|---|---|---|
| **Độ phức tạp ban đầu** | Thấp | Cao | Trung bình | Thấp | Rất cao |
| **Độ phức tạp vận hành** | Thấp | Rất cao | Cao | Thấp | Rất cao |
| **Khả năng mở rộng** | Thấp | Cao | Rất cao | Rất cao | Cao |
| **Độ trễ** | Trung bình | Trung bình | Thấp | Thay đổi* | Rất thấp |
| **Khả năng offline** | Không | Không | Không | Không | Đầy đủ |
| **Hiệu quả băng thông** | Không | Không | Không | Không | Rất cao |
| **Quy mô nhóm cần thiết** | 1–3 người | 5+ người | 3–5 người | 1–3 người | 5+ người |
| **Chi phí ban đầu** | Thấp | Trung bình | Trung bình | Rất thấp | Cao (HW biên) |
| **Chi phí liên tục (quy mô lớn)** | Cao (vertical) | Trung bình | Trung bình | Cao** | Trung bình |
| **Debug/quan sát** | Dễ | Khó | Rất khó | Khó | Rất khó |

\* Serverless: thấp với warm function (< 10ms), cao với cold start (100ms–2s)
\** Chi phí Serverless vượt container trên ~1M lần gọi/ngày ở tải liên tục

#### Con đường tiến hóa mẫu kiến trúc

Hệ thống IoT thường tiến hóa qua các mẫu khi tăng trưởng. **Bắt đầu từ trái và di chuyển sang phải khi cơn đau của mẫu hiện tại vượt chi phí di chuyển:**

```
Giai đoạn 1: POC/MVP   Giai đoạn 2: Tăng trưởng   Giai đoạn 3: Quy mô   Giai đoạn 4: Doanh nghiệp
┌────────────────┐      ┌────────────────┐           ┌──────────────────┐   ┌─────────────────────┐
│   Phân lớp     │ →    │ Microservices  │ →         │  Microservices   │ → │  Edge-Cloud Hybrid  │
│   Monolith     │      │ + Serverless   │           │  + EDA           │   │  + Microservices    │
│                │      │                │           │                  │   │  + EDA              │
│ < 1K thiết bị  │      │ 1K–100K thiết bị│          │ 100K+ thiết bị   │   │ Doanh nghiệp lớn   │
└────────────────┘      └────────────────┘           └──────────────────┘   └─────────────────────┘
```

**Dấu hiệu cần di chuyển:**
- Layered → Microservices: xung đột deploy, cần mở rộng độc lập, > 5 kỹ sư
- Microservices → +EDA: gọi REST giữa dịch vụ gây cascade độ trễ, thông lượng > 100K sự kiện/giây
- +EDA → +Edge: yêu cầu độ trễ < 100ms, cần hoạt động offline, chi phí băng thông đáng kể

### 5.6 Hướng dẫn chọn mẫu kiến trúc

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

## 6. Giao thức truyền thông trong IoT

### 6.1 MQTT — Tiêu chuẩn IoT

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

### 6.2 Bảng so sánh giao thức

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

### 6.3 Cây quyết định chọn giao thức

```
Thiết bị cực kỳ hạn chế (< 10KB RAM)?        → CoAP
Cần pub/sub + gửi tin cậy?                    → MQTT
Cần routing phức tạp + tích hợp doanh nghiệp? → AMQP
Dashboard web / UI thời gian thực?             → WebSocket
API quản lý thiết bị?                          → HTTP/REST
Mặc định?                                     → MQTT
```

### 6.4 Kiến trúc đa giao thức

Trong thực tế, hệ thống IoT thường sử dụng **nhiều giao thức** tại các điểm khác nhau:

```
Cảm biến(BLE) → Gateway(CoAP→MQTT) → MQTT Broker → Cloud
                                          ├── AMQP → ERP
                                          ├── HTTP API → Dashboard
                                          └── WebSocket → UI thời gian thực
```

---

## 7. Bảo mật trong kiến trúc IoT

### 7.1 Thách thức bảo mật IoT

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

### 7.2 Bảo mật theo từng lớp

| Lớp | Mối đe dọa | Biện pháp giảm thiểu |
|---|---|---|
| **Cảm nhận** | Can thiệp vật lý, sao chép thiết bị | Secure boot, HSM, TPM |
| **Mạng** | Nghe lén, MITM, replay | TLS/DTLS, certificate pinning, mTLS |
| **Biên** | Truy cập trái phép, malware | Firewall, container isolation, signed firmware |
| **Ứng dụng** | Rò rỉ dữ liệu, injection, DDoS | OAuth 2.0, rate limiting, WAF, mã hóa |

### 7.3 Kiến trúc Zero Trust cho IoT

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

### 7.4 Quyền riêng tư dữ liệu

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

## 8. Điện toán biên (Edge Computing)

### 8.1 Tại sao Edge Computing là thiết yếu

Edge computing đã chuyển từ khái niệm bổ sung thành **thành phần cốt lõi** của kiến trúc IoT hiện đại.

| Lợi ích | Mô tả |
|---|---|
| **Giảm độ trễ** | Xử lý tại chỗ thay vì round-trip đến cloud (ms thay vì 100ms+) |
| **Tiết kiệm băng thông** | Xử lý 80% dữ liệu cục bộ, gửi 20% lên cloud |
| **Hoạt động offline** | Tiếp tục hoạt động khi mất kết nối internet |
| **Tuân thủ dữ liệu** | Giữ dữ liệu trong phạm vi địa lý (GDPR, luật pháp VN) |
| **Khả năng mở rộng** | Phân tán tải xử lý thay vì tập trung tại cloud |

### 8.2 Công nghệ Edge Computing

| Công nghệ | Loại | Mô tả |
|---|---|---|
| **AWS Greengrass** | Nền tảng | Chạy Lambda functions tại edge |
| **Azure IoT Edge** | Nền tảng | Container-based edge modules |
| **K3s** | Orchestration | Kubernetes nhẹ cho edge |
| **KubeEdge** | Orchestration | Kubernetes mở rộng cho edge |
| **NVIDIA Jetson** | Phần cứng | GPU-enabled edge AI |
| **Node-RED** | Low-code | Flow-based programming cho IoT |

### 8.3 AI tại biên (Edge AI)

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

## 9. Xu hướng công nghệ mới

### 9.1 AIoT (AI + IoT)

AI tích hợp trực tiếp vào hệ thống IoT, biến thiết bị từ **nguồn dữ liệu thụ động** thành **tác nhân thông minh tự chủ**.

- **Suy luận trên thiết bị**: Mô hình ML chạy trên MCU nhúng
- **Học liên kết (Federated Learning)**: Huấn luyện mô hình phân tán không cần tập trung dữ liệu
- **Ra quyết định tự chủ**: Thiết bị hoạt động không cần kết nối cloud

### 9.2 Digital Twins (Bản sao số)

- Bản sao ảo thời gian thực của tài sản vật lý
- Đồng bộ liên tục qua luồng dữ liệu IoT
- Mô phỏng kịch bản "what-if" trước khi thay đổi thực tế
- **Công cụ**: Azure Digital Twins, AWS IoT TwinMaker, NVIDIA Omniverse

### 9.3 5G + IoT

| Tính năng 5G | Ứng dụng IoT |
|---|---|
| **URLLC** (< 1ms latency) | Phẫu thuật từ xa, xe tự lái |
| **mMTC** (1M thiết bị/km²) | Triển khai cảm biến quy mô lớn |
| **Network Slicing** | Mạng ảo chuyên dụng cho mỗi ứng dụng IoT |

### 9.4 IoT bền vững (Green IoT)

- Thu hoạch năng lượng (solar, thermal, kinetic) cho cảm biến không pin
- Tối ưu duty cycling và chế độ ngủ
- Carbon-aware computing: dịch chuyển workload đến vùng năng lượng sạch
- Thiết kế thiết bị có tuổi thọ 10+ năm

### 9.5 Matter Protocol

- Tiêu chuẩn thống nhất cho nhà thông minh
- Hỗ trợ bởi Apple, Google, Amazon, Samsung
- Xây dựng trên Thread (mesh) và WiFi
- Một thiết bị hoạt động với tất cả hệ sinh thái

### 9.6 IoT vệ tinh

- Kết nối trực tiếp thiết bị-vệ tinh cho tài sản ở xa
- Ứng dụng: hàng hải, nông nghiệp, dầu khí, lâm nghiệp
- Nhà cung cấp: Swarm (SpaceX), Astrocast, Myriota

---

## 10. Nghiên cứu tình huống thực tế

> Các tình huống dưới đây là các dự án **thực tế từ các công ty lớn**, có tài liệu công khai từ AWS re:Invent, Microsoft Customer Stories, Databricks Blog và học thuật Harvard.

### 10.1 Rolls-Royce — IntelligentEngine (Bảo trì dự đoán động cơ máy bay)

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Rolls-Royce Holdings plc |
| **Lĩnh vực** | Hàng không dân dụng / IIoT |
| **Quy mô** | 4,500+ động cơ máy bay được giám sát liên tục |
| **Mô hình kinh doanh** | TotalCare "Power by the Hour" — hãng bay trả theo giờ bay |
| **Kiến trúc** | Edge-Cloud Hybrid + Digital Twin |
| **Cloud** | Microsoft Azure (IoT Backend + Databricks) |
| **Giao thức** | MQTT qua liên kết vệ tinh/mặt đất |
| **Đặc tính chủ đạo** | Độ tin cậy, Hiệu năng, Khả năng quan sát |

**Quyết định kiến trúc quan trọng:**
- **Xử lý tại biên (ECU)**: Tiền xử lý tại chỗ giảm khối lượng truyền và phát hiện bất thường tức thì
- **Lớp chuẩn hóa dữ liệu**: Hàng trăm biến thể động cơ → 1 định dạng thống nhất cho analytics
- **Azure Databricks + ML**: Mô hình tự học dự đoán hao mòn linh kiện và tuổi thọ còn lại của từng động cơ
- **Digital Twin**: Bản sao số đồng bộ thời gian thực — mô phỏng "what-if" trước khi can thiệp bảo trì

**Kết quả** (nguồn: Microsoft Customer Story):
- Tiết kiệm **hàng triệu USD** nhờ tránh được bảo trì ngoài kế hoạch
- Giám sát thời gian thực **4,500+ động cơ** trên toàn cầu đội bay thương mại
- Mô hình TotalCare khả thi: chỉ có thể cam kết SLA khả dụng khi IoT dự đoán được chính xác

**Bài học kiến trúc:** *Mô hình kinh doanh và kiến trúc gắn liền nhau* — dịch vụ "Power by the Hour" không thể tồn tại nếu không có khả năng giám sát dự đoán liên tục. Kiến trúc IoT không phải là tính năng, mà là sản phẩm.

---

### 10.2 John Deere — Nền tảng Nông nghiệp Chính xác IoT

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Deere & Company (John Deere) |
| **Lĩnh vực** | Nông nghiệp / Thiết bị kết nối |
| **Quy mô** | 1.5 triệu máy kết nối, 500 triệu mẫu Anh được quản lý (mục tiêu 2026) |
| **Tần suất** | Telemetry mỗi **5 giây** từ mỗi máy |
| **Tăng trưởng dữ liệu** | Khối lượng dữ liệu tăng gấp đôi/gấp ba mỗi năm |
| **Kiến trúc** | Multi-Cloud Hybrid + Edge + Microservices |
| **Cloud** | AWS (chính) + Azure Arc + Data Center riêng |
| **Công nghệ AWS** | DynamoDB, Kinesis, OpenSearch, S3 |
| **Đặc tính chủ đạo** | Khả năng mở rộng, Tính tương tác, Hiệu năng |

**Quyết định kiến trúc quan trọng:**
- **Multi-cloud có chủ đích**: AWS cho IoT/analytics; Azure cho vận hành nhà máy sản xuất — các workload khác nhau cần nền tảng khác nhau
- **Kinesis cho streaming**: Xử lý dữ liệu thời gian thực ở quy mô tăng trưởng hàm mũ
- **AI cấp độ từng cây trồng**: Mô hình Databricks xác định lượng hạt giống/phân bón tối ưu cho từng cây trong hàng — hàng nghìn cây/giây/máy
- **Telemetry 5 giây**: Cân bằng giữa dữ liệu thời gian thực có hành động và băng thông mạng di động

**Kết quả** (nguồn: Databricks Blog, AWS Case Study):
- Kết nối **1.5 triệu máy**, quản lý **500 triệu mẫu Anh** với dữ liệu thời gian thực
- **Nông nghiệp chính xác cấp cây trồng**: AI xác định xử lý tối ưu cho từng cây — chuyển đổi từ quy mô cánh đồng sang quy mô cây trồng
- Giảm lãng phí phân bón, thuốc trừ sâu, hạt giống thông qua công nghệ tỷ lệ biến thiên
- Kiến trúc được thiết kế để xử lý **dữ liệu tăng gấp đôi mỗi năm** mà không cần tái kiến trúc

**Bài học kiến trúc:** *Thiết kế cho tăng trưởng 10× ngay từ đầu* — dữ liệu tăng hàm mũ, không tuyến tính. Multi-cloud không phải để tránh lock-in mà để chọn nền tảng phù hợp nhất cho từng loại workload.

---

### 10.3 Siemens MindSphere / Insights Hub — Nền tảng IIoT PaaS

| Yếu tố | Chi tiết |
|---|---|
| **Công ty** | Siemens AG |
| **Lĩnh vực** | Nền tảng IoT công nghiệp (PaaS) |
| **Đặc điểm** | Multi-tenant — phục vụ hàng nghìn khách hàng công nghiệp toàn cầu |
| **Kiến trúc** | Microservices + EDA + Hybrid Cloud |
| **Cloud** | AWS (hosted) |
| **Streaming** | Amazon Kinesis (managed) HOẶC Apache Kafka (self-managed) |
| **App platform** | Cloud Foundry |
| **Tài liệu công khai** | AWS re:Invent 2019, session MFG202 |
| **Đặc tính chủ đạo** | Khả năng mở rộng, Tính tương tác (200+ giao thức công nghiệp), Bảo trì |

**Quyết định kiến trúc quan trọng:**
- **"Đi theo sáng tạo của AWS"**: Siemens chọn dịch vụ managed của AWS thay vì tự quản lý — giảm chi phí vận hành và tự động được cải tiến hạ tầng
- **Cung cấp cả Kinesis lẫn Kafka**: Khách hàng có yêu cầu quản trị khác nhau — managed service (Kinesis) cho đơn giản, self-hosted (Kafka) cho kiểm soát
- **Microservices lỏng lẻo**: Mỗi chức năng (quản lý tài sản, identity, analytics, time-series) scale và cập nhật độc lập
- **MindConnect Elements**: Adapter phần cứng agnostic kết nối bất kỳ thiết bị công nghiệp nào (OPC UA, Modbus, S7, REST, MQTT) với nền tảng

**Kết quả — Triển khai Rittal Blue e+** (nguồn: AWS Case Study, Siemens Whitepaper):
- Giảm **75%** tiêu thụ năng lượng của hệ thống làm mát kết nối mạng
- Giảm **75%** lượng carbon thải ra
- ROI dưới **3 tháng** được ghi nhận trong các kịch bản bảo trì dự đoán
- **15+ năm** phát triển nền tảng — hệ sinh thái **1,700+ đối tác** và ứng dụng

**Bài học kiến trúc:** *Tính tương tác giao thức mới là hào kinh tế thực sự* — khả năng kết nối bất kỳ thiết bị công nghiệp nào (200+ giao thức) khó sao chép hơn nhiều so với stack analytics trên đám mây. Cung cấp lựa chọn managed vs self-managed streaming cho phép đáp ứng các yêu cầu quản trị khác nhau mà không phân nhánh kiến trúc.

---

## 11. Kết luận và khuyến nghị

### 11.1 Tóm tắt các điểm chính

1. **Kiến trúc phân lớp** cung cấp phân tách trách nhiệm rõ ràng — hiểu vai trò của từng lớp
2. **Chọn mẫu kiến trúc dựa trên yêu cầu**, không phải xu hướng — sử dụng framework chọn lựa
3. **MQTT là tiêu chuẩn thực tế** cho IoT messaging — bắt đầu từ đây trừ khi có ràng buộc đặc biệt
4. **Edge computing là thiết yếu**, không phải tùy chọn — xử lý cục bộ, đồng bộ toàn cục
5. **Bảo mật phải được tích hợp từ đầu** — Zero Trust từ thiết bị đến đám mây
6. **AIoT và Digital Twins** đang định hình lại khả năng của IoT
7. **Không có mẫu nào phù hợp tất cả** — kết hợp các mẫu cho nhu cầu cụ thể

### 11.2 Khuyến nghị cho tổ chức

| Quy mô | Khuyến nghị kiến trúc |
|---|---|
| **Startup / POC** | Phân lớp + MQTT + Serverless |
| **SME (100–1K thiết bị)** | Microservices + MQTT + Edge gateway |
| **Doanh nghiệp (10K+)** | Edge-Cloud Hybrid + Microservices + EDA |
| **IoT công nghiệp** | Edge-First + AI at Edge + MQTT + Digital Twins |

### 11.3 Lộ trình triển khai đề xuất

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

## 12. Tài liệu tham khảo

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
