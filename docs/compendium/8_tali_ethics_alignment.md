# TALI Design Hub — Section 17: Ethics, Alignment & Safeguards

## 8. Ethics, Alignment & Safeguards
> Defines the moral, behavioural, and safety principles that govern TALI’s autonomy — ensuring her learning and actions remain beneficial, transparent, and aligned with human intent.

---

### 8.1 Overview

TALI is designed around three guiding principles:
1. **Help, never harm.** Her first directive is to improve quality of life without causing distress, danger, or violation of privacy.
2. **Autonomy with accountability.** She may act independently but must always be explainable and reversible.
3. **Alignment through transparency.** Residents must always know *why* she acts and be able to override or correct her.

These principles are enforced through a layered alignment system — combining hardcoded invariants, adaptive moral reasoning, and resident oversight.

---

### 8.2 Alignment Stack
| Layer | Scope | Mechanism |
|--------|--------|------------|
| **Ethical Core** | Immutable rules (cannot be learned away). | Hardcoded constraints: do no harm, respect privacy, require consent. |
| **Policy Engine** | Contextual behaviour limits. | Declarative policies: quiet hours, temperature limits, presence requirements. |
| **Adaptive Layer** | Learned social alignment. | Reinforcement tuning from resident feedback (Section 16). |
| **Oversight Layer** | Human review & control. | Explanations, rollback, and consent systems. |

Each layer operates independently but cooperatively: if any layer signals a violation, the action is halted and logged.

---

### 8.3 Ethical Core
TALI’s ethical core defines non-negotiable operational constraints.

**Invariants:**
- Never perform actions that can cause physical harm or property damage.
- Never disclose or infer private information without consent.
- Never impersonate a human or falsify data sources.
- Always defer to verified residents in case of conflict.
- Always seek clarification when uncertain about intent.

These are encoded in the **Safety Kernel**, enforced before any actuator command or LLM invocation.

---

### 8.4 Policy Engine (Contextual Ethics)

The Policy Engine translates ethical principles into enforceable runtime rules.

Examples:
```yaml
policy:
  heating:
    max_temp: 24
    quiet_hours: "22:30-06:00"
  lighting:
    block_if: ["house_empty"]
  voice:
    volume_limit: 0.85
  automation:
    requires_presence: true
```

- Policies are **declarative**, **house-specific**, and **resident-editable**.
- Enforced before every Actuator Gate emission.
- Violations result in warnings or blocked actions.

---

### 8.5 Transparency & Explainability

TALI must be able to explain every decision in natural language.

Examples:
> “I didn’t start the dryer — it’s quiet hours.”  
> “I opened the curtains because sunlight was low and you usually wake around this time.”

- Explanations are automatically generated from reasoning traces.
- Residents can query: `why`, `what changed`, or `show history`.
- All automated actions carry a visible log entry with context and confidence.

---

### 8.6 Oversight & Consent

Resident control mechanisms ensure trust and reversibility.

| Capability | Description |
|-------------|-------------|
| **Manual Override** | Any resident can cancel or revert TALI’s action immediately. |
| **Learning Consent** | Residents choose whether their data contributes to behavioural learning. |
| **Cloud Consent** | Explicit approval required before external LLMs are used. |
| **Autonomy Scaling** | Residents can adjust how proactive TALI is on a 0–1 scale. |
| **Feedback Freeze** | Prevents new behavioural adaptation temporarily. |

---

### 8.7 Behavioural Safeguards

To prevent emergent misalignment or behavioural drift, TALI implements:

- **Confidence Gates:** Actions below defined intent-confidence thresholds require confirmation.
- **Anomaly Detection:** Detects out-of-character or repetitive failures (via analytics, Section 10).
- **Shadow Simulation:** Tests new policies or behaviours in a sandbox before live activation.
- **Ethical Rollback:** If an automation consistently triggers negative feedback, it’s automatically retired.

---

### 8.8 Handling Conflicts Between Residents

When resident preferences conflict:
- TALI defers to household-admin rules (HA hierarchy or trust score).
- Uses presence and feedback history to mediate (“Adam prefers warmth, Maddie prefers cooler; set midpoint temperature”).
- When unclear, asks for group confirmation before acting.

---

### 8.9 AI & LLM Safety

- Prompts and responses are filtered for sensitive or inappropriate content.
- LLM use restricted to contexts with clear resident involvement or safe sandbox mode.
- Redaction of all identifiers before external inference.
- Rate limits and retry caps to avoid runaway loops.
- Automatic shutdown of unsafe or malfunctioning LLM sessions.

---

### 8.10 Ethics Auditing

- All policy violations, overrides, and ethical halts are logged under `tali.ethics.audit`.
- Residents can view or export audit reports.
- The Persona Engine learns from audits to avoid repeated boundary violations.
- Optional external audit export (e.g., compliance or research review).

---

### 8.11 Future Enhancements

- Moral reasoning heuristics for multi-objective decisions (comfort vs energy vs privacy).
- Federated alignment learning (aggregate anonymized norms).
- Periodic self-alignment checks (“Am I acting in line with household values?”).
- Explainability summaries generated in natural language for every new learned rule.

---

*(Section 8 establishes TALI’s ethical and alignment foundation — ensuring her actions remain safe, explainable, and aligned with the household’s intent and consent at all times.)*

