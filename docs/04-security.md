# 🔒 Security Considerations for IoT

> IoT expands the attack surface dramatically. Security must be an integral part of the architecture from day one.

---

## The IoT Security Challenge

- **Scale**: Millions of devices = millions of potential entry points
- **Diversity**: Heterogeneous hardware, OS, and firmware versions
- **Constraints**: Many devices can't run traditional security software
- **Longevity**: Devices deployed for 10+ years must receive updates
- **Physical exposure**: Devices in public or uncontrolled environments

### Notable IoT Security Incidents
| Incident | Impact |
|---|---|
| **Mirai Botnet (2016)** | 600K+ compromised IoT devices used for massive DDoS |
| **Casino Fish Tank Hack** | Attackers pivoted through a smart thermometer to steal data |
| **Stuxnet** | Industrial SCADA attack on nuclear centrifuges |
| **Ring Camera Breaches** | Default credentials exploited for unauthorized surveillance |

---

## Security at Every Layer

### Perception Layer
| Threat | Mitigation |
|---|---|
| Physical tampering | Tamper-evident enclosures, secure boot |
| Device cloning | Unique device identity (X.509 certs, TPM) |
| Firmware extraction | Encrypted firmware, code obfuscation |
| Side-channel attacks | Hardware security modules (HSM) |

### Network Layer
| Threat | Mitigation |
|---|---|
| Eavesdropping | TLS 1.3 / DTLS encryption |
| Man-in-the-Middle | Certificate pinning, mutual TLS (mTLS) |
| Replay attacks | Timestamps + nonces in messages |
| Protocol exploitation | Strict input validation, protocol hardening |

### Edge Layer
| Threat | Mitigation |
|---|---|
| Unauthorized access | Role-based access control (RBAC) |
| Malware injection | Container isolation, signed images |
| Data exfiltration | Network segmentation, firewall rules |
| Compromised gateway | Signed firmware, secure boot chain |

### Application Layer
| Threat | Mitigation |
|---|---|
| Data breach | Encryption at rest (AES-256), access controls |
| API abuse | OAuth 2.0, rate limiting, API keys |
| Injection attacks | Input validation, parameterized queries |
| DDoS | WAF, CDN, auto-scaling, rate limiting |

---

## Zero Trust Architecture for IoT

**Principle: Never trust, always verify — even internal devices.**

### Key Components

1. **Unique Device Identity**
   - Every device gets a unique X.509 certificate or device token
   - Provisioned during manufacturing or first boot
   - Certificates rotated periodically

2. **Mutual TLS (mTLS)**
   - Both device AND server authenticate each other
   - Prevents rogue servers and unauthorized devices

3. **MQTT Topic-Level Access Control**
   ```
   Device sensor-42:
     PUBLISH: factory/zone1/sensor-42/#     ✅
     SUBSCRIBE: cmd/sensor-42/#             ✅
     PUBLISH: factory/zone1/sensor-99/#     ❌ (not its own data)
     SUBSCRIBE: cmd/sensor-99/#             ❌ (not its commands)
   ```

4. **Continuous Monitoring**
   - Anomaly detection on device behavior
   - Unusual data patterns, unexpected connections, traffic spikes

5. **Secure OTA Updates**
   - Code signing with asymmetric keys (RSA/ECDSA)
   - Rollback capability if update fails
   - Staged rollouts (canary → percentage → full fleet)

---

## Data Privacy

### Regulatory Landscape
| Regulation | Region | Key Requirement |
|---|---|---|
| **GDPR** | EU | Data minimization, right to erasure, consent |
| **CCPA/CPRA** | California | Consumer rights, opt-out of data sale |
| **HIPAA** | US Healthcare | PHI protection, audit trails |
| **POPIA** | South Africa | Lawful processing, data subject rights |

### Privacy-by-Design Principles
1. **Data minimization**: Collect only necessary data
2. **Edge processing**: Keep sensitive data local when possible
3. **Anonymization**: Remove PII before cloud upload
4. **Encryption**: End-to-end encryption for sensitive streams
5. **Retention policies**: Auto-delete data after defined periods
6. **Audit logging**: Track who accessed what data, when

---

## Security Architecture Pattern

```
┌─────────── Cloud ────────────┐
│  ┌─────────────────────────┐ │
│  │   Identity & Access     │ │
│  │   Management (IAM)      │ │
│  └─────────┬───────────────┘ │
│            │                  │
│  ┌─────────┴───────────────┐ │
│  │ Certificate Authority   │ │
│  │ (PKI Infrastructure)    │ │
│  └─────────┬───────────────┘ │
│            │                  │
│  ┌─────────┴───────────────┐ │
│  │ Security Monitoring     │ │
│  │ (SIEM + Anomaly Detect) │ │
│  └─────────────────────────┘ │
└──────────────┬───────────────┘
               │ mTLS
┌──────────────┴───────────────┐
│         Edge Gateway         │
│  • Firewall rules            │
│  • Certificate validation    │
│  • Protocol filtering        │
│  • Local encryption          │
└──────────────┬───────────────┘
               │ Encrypted
┌──────────────┴───────────────┐
│         IoT Devices          │
│  • Secure boot (TPM/HSM)     │
│  • Unique X.509 certificate  │
│  • Encrypted storage         │
│  • Signed firmware           │
└──────────────────────────────┘
```

---

## Security Checklist

- [ ] Every device has a unique cryptographic identity
- [ ] All communication encrypted (TLS/DTLS)
- [ ] Mutual authentication (mTLS) enabled
- [ ] Topic/resource-level access control configured
- [ ] OTA update mechanism with code signing
- [ ] Firmware rollback capability
- [ ] Network segmentation (IoT VLAN)
- [ ] Anomaly detection monitoring active
- [ ] Data retention and deletion policies defined
- [ ] Regular security audits and penetration testing
- [ ] Incident response plan documented

---

## Next Steps
- [Case Studies](05-case-studies.md) — See security in real-world IoT deployments
