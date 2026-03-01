# 🏗️ IoT Reference Architecture

> A detailed exploration of the layered architecture that underpins modern IoT systems

---

## Overview

An IoT reference architecture provides a standardized framework for designing connected systems. It defines the major building blocks, their responsibilities, and how they interact. While specific implementations vary, most IoT architectures follow a layered model that separates concerns and enables independent scaling.

---

## The 4-Layer Model

```
┌─────────────────────────────────────────────────────┐
│                 APPLICATION LAYER                    │
│  Dashboards │ Analytics │ Business Logic │ APIs      │
├─────────────────────────────────────────────────────┤
│              EDGE / PROCESSING LAYER                 │
│  Gateways │ Edge Compute │ Local Storage │ Rules     │
├─────────────────────────────────────────────────────┤
│                  NETWORK LAYER                       │
│  WiFi │ 5G │ LoRaWAN │ BLE │ MQTT │ CoAP │ HTTP     │
├─────────────────────────────────────────────────────┤
│                PERCEPTION LAYER                      │
│  Sensors │ Actuators │ Embedded Firmware │ MCUs      │
└─────────────────────────────────────────────────────┘
```

---

## Layer 1: Perception Layer

### Purpose
The perception layer is the physical interface between the digital and physical worlds. It comprises the hardware devices that sense environmental conditions and act upon the physical world.

### Components

| Component | Role | Examples |
|---|---|---|
| **Sensors** | Measure physical phenomena | Temperature (DS18B20), humidity (DHT22), GPS, accelerometer, camera |
| **Actuators** | Perform physical actions | Motors, relays, solenoid valves, LED displays |
| **Microcontrollers** | Run embedded firmware | ESP32, STM32, Arduino, Raspberry Pi Pico |
| **Embedded OS** | Manage device resources | FreeRTOS, Zephyr RTOS, Mbed OS, Contiki-NG |

### Design Considerations

- **Power management**: Battery-powered devices must be ultra-efficient. Use deep sleep modes, duty cycling, and efficient communication protocols.
- **Constrained resources**: Many IoT devices have limited CPU (MHz), RAM (KB), and storage. Choose lightweight protocols and optimize firmware.
- **Hardware security**: Use Trusted Platform Modules (TPM) or Hardware Security Modules (HSM) for secure key storage and secure boot.
- **Over-the-Air (OTA) updates**: Design firmware to support partial or full OTA updates from day one.
- **Physical environment**: Devices must withstand their deployment environment (temperature, humidity, vibration, IP rating).

### Example: Industrial Temperature Monitoring

```
[DS18B20 Sensor] → [ESP32 MCU] → [WiFi/MQTT] → Gateway
                     │
                     ├── Reads temperature every 30s
                     ├── Applies Kalman filter for noise reduction
                     ├── Publishes to MQTT topic: factory/zone1/temp
                     └── Deep sleeps between readings
```

---

## Layer 2: Network Layer

### Purpose
The network layer handles all communication between devices, gateways, and cloud services. It includes both the physical connectivity and the application-layer protocols used for data transport.

### Connectivity Technologies

| Technology | Range | Bandwidth | Power | Cost | Best For |
|---|---|---|---|---|---|
| **BLE 5.0** | 100m | 2 Mbps | Very Low | Low | Wearables, proximity |
| **Zigbee** | 100m (mesh) | 250 kbps | Low | Low | Smart home, lighting |
| **Thread/Matter** | 100m (mesh) | 250 kbps | Low | Low | Smart home (unified) |
| **WiFi 6** | 100m | 9.6 Gbps | High | Low | High-bandwidth indoor |
| **LoRaWAN** | 15 km | 50 kbps | Very Low | Low | Agriculture, utilities |
| **NB-IoT** | 10 km | 250 kbps | Low | Medium | Asset tracking, metering |
| **5G (mMTC)** | 1 km | 10 Gbps | Medium | High | Autonomous vehicles, AR |
| **Satellite** | Global | Variable | High | High | Maritime, remote assets |

