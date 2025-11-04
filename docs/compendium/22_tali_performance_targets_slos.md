# TALI Design Hub — Section 25: Performance Targets & SLOs

## 22. Performance Targets & Service Level Objectives (SLOs)
> Defines the quantitative goals for responsiveness, reliability, and efficiency across TALI’s subsystems to ensure predictable, real-time household operation.

---

### 22.1 Overview

TALI’s core design prioritizes **low-latency response**, **high reliability**, and **predictable throughput**. These Service Level Objectives (SLOs) provide measurable targets that guide both development and runtime monitoring. Metrics are collected continuously via Prometheus and evaluated against these SLOs in Grafana.

TALI must operate smoothly even on modest home hardware — therefore, performance goals are expressed as ranges, accommodating both high-end and minimal configurations.

---

### 22.2 Global System Objectives

| Category | Metric | Target | Notes |
|-----------|---------|---------|-------|
| **System Availability** | Uptime | ≥ 99.5% | Measured over 30 days; excludes planned maintenance. |
| **Startup Time** | Full boot (core + services) | ≤ 90 seconds | Includes database and broker initialization. |
| **Resource Utilization** | CPU Load | ≤ 65% sustained | Under normal automation and voice operation. |
| **Memory Use** | Baseline footprint | ≤ 8 GB active | On 16 GB system; may scale higher during learning tasks. |
| **Power Consumption** | Standby / Active | ≤ 10W / 40W | Based on M2 Mac Mini baseline. |

---

### 22.3 Communication Latency Targets

| Path | Ideal | Acceptable | Description |
|------|--------|-------------|-------------|
| **ESP Mic → MQTT Broker** | ≤ 5 ms | ≤ 10 ms | Raw audio streaming and wake/VAD events. |
| **Broker → TALI Core** | ≤ 10 ms | ≤ 25 ms | Sensor and state updates. |
| **TALI Core → HA Service Call** | ≤ 20 ms | ≤ 50 ms | Control execution or mimic command. |
| **LLM Query (Local)** | ≤ 200 ms | ≤ 500 ms | Local inference with quantized model. |
| **LLM Query (Cloud)** | ≤ 1.5 s | ≤ 2.5 s | Cloud roundtrip including encryption overhead. |
| **Voice Response (STT → TTS)** | ≤ 1.5 s | ≤ 2.5 s | Total conversational response time. |

---

### 22.4 Data Throughput & Storage Goals

| Subsystem | Metric | Target | Notes |
|------------|---------|---------|-------|
| **MQTT Bus** | Message throughput | ≥ 5k msg/s | Burst capacity during event storms. |
| **Database (PostgreSQL)** | Query latency | ≤ 20 ms avg | Indexed queries under typical load. |
| **Redis Cache** | Read/write ops | ≥ 50k ops/s | Real-time caching for votes, presence, etc. |
| **Neo4j Graph** | Context traversal | ≤ 100 ms | Query path ≤ 10 hops. |
| **DuckDB Analytics** | Query aggregation | ≤ 500 ms | In-memory vector queries. |

---

### 22.5 Voice & Interaction Responsiveness

| Layer | Metric | Target |
|--------|---------|--------|
| **Wake Word Detection** | Trigger to recognition start | ≤ 250 ms |
| **Speech Recognition (ASR)** | Phrase-to-text latency | ≤ 700 ms |
| **Speaker Identification** | Match to profile | ≤ 100 ms |
| **Intent Processing** | ASR → Intent Resolution | ≤ 150 ms |
| **Persona Reply Generation** | Intent → Text | ≤ 300 ms (local) / ≤ 1 s (cloud) |
| **Text-to-Speech Output** | Text → Audio playback | ≤ 500 ms |

Total end-to-end conversational latency target: **≤ 2.5 seconds (local)** or **≤ 4 seconds (cloud fallback)**.

---

### 22.6 Learning & Simulation Performance

| Task | Metric | Target | Notes |
|------|---------|---------|-------|
| **Sandbox Iteration** | Simulation cycle time | ≤ 500 ms | Parallel across 8–16 threads. |
| **Policy Evaluation** | Candidate testing throughput | ≥ 1k/s | Optimized via parallel mutation. |
| **Model Training Epoch** | Small model convergence | ≤ 5 min | For behaviour-weight updates. |
| **Prediction Service** | Query response | ≤ 150 ms | Cached for live decision use. |

---

### 22.7 Reliability & Fault Recovery

| Failure Mode | Recovery Target | Behaviour |
|---------------|----------------|------------|
| **Service Crash (non-core)** | Auto-restart ≤ 10s | Spring Boot health monitor triggers restart. |
| **Database Unavailable** | Retry/backoff within 60s | Redis fallback for read-only ops. |
| **MQTT Disconnect** | Reconnect ≤ 5s | Persistent sessions + retained topics. |
| **LLM Timeout (cloud)** | Fallback ≤ 2s | Switch to local summary or cached response. |
| **High Load (>80%)** | Recovery ≤ 30s | Load-shedding, defer low-priority tasks. |

---

### 22.8 Monitoring & Enforcement

- All SLO metrics exported via Prometheus endpoints (`/actuator/prometheus`).  
- Grafana dashboards display rolling averages and trend thresholds.  
- Alerts trigger when 3 consecutive 5-minute intervals exceed targets.  
- AI-driven anomaly detection flags sustained deviations for tuning or self-optimization.  

---

### 22.9 Stretch Goals (Future Revisions)

- Sub-100ms round-trip for local intent execution (MQTT → HA → response).  
- <1.5s end-to-end conversational turnaround including LLM.  
- Predictive resource pre-scaling during high-traffic periods.  
- Adaptive learning throttling based on CPU temperature and power draw.  

---

*(Section 22 defines quantitative performance and reliability goals for TALI — ensuring responsive interaction, efficient learning, and consistent household automation performance across both high-end and modest hardware setups.)*