# TALI Design Hub — Section 4: AI & Learning Engine

## 6. AI & Learning Engine
> Defines how TALI perceives, learns, and adapts from its environment.

### 6.1 Overview
The **Learning Engine** is the heart of TALI’s intelligence. It continuously observes the home’s behaviour, analyses trends, and autonomously constructs or refines automations. Unlike rule-based systems, TALI builds an evolving model of user patterns, context, and intent.

### 6.2 Learning Philosophy
- **Observation first, intervention later.** TALI learns passively before acting.
- **Explainable learning.** Every hypothesis or automation is traceable back to its input signals.
- **Continuous refinement.** Feedback loops ensure learning accuracy improves over time.
- **Safe experimentation.** All new logic is sandbox-tested before live deployment.
- **Human oversight.** Suggestions and actions can always be reviewed, approved, or reverted.

### 6.3 Learning Pipeline
```
[HA / MQTT / DB Data] → [Integration Layer] → [Feature Extractor] → [Learning Engine]
       ↓                         ↓                   ↓                     ↓
  (Events, States)       (Features, Context)   (Training, Testing)   (Hypotheses → Sandbox)
```

| Stage | Component | Purpose |
|--------|------------|----------|
| **Ingestion** | Integration Layer | Normalizes MQTT, HA, and database data into event streams. |
| **Feature Extraction** | Feature Service | Generates structured features from raw events (time, frequency, correlation, energy use, presence). |
| **Model Training** | Learning Engine | Applies heuristic, statistical, or ML-based learning methods to identify routines or triggers. |
| **Hypothesis Evaluation** | Sandbox Manager | Tests new hypotheses in a controlled environment before live activation. |
| **Reflection & Feedback** | Core Orchestrator | Scores accuracy, adjusts weights, and feeds results back into the model. |

### 6.4 Data Inputs
TALI’s intelligence depends on diverse, high-resolution data sources:
- **MQTT Streams:** Zigbee2MQTT, device events, sensors, and switches.
- **Home Assistant API:** Entity states, automations, user presence, schedules.
- **HA Databases (InfluxDB / TimescaleDB):** Time-series data for trend recognition and model validation.
- **Local Databases:** PostgreSQL (state), Redis (real-time cache), Neo4j (context graph), and DuckDB (analytics).

### 6.5 Learning Techniques
| Technique | Description |
|------------|-------------|
| **Statistical Inference** | Detects correlations between user actions and environment changes. |
| **Pattern Mining** | Identifies recurring event sequences and potential routines. |
| **Contextual Mapping** | Uses Neo4j to map relationships between entities, rooms, and occupants. |
| **Anomaly Detection** | Spots deviations from normal behaviour (e.g., device left on). |
| **Reinforcement Loops** | Rewards successful predictions and penalizes false assumptions. |
| **LLM-Assisted Reasoning** | (Optional, with consent) offloads reasoning tasks to cloud or local LLMs for high-level inference. |

### 6.6 Learning Outputs
| Output Type | Description |
|--------------|-------------|
| **Hypotheses** | Candidate automations or behavioural insights awaiting validation. |
| **Learned Automations** | Fully validated routines deployed to TALI’s action registry. |
| **Context Graphs** | Neo4j maps showing entity relationships and behavioural clusters. |
| **Metrics & Reports** | Confidence scores, false-positive rates, and user feedback logs. |

### 6.7 Safety & Testing
- All hypotheses are sandboxed before deployment.
- TALI Core reviews learning cycles for runaway or biased models.
- Validation includes A/B tests where possible (comparing baseline vs new automation).
- Failing models are automatically quarantined for review.
- User approval can be enforced for high-impact automations (e.g., heating, security).

### 6.8 Cloud & Local Model Blending
When enabled, TALI can delegate specific reasoning or model refinement steps to external LLMs:
- **Local Mode:** Uses on-device models (e.g., Ollama, LM Studio, or M-series accelerated LLMs).
- **Cloud Mode:** Sends anonymized feature vectors or model summaries for remote optimization. No PII or identifiable data is ever transmitted.

### 6.9 Continuous Learning Loop
```
Observe → Hypothesize → Test → Reflect → Adapt
```
1. **Observe:** Monitor all available sensors, state changes, and user interactions.
2. **Hypothesize:** Form potential automation or pattern hypotheses.
3. **Test:** Run hypotheses in sandbox and measure outcomes.
4. **Reflect:** Analyse performance, feedback, and confidence metrics.
5. **Adapt:** Promote, modify, or retire automations.

### 6.10 Mathematical & Reinforcement Framework
TALI employs an **evolutionary reinforcement system** for discovering and optimizing automations. The process blends parallel hypothesis simulation, mathematical scoring, and Darwinian selection principles.