### Application-Layer Protocols

Detailed comparison — see [Protocols document](03-protocols.md).

### Design Considerations

- **Protocol bridging**: Gateways often translate between local protocols (BLE, Zigbee) and internet protocols (MQTT, HTTP).
- **Network reliability**: IoT networks are inherently unreliable. Design for intermittent connectivity with store-and-forward mechanisms.
- **Bandwidth optimization**: Compress payloads, use binary formats (CBOR, Protobuf), and send deltas instead of full state.
- **Quality of Service (QoS)**: Match QoS levels to data criticality — not all telemetry needs guaranteed delivery.

---

## Layer 3: Edge / Processing Layer

### Purpose
The edge layer processes data close to its source, reducing latency, conserving bandwidth, and enabling offline operation. This layer has become increasingly important as IoT systems mature.

### Components

| Component | Role | Examples |
|---|---|---|
| **IoT Gateway** | Protocol translation, device management | AWS Greengrass, Azure IoT Edge |
| **Edge Server** | Local compute and storage | Kubernetes (K3s), Docker, NVIDIA Jetson |
| **Stream Processor** | Real-time data processing | Apache Kafka, Apache Flink, Node-RED |
| **Local Database** | Time-series and event storage | InfluxDB, SQLite, LevelDB |
| **Rules Engine** | Automated decision-making | Node-RED, custom rule engines |

### Edge Processing Patterns

#### 1. Data Filtering & Aggregation
```
Raw sensor data (1000 readings/sec)
    → Edge: Average over 1-minute windows
    → Cloud: 1 aggregated reading/min (99.9% reduction)
```

#### 2. Local Inference (AI at the Edge)
```
Camera stream (30 fps)
    → Edge: Run YOLO object detection model
    → Cloud: Send only detected events (anomalies, alerts)
```

#### 3. Store-and-Forward
```
Sensor data during network outage
    → Edge: Buffer in local SQLite database
    → Network restored: Replay buffered data to cloud (ordered by timestamp)
```

#### 4. Local Automation
```
Temperature sensor reads 95°C (threshold: 90°C)
    → Edge: Immediately trigger cooling actuator
    → Cloud: Log event for analytics (non-time-critical)
```

### Design Considerations

- **Resource allocation**: Edge devices have limited compute — prioritize which workloads run locally vs. cloud.
- **Data sovereignty**: Some industries (healthcare, government) require data to stay within geographic boundaries.
- **Synchronization**: Implement conflict resolution strategies for data that's modified both at the edge and in the cloud.
- **Fleet management**: Use tools like AWS Greengrass or Azure IoT Edge to deploy, monitor, and update edge software at scale.

---

## Layer 4: Application Layer

### Purpose
The application layer is where raw IoT data is transformed into business value. It encompasses cloud platforms, data pipelines, analytics engines, and user-facing applications.

### Typical Architecture

```
Ingestion → Processing → Storage → Analytics → Visualization
   │            │            │          │            │
   MQTT      Stream      Time-series   ML/AI     Dashboards
   HTTP      Processing   Document    Rules       Mobile Apps
   AMQP      Batch        Relational  Reports     APIs
```

### Components

| Component | Role | Technology Options |
|---|---|---|
| **Message Ingestion** | Receive device data at scale | AWS IoT Core, Azure IoT Hub, Apache Kafka |
| **Stream Processing** | Real-time transformations | Apache Flink, Spark Streaming, AWS Kinesis |
| **Batch Processing** | Historical analysis | Apache Spark, Hadoop, Databricks |
| **Time-Series DB** | Store telemetry data | InfluxDB, TimescaleDB, Amazon Timestream |
| **Document DB** | Store device metadata | MongoDB, DynamoDB, Cosmos DB |
| **ML Platform** | Predictive analytics | TensorFlow, PyTorch, AWS SageMaker |
| **Visualization** | Dashboards and reports | Grafana, Power BI, Kibana, custom web apps |
| **API Gateway** | External integrations | Kong, AWS API Gateway, Express.js |

