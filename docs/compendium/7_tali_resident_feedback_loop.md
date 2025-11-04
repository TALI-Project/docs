# TALI Design Hub — Section 16: Resident Interaction & Feedback Loop

## 7. Resident Interaction & Feedback Loop
> Defines how TALI interprets, learns from, and adapts to resident reactions — both explicit and implicit — forming the continuous loop between human behaviour and AI learning.

---

### 7.1 Overview

TALI’s intelligence depends on a **closed behavioural feedback loop**. Every action she performs, suggestion she gives, or interaction she initiates feeds into a real-time evaluation of how residents respond. These responses — verbal, behavioural, or environmental — are translated into **reinforcement signals** that refine her decision-making, tone, and autonomy.

This section defines how TALI reads those signals, how they translate into learning adjustments, and how residents maintain control over this process.

---

### 7.2 Feedback Sources

TALI collects reinforcement signals from multiple modalities:

| Source | Example | Feedback Type |
|---------|----------|----------------|
| **Voice/Conversation** | Resident says “Thanks” or “Stop doing that.” | Direct verbal sentiment (positive/negative). |
| **Messenger I/O** | Reacts with 👍 or issues a correction (“not that light”). | Explicit text/emoji feedback. |
| **Action Outcome** | Resident manually undoes an automation. | Implicit negative reinforcement. |
| **Environment** | Expected state didn’t persist (light turned back off). | Contextual negative feedback. |
| **Presence/Usage Patterns** | Residents engage more/less with TALI’s features. | Long-term reward trend. |

---

### 7.3 Reinforcement Mapping

Each feedback signal is normalized to a **reinforcement score** `[-1.0 → 1.0]` and contextualized.

```text
positive verbal / emoji → +0.8 to +1.0
acceptance (no reversal) → +0.4 to +0.6
ignored suggestion → 0.0
manual override → -0.6
explicit rejection ("stop", "no") → -1.0
```

Scores are weighted by **trust**, **intent confidence**, and **resident profile**. Example:
```
final_reward = base_score * trust_factor * (1 - uncertainty)
```

---

### 7.4 Behaviour Adjustment

TALI continuously refines her behaviour using observed reinforcement:
- **Persona Adaptation:** modifies tone, humour, or assertiveness towards each resident.
- **Decision Biasing:** strengthens successful automation strategies, weakens those with repeated reversals.
- **Autonomy Scaling:** gradually increases or decreases proactive actions based on household acceptance.
- **Timing Optimization:** learns optimal notification or action timing from feedback trends.

---

### 7.5 Conversational Clarification Loop

TALI uses a **self-checking dialogue loop** when uncertain:
- If a command is ambiguous or historically misinterpreted, she responds with clarification:  
  > “Did you mean the kitchen or dining light?”
- If ignored or corrected multiple times, she lowers autonomy on that domain until retrained.

This prevents repetitive misfires and preserves trust.

---

### 7.6 Sentiment & Emotion Analysis

- Sentiment derived from tone, volume, lexical choice, and emoji use.
- Emotion estimation (anger, calm, stress, humour) influences tone modulation.
- Negative sentiment triggers soft resets of autonomy weights to avoid overreaction loops.
- Data is transient — **no emotional profiling is stored long-term**; only trend metrics are retained.

---

### 7.7 Resident Profiles & Preferences

Each resident has adaptive weights stored in their **feedback matrix**, forming an evolving behavioural model.

| Field | Description |
|--------|--------------|
| `tone_preference` | Degree of formality and humour tolerance. |
| `autonomy_tolerance` | How freely TALI can act on their behalf. |
| `response_style` | Preferred phrasing or persona variant. |
| `reinforcement_history` | Summary statistics of positive vs. negative interactions. |

Profiles update continuously but can be **reset or frozen** per resident request.

---

### 7.8 Safety & Oversight

- Residents can issue commands: `pause-feedback`, `clear-feedback-history`, or `lock-profile`.
- The Persona Engine filters sarcastic or emotionally charged feedback to prevent model bias.
- Reinforcement updates are sandboxed; extreme outliers trigger manual confirmation before applying.
- Messenger summaries show periodic insights:
  > “You’ve been rejecting lighting automations after 9pm. Should I stop adjusting lights then?”

---

### 7.9 Learning Integration

Feedback loops connect directly to the Learning Engine (Section 4):
- Positive reinforcement → boosts reward signal in simulation cycles.
- Negative reinforcement → adds penalty to policy weight.
- Neutral events → contribute to exploration metrics (when to try new actions).
- Repeated consistent patterns → trigger **policy consolidation**, converting adaptive logic into stable learned behaviours.

---

### 7.10 Transparency & Explainability

TALI maintains transparency in how feedback affects her decisions:
- Messenger and Voice can answer:
  > “Why did you stop pre-heating the bathroom?”  
  → “You cancelled heating three mornings in a row, so I adjusted the schedule.”
- All major behavioural changes logged with cause, resident, and magnitude.
- Grafana panels can visualize reinforcement trends and trust over time.

---

### 7.11 Future Enhancements

- Emotional context fusion (combine voice tone + text sentiment for higher accuracy).
- Adaptive sarcasm detection for improved wit calibration.
- Federated anonymized feedback sharing (aggregate behaviour tuning without PII).
- Conversational trust graph between residents (cross-influence learning).

---

*(Section 7 defines how TALI interprets human feedback — from words, tone, and behaviour — and converts it into safe, adaptive learning signals that continuously shape her personality, autonomy, and performance.)*

