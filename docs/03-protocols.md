# 📡 Communication Protocols for IoT

> Choosing the right protocol is one of the most critical IoT architectural decisions.

---

## Protocol Landscape

```
┌─────────────────────────────────────────┐
│          Application Protocols          │
│   MQTT │ CoAP │ AMQP │ HTTP │ WebSocket │
├─────────────────────────────────────────┤
│         Transport: TCP │ UDP │ TLS      │
├─────────────────────────────────────────┤
│     Network: IPv4 │ IPv6 │ 6LoWPAN     │
├─────────────────────────────────────────┤
│  Physical: WiFi │ BLE │ Zigbee │ LoRa  │
└─────────────────────────────────────────┘
```

---

## 1. MQTT — The IoT Standard

**Message Queuing Telemetry Transport** — Pub/Sub model over TCP.

### Architecture
```
[Sensor] ──publish("sensor/temp")──▶ [MQTT Broker] ──deliver──▶ [Dashboard]
```

### Key Features
| Feature | Description |
|---|---|
| **Pub/Sub** | Decoupled producers/consumers |
| **QoS 0/1/2** | At most once / At least once / Exactly once |
| **Retained Messages** | Last message stored for new subscribers |
| **Last Will (LWT)** | Auto-published on unexpected disconnect |
| **Topic Hierarchy** | `building/floor1/room3/temperature` |
| **Wildcards** | `+` single level, `#` multi-level |

### Topic Design
```
{org}/{site}/{zone}/{device_type}/{device_id}/{measurement}
# Example: factory/plant-a/line1/robot/R-001/temperature
# Commands: cmd/{device_id}/config
```

### QoS Guide
| QoS | Guarantee | Use Case |
|---|---|---|
| **0** | At most once | Non-critical telemetry |
| **1** | At least once | Important events |
| **2** | Exactly once | Critical commands |

### Brokers: Mosquitto (dev), HiveMQ (enterprise), EMQX (high-perf), AWS IoT Core (managed)

---

## 2. CoAP — Constrained Devices

**Constrained Application Protocol** — RESTful over UDP.

- GET/PUT/POST/DELETE like HTTP, but 4-byte header
- Observe pattern for subscriptions
- DTLS for security
- **Best for**: 6LoWPAN, devices with < 10KB RAM

---

## 3. AMQP — Enterprise Integration

- Rich routing (exchanges, queues, bindings)
- Transaction support, flow control
- **Best for**: IoT ↔ ERP/CRM integration, complex routing

---

## 4. HTTP/REST — APIs & Configuration

- **Use for**: Device config APIs, firmware downloads, dashboards, webhooks
- **Avoid for**: Real-time telemetry, constrained devices, high-frequency data

---

## 5. WebSocket — Real-time Dashboards

- Full-duplex, bidirectional over TCP
- **Best for**: Browser dashboards, MQTT-over-WebSocket

---

## Comparison Matrix

| Criteria | MQTT | CoAP | AMQP | HTTP | WebSocket |
|---|---|---|---|---|---|
| Transport | TCP | UDP | TCP | TCP | TCP |
| Model | Pub/Sub | Req/Res | Pub/Sub+Queue | Req/Res | Bidirectional |
| Header | 2 bytes | 4 bytes | 8+ bytes | 100+ bytes | 2-14 bytes |
| Power | Low | Very Low | Medium | High | Medium |
| Bandwidth | Very Low | Very Low | Medium | High | Low |
| QoS | 3 levels | Confirmable | Transactions | None | None |
| Offline | Yes | No | Yes | No | No |
| Browser | WebSocket | No | No | Native | Native |

## Decision Tree

```
Severely constrained device (< 10KB RAM)? → CoAP
Need pub/sub + reliable delivery?         → MQTT
Complex routing + enterprise?             → AMQP
Web dashboard / real-time UI?             → WebSocket (or MQTT over WS)
Device management API?                    → HTTP/REST
Default?                                  → MQTT
```

## Multi-Protocol Architecture

```
Sensor(BLE) → Gateway(CoAP→MQTT) → MQTT Broker → Cloud
                                        ├── AMQP → ERP
                                        ├── HTTP API → Dashboard
                                        └── WebSocket → Real-time UI
```
