# TALI Design Hub — Section 23: Hardware & Environment Requirements

## 4. Hardware & Environment Requirements
> Specifies the recommended hardware, peripherals, and network environment for running TALI efficiently and safely — across both standalone and containerized deployments.

---

### 4.1 Overview

TALI is designed to operate on consumer-grade hardware while maintaining enterprise-level stability. Its architecture supports two primary modes:
1. **Hybrid/Standalone Mode:** TALI Core runs natively (ideal for M‑series Macs or high‑end x86 systems).  
2. **Containerized Mode:** All components, including TALI Core, run via Docker Compose on Linux or macOS hosts.

In most deployments, **Home Assistant acts as the host for device I/O and the MQTT network**, providing a unified communication layer between sensors, ESP microphone boards, and TALI Core.  
Room microphones are expected to be **ESP32/ESP32‑S3‑based boards** with I²S microphones that handle local wake‑word and VAD processing before streaming audio frames to TALI via the HA‑managed MQTT broker.

The environment must balance CPU performance for LLM inference, I/O throughput for database operations, and consistent low‑latency audio capture/playback.

---

### 4.2 Minimum Hardware Specifications

| Component | Minimum | Recommended | Notes |
|------------|----------|--------------|-------|
| **CPU** | 4 cores (Apple M1 / Intel i5 10th‑gen / AMD Ryzen 5) | 8+ cores (Apple M2 / Ryzen 7 / Intel i7) | Multicore parallelism supports sandbox simulations and LLM inference. |
| **RAM** | 16 GB | 32 GB+ | Memory‑intensive during reinforcement simulations and ASR/TTS pipelines. |
| **Storage** | 100 GB SSD | 500 GB NVMe | Fast I/O essential for DuckDB/Neo4j workloads and logging. |
| **GPU / NPU** | Optional | Apple Neural Engine / RTX 30‑series or higher | Accelerates local LLM and TTS inference. |
| **Network** | 1 Gb Ethernet / Wi‑Fi 6 | Dedicated LAN segment with MQTT broker co‑located | Lower latency for packet mimicry and HA communication. |
| **Audio I/O** | One USB microphone array + one output device per room | Studio‑grade mic arrays with echo cancellation | Required for accurate multi‑room ASR and voice presence detection. |

---

### 4.3 Peripheral Requirements

#### 4.3.1 Microphones
- Use **USB or I²S microphone arrays** with hardware echo cancellation (e.g., ReSpeaker 2‑Mic/4‑Mic, MiniDSP UMA‑16, or macOS array).  
- For distributed rooms, configure as separate logical inputs under `audio.rooms` in configuration YAML.  
- Each mic must have unique system identifiers to prevent mapping confusion.

#### 4.3.2 Speakers
- Use low‑latency output devices capable of < 80 ms round‑trip audio.  
- Smart speakers may be used if accessible via network streaming endpoints.  
- Prefer wired (USB/C‑Audio) or local Bluetooth A2DP over Wi‑Fi streaming to avoid jitter.

#### 4.3.3 Storage
- NVMe or SSD drives strongly recommended for fast access to analytics, vector embeddings, and Neo4j graphs.  
- Avoid external USB drives for live databases.

#### 4.3.4 Sensors and Actuators
- Zigbee2MQTT or Matter bridges connected via serial or network.  
- Presence sensors (Bermuda/mmWave) and thermostatic TRVs supported natively.  
- Optional hardware security module (HSM) for key management.

---

### 4.4 Network Environment

| Requirement | Description |
|--------------|-------------|
| **MQTT Broker** | Must run locally (Mosquitto or EMQX). TALI expects TLS‑secured communication with unique client certificates. |
| **Home Assistant Instance** | Local or LAN‑reachable; WebSocket and API ports must be accessible to TALI Core. |
| **Isolation Zones** | Recommended VLAN segmentation:  
  - *Zone 1:* TALI Core and Databases.  
  - *Zone 2:* MQTT + Zigbee2MQTT.  
  - *Zone 3:* Resident devices (phones, speakers, etc.). |
| **Latency Targets** | ≤ 5 ms local broker RTT, ≤ 20 ms HA API response. |
| **External Access** | Outbound HTTPS only; inbound ports closed unless explicitly allowed for dashboards. |

---

### 4.5 Power & Thermal Considerations

- Continuous 24/7 operation requires adequate cooling (especially for Intel/AMD builds).  
- For Raspberry Pi 5 or similar SBC deployments: use active cooling and high‑endurance SSD storage.  
- Standalone Mac Mini (M1/M2) is preferred for silent, thermally efficient always‑on operation.

---

### 4.6 Docker Environment

A minimal TALI Docker Compose stack includes:
- `tali-core` (Spring Boot application)  
- `postgres`  
- `redis`  
- `neo4j`  
- `minio`  
- `loki` / `prometheus` / `grafana`  
- `mqtt` (EMQX or Mosquitto)  

**Host Requirements:**
- Docker 25+ with BuildKit enabled.  
- Minimum 16 GB allocated to Docker.  
- Shared network mode with MQTT broker for low‑latency IPC.  
- Volume mounts on SSD path for persistence.

---

### 4.7 Optional Performance Enhancements

- **Local LLM Acceleration:** Run smaller models via Ollama or llama.cpp with Metal or CUDA acceleration.  
- **Audio Offloading:** Offload wake‑word and ASR preprocessing to edge microcontrollers (ESP32 S3 + ReSpeaker).  
- **Cache Compression:** Redis and DuckDB caches can be memory‑mapped for faster retrieval.  
- **Graph Parallelism:** Enable Neo4j multi‑threaded query execution for large context maps.

---

### 4.8 Deployment Examples

#### Example A — Standalone Mac Mini (Recommended)
- macOS 15 + Docker 25 + 8‑core M2 CPU  
- Local LLM (4 GB quantized model via Metal)  
- Native audio I/O for voice interaction  
- Grafana & databases containerized.

#### Example B — Linux Server (Full Containerized)
- Ubuntu 24.04 LTS + Docker Compose  
- All components containerized  
- USB mic array passthrough via ALSA  
- GPU (CUDA) for LLM acceleration.

#### Example C — SBC / Edge Deployment (Lightweight)
- Raspberry Pi 5 + active cooling  
- External SSD  
- Remote HA + shared MQTT broker  
- No local LLM (cloud inference fallback).

---

### 4.9 Environmental Recommendations

- **Ambient Noise:** Keep under 45 dB in rooms with voice capture for consistent ASR accuracy.  
- **Lighting:** Avoid flickering LED sources near optical sensors.  
- **Network Stability:** Use UPS‑backed router and switch to maintain uptime.  
- **Physical Placement:** Centralize compute node; minimize Wi‑Fi hops between TALI Core and HA broker.

---

*(Section 4 defines the hardware, peripherals, and environmental expectations for TALI deployments — ensuring reliable operation, low‑latency communication, and stable always‑on performance in home environments.)*

