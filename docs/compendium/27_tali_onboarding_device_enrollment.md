# TALI Design Hub — Section 27: Onboarding & Device Enrollment

## 27. Onboarding & Device Enrollment
> Defines the process by which new residents, devices, and rooms are securely introduced into TALI’s ecosystem — ensuring proper identity linking, access control, and context mapping.

---

### 27.1 Overview

Onboarding in TALI covers three major domains:
1. **Resident Enrollment:** Identifying and personalizing user profiles.  
2. **Device Enrollment:** Securely registering sensors, microphones, and actuators.  
3. **Room & Context Setup:** Mapping physical locations and contextual relationships for automation logic.

Each onboarding action is interactive, guided by TALI’s Persona layer, and validated through the Policy Engine to prevent misconfiguration or privilege escalation.

---

### 27.2 Resident Enrollment

#### 27.2.1 Discovery & Authentication
- New residents are introduced via the Messenger or Voice channel.  
- TALI generates a unique Resident ID linked to a verified communication endpoint (Discord, mobile device, etc.).  
- Optional multi-factor validation (voice embedding + device token) ensures identity.

#### 27.2.2 Profile Initialization
- Residents receive a **personal overlay profile** derived from the base persona.  
- Default behavioural parameters (humour, empathy, authority) initialized.  
- TALI requests optional personalization data: preferred language, wake phrase, communication style.

#### 27.2.3 Permissions & Roles
| Role | Capabilities |
|------|---------------|
| **Admin** | Full configuration control and policy overrides. |
| **Resident** | Normal operation, can influence learning behaviour. |
| **Guest** | Limited control; no access to history or policy settings. |

Role assignment is stored in PostgreSQL and mirrored to the Policy Engine for runtime enforcement.

---

### 27.3 Device Enrollment

#### 27.3.1 Discovery Process
- Devices discovered automatically through Home Assistant’s MQTT auto-discovery or Matter pairing.  
- For manual registration, residents can instruct TALI:  
  > “Add new device in the kitchen.”  
  TALI listens for matching MQTT join events or Matter advertisements.

#### 27.3.2 Certificate & Identity Issuance
- Upon discovery, TALI issues a **device certificate** via its internal CA or defers to HA’s certificate authority.  
- Each device is bound to a unique topic path and access scope (publish, subscribe, or both).  
- Device metadata (model, firmware, room, purpose) stored in the Neo4j context graph.

#### 27.3.3 Validation & Testing
- Device pings verified for health.  
- Basic function test (e.g., light toggle, TRV readout).  
- Device added to **quarantine mode** until confirmed operational for 24h without errors.

---

### 27.4 Room & Context Setup

Rooms act as the core contextual anchor for devices, presence, and automation logic.

| Step | Action | Notes |
|------|---------|-------|
| **1. Detection** | HA entity metadata or resident declaration identifies room. | Example: “This sensor is in the bedroom.” |
| **2. Context Graph Update** | TALI adds relationships in Neo4j between device, resident, and room node. | Enables situational awareness. |
| **3. Presence Calibration** | TALI observes mmWave and Bluetooth presence patterns to associate residents. | Forms the baseline for behavioural context. |
| **4. Confirmation** | Persona verbally confirms mapping. | “Got it — that motion sensor is now part of the kitchen.” |

---

### 27.5 Voice Device Enrollment

- ESP microphone nodes identified by unique MQTT client IDs.  
- TALI prompts residents to speak a short calibration phrase for **voice embedding enrollment**.  
- Five samples minimum required for confident speaker recognition.  
- Enrollment confirmation stored locally with anonymized embeddings.

---

### 27.6 Automation Security & Consent

During onboarding, all new devices and residents are registered under **restricted permissions** until explicitly approved.  
TALI announces new additions verbally or via Messenger:
> “A new sensor was detected: Living Room TRV. Should I add it to your climate group?”

If no approval is received, the device remains isolated in the **Pending Zone** of the context graph.

---

### 27.7 Data Synchronization & Backups

- Enrollment data synchronized to PostgreSQL and Neo4j nightly.  
- Snapshot backups stored in MinIO object storage.  
- Restores automatically reconcile device IDs with latest MQTT topology.  

---

### 27.8 Edge Cases

| Scenario | Handling |
|-----------|-----------|
| **Duplicate Device ID** | Conflict resolution via hash comparison; old device retired. |
| **Resident Leaves Home** | Profile archived but not deleted; retained for analytics unless purged by admin. |
| **Device Offline > 7 days** | Flagged for review; TALI may prompt admin to confirm removal. |

---

### 27.9 Guided Setup Mode

For new installations, TALI runs an **interactive onboarding wizard**:
1. Scans the HA instance for connected devices.  
2. Prompts the admin to assign rooms and residents.  
3. Calibrates presence and microphones.  
4. Performs baseline sensor readings to establish environmental norms.  

Progress displayed via web dashboard or Messenger channel for clarity.

---

*(Section 27 defines the complete onboarding and enrollment process for residents, devices, and rooms — combining security, personalization, and guided context mapping to integrate new elements into TALI’s intelligent home ecosystem safely and intuitively.)*