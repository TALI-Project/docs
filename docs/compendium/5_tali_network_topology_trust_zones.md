# TALI Design Hub — Section 24: Network Topology & Trust Zones

## 5. Network Topology & Trust Zones
> Defines the recommended network segmentation, trust models, and communication pathways that ensure secure, low-latency operation for TALI and its supporting services.

---

### 5.1 Overview

TALI’s distributed architecture depends on a robust and predictable network topology. While the documentation outlines best practices using VLANs and trust zones, TALI is designed to **operate on standard home networks** where advanced segmentation may not be present. In such configurations, TALI uses logical separation, encrypted communication, and topic-level access control to maintain security even within a flat network.

The system minimizes latency for real-time interactions while maintaining strict logical boundaries between trusted, semi-trusted, and untrusted entities. Each communication link still enforces authentication, encryption, and device attribution regardless of physical topology.

Home Assistant typically acts as the **network orchestrator** — hosting the MQTT broker, managing device I/O, and bridging sensors, ESP microphones, and Zigbee/Matter networks. TALI Core connects as a **privileged client** with read/write access to specific topics and APIs, while all other components communicate through controlled gateways.

---

### 5.2 Network Zones

| Zone | Trust Level | Description | Typical Components |
|------|--------------|-------------|--------------------|
| **Zone 0 — Secure Core** | Full trust | Contains critical logic, databases, and services. Restricted to system admin access only. | TALI Core, PostgreSQL, Redis, Neo4j, MinIO, Prometheus, Grafana. |
| **Zone 1 — Integration Layer** | High trust | Handles communication between TALI and Home Assistant or brokers. Enforces TLS and authentication. | Home Assistant, MQTT Broker (EMQX/Mosquitto), API Gateway. |
| **Zone 2 — Edge Devices** | Controlled trust | Physical sensors and microphones that stream telemetry or audio. Restricted topic scopes and device certificates. | ESP32 microphones, Zigbee2MQTT, Matter bridges, mmWave sensors. |
| **Zone 3 — Resident Network** | User trust | Resident devices and interfaces. Indirect access through HA or Messenger APIs only. | Phones, tablets, smart speakers, Discord/Voice clients. |
| **Zone 4 — External Services** | Minimal trust | Cloud services or APIs used under explicit consent. | Optional LLM endpoints, OTA update servers, weather APIs. |

---

### 5.3 Communication Model

```
[Zone 2: Edge Devices]  →  [Zone 1: MQTT Broker / HA]  →  [Zone 0: TALI Core]  →  [Zone 3: Residents / Zone 4: Cloud]
```

All communications follow the **brokered model**:
- Edge nodes communicate *only* with HA/MQTT brokers (never directly with TALI Core).  
- TALI subscribes to HA/MQTT topics to read sensor and device events.  
- TALI emits control commands through the Actuator Gate, which re-publishes via HA/MQTT within its permitted scope.  
- Messenger or external integrations (Discord, OTA) reside in Zone 3 or 4 and communicate via secure webhooks or REST APIs.

---

### 5.4 Authentication & Encryption

- **MQTT:** Mutual TLS (mTLS) using per-device certificates. Each ESP and bridge must have a unique identity.  
- **Home Assistant API:** Long-lived access tokens with scope restrictions.  
- **TALI Internal Links:** Encrypted with gRPC or HTTPS and verified via self-signed PKI managed by the Core.  
- **External Services:** Always over HTTPS; outbound only; explicit resident approval required.

---

### 5.5 Broker Configuration

| Setting | Recommended Value | Notes |
|----------|------------------|-------|
| **Port** | 8883 (TLS) | Standard secure MQTT port. |
| **Protocol** | MQTT v5 | Ensures topic-level ACLs and properties. |
| **Authentication** | x.509 certificates | Issued by local CA managed by TALI or HA. |
| **ACLs** | Strict | Edge devices: publish-only to their topic; TALI: subscribe + restricted control topics. |
| **Bridging** | Optional | Allows multi-broker sync for large deployments. |