#### 6.10.1 Parallel Simulation Model
- Each candidate automation (hypothesis) is deployed in a **dedicated sandbox instance**.
- Sandboxes execute **simultaneously**, using either real historical replay data or live mirrored event streams.
- Each variant operates with slightly different parameters, timing, or conditions to explore the solution space.
- Performance metrics (success rate, energy efficiency, comfort satisfaction, stability) are tracked per sandbox.

#### 6.10.2 Evolutionary Loop
1. **Generate:** Produce a population of hypothesis variants through parameter mutations and recombination of successful prior automations.
2. **Simulate:** Run all candidates in isolated sandboxes in parallel.
3. **Score:** Evaluate based on reward metrics — outcome accuracy, user satisfaction signals, energy cost, failure rate.
4. **Select:** Retain top performers and archive or retire underperformers.
5. **Mutate:** Slightly modify surviving candidates to explore adjacent solution space.
6. **Recombine:** Merge logic components or decision trees between top-performing hypotheses.
7. **Iterate:** Continue the cycle until convergence or plateau.

This creates a **self-evolving automation discovery engine**, capable of optimizing both simple and complex behaviors.

#### 6.10.3 Mathematical Framework (Proposed)
| Aspect | Method |
|---------|--------|
| **Reward Scoring** | Weighted reward function combining outcome precision, user feedback, and system cost. |
| **Search Space Optimization** | Genetic algorithm–style population evolution. |
| **Exploration vs Exploitation** | ε-greedy exploration to balance new variant testing against proven strategies. |
| **Simulation Time Budget** | Dynamic wall-time allocation per sandbox based on prior iteration success. |
| **Result Aggregation** | Statistical confidence intervals computed via bootstrapping across multiple simulations. |

#### 6.10.4 Safety Controls
- Evolutionary runs occur **only in simulation**; no live automation is altered until post-validation.
- Mutation parameters are clamped to safe operational limits (e.g., temperature bounds, device cycles).
- Each evolutionary cycle produces an **audit log** with candidate lineage, reward scores, and selection outcomes.
- Failures are automatically pruned; runaway or degenerate behaviors are quarantined.

#### 6.10.5 Integration with Learning Loop
The evolutionary reinforcement process runs as an extension of the core Observe → Hypothesize → Test → Reflect → Adapt cycle:
```
Observe → Hypothesize → (Evolve → Simulate → Select) → Test → Reflect → Adapt
```
This allows TALI to continuously refine its automations not only through observation, but also through **mathematical evolution** — a process inspired by Darwinian natural selection.

### 6.10.6 Reward & Scoring System
TALI’s reinforcement engine operates **fully autonomously** — requiring no user configuration. It self-discovers the context of devices, environmental parameters, and the appropriate weighting of outcomes based on observed behaviour.

#### 6.10.6.1 Autonomous Context Formation
- TALI analyses event topology from Home Assistant and Neo4j to infer contextual relationships (room, occupant, device role).
- It automatically classifies sensors and actuators, identifying controllable vs observational entities.
- Initial reward weights are derived heuristically from feature importance and correlation analysis.
- Over time, these weights self-adjust through feedback loops and statistical evaluation.

#### 6.10.6.2 Reward Function Design
Each sandboxed hypothesis receives a **composite reward score** \(R\) calculated from multiple dynamically weighted components:

\[
R = w_1 * A + w_2 * E + w_3 * U + w_4 * S + w_5 * C
\]

Where:
- **A (Accuracy):** Degree to which predicted outcome matches real-world result.
- **E (Efficiency):** Energy or resource efficiency improvement relative to baseline.
- **U (User Experience):** Implicit satisfaction score (based on interaction patterns, overrides, or dwell time improvements).
- **S (Stability):** Consistency of automation behaviour under varying conditions.
- **C (Confidence):** Statistical certainty derived from repeated validation cycles.

Weights \(w_1…w_5\) are **self-learned**, adapting per domain (e.g., lighting vs climate vs security).

#### 6.10.6.3 Adaptive Reward Evolution
- The engine continuously recalibrates weights using Bayesian updating and normalization.
- When performance plateaus, mutation introduces new combinations of weights to test alternative scoring landscapes.
- Feedback from failed or low-performing variants is used to shrink or expand parameter search boundaries dynamically.

#### 6.10.6.4 Human Transparency (Not Configuration)
- While users never need to configure models, all reward structures and context maps are transparent through the TALI UI and Grafana dashboards.
- Users can view rationale, lineage, and reward breakdowns for any automation, ensuring explainability.
- Optional manual hints ("I prefer warmer evenings" or "save energy aggressively") can bias weights safely without breaking autonomy.

### 6.11 Future Enhancements
- Federated learning between multiple TALI instances (opt-in).
- Graph-based reasoning via Neo4j traversal queries.
- Self-optimization of feature extraction pipelines.
- Integration with local vision and speech models for richer perception.

---

*(Section 6 now fully details TALI’s autonomous reward and scoring system — a self-calibrating, context-aware framework requiring zero human configuration.)*

