# TALI Design Hub — Section 29: Disaster Recovery & Runbooks

## 24. Disaster Recovery & Runbooks
> Defines the procedures, safeguards, and operational guidelines for recovering from system failures, database corruption, or critical service outages within the TALI ecosystem.

---

### 24.1 Overview

TALI is designed for resilience and continuity. In the event of catastrophic failure — whether due to data corruption, hardware loss, or human error — recovery procedures ensure minimal downtime, data integrity, and safe reactivation of automations.

This section outlines the recovery lifecycle, including automated mechanisms (self-repair, snapshots) and human-guided runbooks for manual restoration or containment.

---

### 24.2 Recovery Objectives

| Metric | Target | Description |
|---------|---------|-------------|
| **RPO (Recovery Point Objective)** | ≤ 24 hours | Maximum acceptable data loss interval. |
| **RTO (Recovery Time Objective)** | ≤ 60 minutes | Maximum downtime before TALI Core returns to operational state. |
| **Backup Retention** | 30 days | Rolling daily snapshots stored in MinIO or external backup volume. |
| **System Availability** | ≥ 99.5% | Maintained via automated restart and redundancy mechanisms. |

---

### 24.3 Automatic Recovery Framework

TALI’s **Self-Maintenance System** (see Section 14) manages automated recovery:
- Periodic health checks via Spring Actuator endpoints.  
- Automatic container restarts using Docker health probes.  
- Snapshot restore on startup if database integrity check fails.  
- Sandbox rehydration for interrupted simulations.  
- Model registry revalidation for integrity and signature verification.

---

### 24.4 Backup & Snapshot Strategy

**Daily Backups:**
- PostgreSQL and Neo4j databases dumped daily to MinIO.  
- Redis ephemeral data serialized hourly for state recovery.  
- Object storage (models, metrics) versioned via timestamped directories.  

**Snapshot Validation:**
- TALI verifies each backup post-write using checksum comparison.  
- Failed validations trigger an alert in Grafana + Messenger.  

**Off-Site Replication (Optional):**
- Backups can be mirrored to a secondary system (NAS, external drive, or cloud bucket).  
- Secure sync via HTTPS or rclone with AES-256 encryption.

---

### 24.5 Manual Recovery Runbooks

#### 24.5.1 Full System Restore
1. Stop all running containers (`docker compose down`).  
2. Restore latest verified snapshot from `/backups/YYYY-MM-DD/`.  
3. Validate database integrity (`pg_verify_checksums`, `neo4j-admin check-consistency`).  
4. Relaunch containers (`docker compose up -d`).  
5. Verify health via `/actuator/health` endpoint.  
6. Confirm event ingestion and MQTT reconnection.  

#### 24.5.2 Partial Database Recovery
- Restore individual databases using respective tools (e.g., `pg_restore`, `neo4j-admin load`).  
- Update environment variables if schema versions differ.  
- Trigger `tali-core` resynchronization job (`/actuator/resync`).

#### 24.5.3 LLM or Persona Corruption
- Replace corrupted models from MinIO registry snapshot.  
- Validate SHA-256 signature matches manifest.  
- Reload Persona definitions via `/persona/reload` endpoint.  

---

### 24.6 Containment & Fail-Safe Mode

If TALI detects cascading faults or policy breaches:
- Enter **Safe Mode** — automations disabled, sensors and logs remain active.  
- Core process isolates unverified behaviours or learning weights.  
- Notifies admin through Messenger and dashboard alert.  
- Requires manual confirmation to resume normal operation.

---

### 24.7 Network & MQTT Broker Recovery

| Scenario | Procedure |
|-----------|------------|
| **Broker Crash** | Restart container; retained messages will restore last state. |
| **TLS Certificate Expired** | Regenerate via TALI or HA CA; redeploy broker; reconnect clients. |
| **Network Partition** | TALI caches events in Redis and replays upon reconnection. |
| **Duplicate Client IDs** | Broker will disconnect duplicates; investigate using retained logs. |

---

### 24.8 Hardware Replacement

When moving TALI to new hardware:
1. Export `.env` and configuration YAML.  
2. Copy `/data` and `/backups` directories.  
3. Deploy fresh Docker environment.  
4. Restore snapshots and relink audio devices or MQTT brokers.  
5. Validate LLM and Persona integrity before reactivation.  

---

### 24.9 Monitoring & Alerting

- Prometheus metrics track uptime, recovery frequency, and data consistency.  
- Grafana panels display backup status and restoration success rates.  
- Anomaly detection warns if RTO or RPO targets are exceeded.  
- Optional integration with Home Assistant notifications for redundancy alerts.

---

### 24.10 Future Improvements

- Distributed backup scheduler for multi-home setups.  
- Transactional recovery with WAL (Write-Ahead Log) replay.  
- Incremental model recovery for partial corruption events.  
- AI-assisted incident summarization for self-documenting outages.  

---

*(Section 24 defines TALI’s disaster recovery and operational runbooks — ensuring predictable restoration, containment, and resilience in the face of data loss, hardware failure, or unexpected behavioural faults.)*