---

### 5.6 VLAN & Subnet Layout (Recommended)

| Zone | VLAN | Example Subnet | Purpose |
|------|------|----------------|----------|
| Zone 0 — Core | VLAN 10 | 192.168.10.0/24 | Databases and analytics — isolated from IoT noise. |
| Zone 1 — Integration | VLAN 20 | 192.168.20.0/24 | MQTT, HA, Zigbee2MQTT bridges. |
| Zone 2 — Edge Devices | VLAN 30 | 192.168.30.0/24 | ESP microphones, sensors, mmWave, TRVs. |
| Zone 3 — Residents | VLAN 40 | 192.168.40.0/24 | Phones, tablets, UI clients. |
| Zone 4 — External | VLAN 50 | 192.168.50.0/24 | Cloud access through outbound NAT only. |

Each VLAN should be isolated with inter-VLAN routing controlled via firewall policies — only the Core can initiate cross-zone communication.

---

### 5.7 Firewall & Trust Policies

- **Default-Deny:** All inbound traffic to TALI Core denied except via authenticated gateways.  
- **Broker Isolation:** Only MQTT 8883 open between Zones 0↔1; no other inbound ports.  
- **Edge Limitation:** Zone 2 devices can publish but cannot subscribe beyond their topic path.  
- **Outbound Whitelist:** Only specific external endpoints (update, optional LLMs) allowed.  
- **mDNS Control:** Disable multicast DNS between zones to prevent device leakage.

---

### 5.8 Latency & Reliability Targets

| Path | Target RTT | Notes |
|------|-------------|-------|
| Edge → MQTT Broker | ≤ 5 ms | Ensures real-time voice/telemetry. |
| Broker → TALI Core | ≤ 10 ms | Critical for low-latency decision loops. |
| Core → Resident UI | ≤ 50 ms | Comfortable conversational response. |
| Core → External Cloud | ≤ 150 ms | Non-critical async operations. |

---

### 5.9 Trust Hierarchy

| Tier | Authority | Example Policies |
|------|------------|-----------------|
| **Root CA** | TALI Core or HA Supervisor | Issues device and service certificates. |
| **Broker ACL Authority** | MQTT Broker | Grants topic access rights. |
| **Automation Authority** | TALI Policy Engine | Determines whether an automation is permissible. |
| **Resident Oversight** | Human Admin | Final approval and ethical control. |

---

### 5.10 Monitoring & Intrusion Detection

- Network telemetry aggregated in Prometheus + Grafana.  
- Suspicious MQTT patterns (e.g., replay loops, burst floods) trigger internal security events.  
- Zone-level packet statistics analyzed to detect misrouted devices.  
- Optional integration with HA’s Supervisor security dashboard for unified alerting.

---

### 5.11 Example Topology Diagram

```
         [ Zone 4 - Cloud / External ]
                   ↑ HTTPS (Outbound)
                   │
        ┌─────────────────────────────┐
        │         Firewall/NAT        │
        └─────────────┬───────────────┘
                      │
           [ Zone 3 - Resident VLAN ]
               │        ↑ REST / WebSocket
               ▼
     [ Zone 1 - Integration VLAN ] ←→ [ Zone 2 - Edge VLAN ]
          (MQTT + HA)                 (ESP Mics / Sensors)
               │
               ▼
          [ Zone 0 - TALI Core ]
   (Databases, AI, Policy, Learning)
```

---

### 5.12 Future Enhancements

- Auto-provisioning of TLS certificates for new edge nodes.  
- Zero-trust mutual attestation for broker-client pairs.  
- Latency-aware routing for multi-broker or multi-home setups.  
- Dynamic QoS tuning based on policy priority (safety > analytics).  

---

*(Section 5 defines the secure network topology and trust model for TALI — ensuring isolation between operational zones, authenticated communication, and reliable low-latency data flow across all layers of the system.)*