### Design Considerations

- **Multi-tenancy**: Design for multiple customers/organizations from the start if building a platform.
- **Data retention**: Implement tiered storage — hot (SSD), warm (HDD), cold (S3/Glacier) — based on data age.
- **Horizontal scaling**: Use auto-scaling groups and container orchestration to handle variable loads.
- **API versioning**: Maintain backward compatibility as the platform evolves.

---

## Cross-Cutting Concerns

These concerns span all four layers and are the implementation mechanisms for several architecture characteristics. The concerns below — Device Management, Monitoring & Observability, and Data Governance — directly implement the **Maintainability**, **Observability**, and **Security** characteristics respectively. For the theoretical framework that explains *why* these concerns require explicit architectural decisions, see [06-architecture-characteristics.md](06-architecture-characteristics.md).

### 1. Device Management
- Device provisioning and registration
- Configuration management
- Firmware updates (OTA)
- Device lifecycle (active, inactive, decommissioned)
- Device twin / shadow (cloud representation of device state)

### 2. Monitoring & Observability
- Device health metrics (battery, connectivity, uptime)
- Platform metrics (throughput, latency, error rates)
- Distributed tracing across layers
- Alerting and incident response
- Tools: Prometheus, Grafana, ELK Stack, Datadog

### 3. Data Governance
- Data lineage tracking
- Schema management and versioning
- Data quality validation at ingestion
- Compliance with regulations (GDPR, HIPAA)
- Audit logging

---

## Reference Architecture: Smart Building Example

```
                           ┌──────────────────────┐
                           │   APPLICATION LAYER   │
                           │                       │
                           │  ┌─────────────────┐  │
                           │  │ Building Mgmt   │  │
                           │  │ Dashboard        │  │
                           │  └────────┬────────┘  │
                           │           │           │
                           │  ┌────────┴────────┐  │
                           │  │ Analytics Engine │  │
                           │  │ (Energy, HVAC,   │  │
                           │  │  Occupancy)      │  │
                           │  └────────┬────────┘  │
                           │           │           │
                           │  ┌────────┴────────┐  │
                           │  │ Cloud Platform   │  │
                           │  │ (Azure IoT Hub)  │  │
                           │  └────────┬────────┘  │
                           └───────────┼───────────┘
                                       │
                           ┌───────────┼───────────┐
                           │     EDGE LAYER        │
                           │                       │
                           │  ┌────────┴────────┐  │
                           │  │ Building Gateway │  │
                           │  │ (Azure IoT Edge) │  │
                           │  │                  │  │
                           │  │ • HVAC rules     │  │
                           │  │ • Occupancy ML   │  │
                           │  │ • Data aggregation│ │
                           │  └────────┬────────┘  │
                           └───────────┼───────────┘
                                       │
              ┌────────────────────────┼────────────────────────┐
              │                        │                        │
    ┌─────────┴──────────┐  ┌─────────┴──────────┐  ┌─────────┴──────────┐
    │  HVAC Zone Ctrl    │  │  Lighting System   │  │  Access Control    │
    │  (BACnet/MQTT)     │  │  (DALI/Zigbee)     │  │  (RFID/BLE)        │
    │                    │  │                    │  │                    │
    │ • Temp sensors     │  │ • Occupancy sensors│  │ • Card readers     │
    │ • Humidity sensors │  │ • Light sensors    │  │ • Door actuators   │
    │ • Valve actuators  │  │ • LED controllers  │  │ • Cameras          │
    └────────────────────┘  └────────────────────┘  └────────────────────┘
            PERCEPTION LAYER
```

---

## Next Steps

- [Architecture Patterns](02-patterns.md) — Explore patterns like microservices, event-driven, and serverless
- [Communication Protocols](03-protocols.md) — Deep-dive into MQTT, CoAP, and protocol selection
- [Security](04-security.md) — Security considerations across all layers
