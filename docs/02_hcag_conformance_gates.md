# 02. Human-Centered Agentic Governance (HCAG) Conformance Gates: Deterministic Interdiction Logic

---

## 1. Executive Summary: From Inference to Deterministic Control

Legacy safety architectures attempt to infer bad intent from static content strings. The Human-Centric Agentic Governance (HCAG) framework replaces inference with deterministic state gates.

By evaluating continuous telemetry across three mechanical dimensions—Account Asymmetry, Economic Reciprocity, and Graph Topology—the system enforces hard boundaries that break the operational requirements of online grooming loops without relying on text sentiment or single-timestamp biometric checks.

---

## 2. The Algorithmic Engine: The Composite Anomaly Formula


Beneath the individual gate thresholds, the governance engine continuously calculates a deterministic Structural Anomaly Score ($\mathcal{S}_{total}$) for any interacting account pair $(i, j)$ at time $t$. This composite score scales baseline behavioral risk using velocity and sequence modifiers:

$$\mathcal{S}_{total} = \left[ w_1 \cdot \text{AI}_{i,j} + w_2 \cdot (1 - \text{RR}_{i,j}) + w_3 \cdot \text{ISI}_{i,j} \right] \times \Phi(\Delta t) \times \mathcal{M}_{seq}$$

---

### Variable Definitions & Mathematical Bounds

| Symbol | Variable Name | Definition & Mathematical Range |
| :--- | :--- | :--- |
| $\text{AI}_{i,j}$ | **Asymmetry Index** | Measures account disparity (age, playtime, spend tier) between initiator $i$ and target $j$. Range: $[0.0, 1.0]$. |
| $\text{RR}_{i,j}$ | **Reciprocity Ratio** | Measures economic balance. Inverted as $(1 - \text{RR})$ so one-way gifting approaches $1.0$. Range: $[0.0, 1.0]$. |
| $\text{ISI}_{i,j}$ | **Isolation Spike Index** | Measures the percentage of target $j$'s playtime spent exclusively with $i$ relative to baseline. Range: $[0.0, 1.0]$. |
| $w_1, w_2, w_3$ | **Orthogonal Weights** | Explicit governance coefficients where $\sum w_n = 1.0$, prioritizing which vector (social, economic, spatial) carries the most risk. |
| $\Phi(\Delta t)$ | **Velocity Function** | A time-compression multiplier. Spikes when multiple gate conditions occur in rapid succession ($\Delta t \to 0$). |
| $\mathcal{M}_{seq}$ | **Sequence Multiplier** | Step-order penalty. $\mathcal{M}_{seq} = 1.0$ for sequential progression, $\ge 2.5$ for skipped steps (e.g., cold-call gifting), and $\infty$ for instant off-platform routing (`MIS = TRUE`). |

---

### Step-Function Gate Enforcement

The deterministic IF/THEN gates function as step-function thresholds ($\theta$) applied to $\mathcal{S}_{total}$:

$$\text{Gate Action} = \begin{cases} \text{No Action}, & \text{if } \mathcal{S}_{total} < \theta_1 \\\\ \text{Stage 1 Trap (Rate-Limit)}, & \text{if } \theta_1 \le \mathcal{S}_{total} < \theta_2 \\\\ \text{Stage 2 Interdiction (Ledger Freeze)}, & \text{if } \theta_2 \le \mathcal{S}_{total} < \theta_3 \\\\ \text{Stage 3 Severance (Terminal Kill)}, & \text{if } \mathcal{S}_{total} \ge \theta_3 \end{cases}$$

---

## 3. Gate 01: The Asymmetry & Velocity Gate (Stage 1 — Targeting)

### Core Engineering Objective

Intercept reconnaissance and "fishing" behavior before rapport can be established. This gate evaluates the disparity between two interacting nodes and the velocity of first-contact outreach.

### Evaluated Variables

- `Asymmetry_Index (AI)`: Normalized delta between Initiator and Target based on account age, total playtime, and historical spend tier ($0.0 - 1.0$).
- `First_Contact_Velocity (FCV)`: Number of unique 1-on-1 private chat sessions initiated by an account within a rolling time window $T_1$.
- `Drop_Rate (DR)`: Percentage of newly initiated sessions abandoned within $<3$ messages when the target fails to respond favorably.

### Deterministic IF/THEN Logic
```
IF
  Initiator.Asymmetry_Index >= 0.75  [High Veteran-to-Newbie Disparity]
  AND
  Initiator.First_Contact_Velocity >= 10  [in Rolling 60-Minute Window]
  AND
  Initiator.Drop_Rate >= 0.60  [High Abandonment/Filtering Pattern]
THEN
  EXECUTE SENTRY_ACTION: STAGE_1_TRAP
```

### Sentry Actions (STAGE_1_TRAP)
- **Action 01 (Shadow Rate-Limit)**: Suppress Initiator's ability to open new 1-on-1 private channels for 24 hours. Existing mutual friend channels remain unaffected.
- **Action 02 (State Flagging)**: Tag Initiator account with internal state `STATE:SENTRY_WATCH_STAGE_1`.
- **Action 03 (Zero Feedback)**: Return standard `200 OK` client-side responses so the actor cannot reverse-engineer the rate-limit threshold.

> **Anti-Gaming Note**: *State decay is non-linear; accounts that repeatedly trigger Gate 01 within a rolling 30-day epoch face exponential cooldown backoffs (72h → 144h → 288h), with a third strike forcing automatic escalation to Gate 02 interdiction.*

---

## 4. Gate 02: The Economic Reciprocity & Isolation Gate (Stage 2 — Conditioning)

### Core Engineering Objective
Neutralize "love-bombing" and psychological debt generation by monitoring asymmetric wealth transfers and sudden 1-on-1 social isolation.

### Evaluated Variables
- `Reciprocity_Ratio (RR)`: Ratio of total market value *received* by Account A versus total market value *given* to Account B over rolling window $T_2$.
- `Isolation_Spike_Index (ISI)`: Percentage of Target's total online playtime spent exclusively in 1-on-1 sessions with Initiator compared to historical baseline.
- `Initiator_State`: Current security state tag of the initiating account.

### Deterministic IF/THEN Logic
```
IF
  (Pair.Reciprocity_Ratio <= 0.10 OR Pair.Reciprocity_Ratio >= 10.0) # Lopsided Transfer
  AND
  Pair.Asset_Transfer_Value >= VALUE_THRESHOLD_HIGH
  AND
  (Target.Isolation_Spike_Index >= 0.70 OR Initiator_State == "STATE:SENTRY_WATCH_STAGE_1")
THEN
  EXECUTE SENTRY_ACTION: STAGE_2_INTERDICTION

```

### Sentry Actions (STAGE_2_INTERDICTION)
- **Action 01 (Ledger Freeze)**: Block asset gifting and one-way trades between the pair; prompt a "Trade Cooldown" notification.
- **Action 02 (Graph Re-Integration)**: Disable private 1-on-1 instance creation for the pair; force interactions into public, moderated multiplayer lobbies.
- **Action 03 (State Flagging)**: Escalate Initiator state to `STATE:SENTRY_WATCH_STAGE_2`.

---

## 5. Gate 03: The Handoff & Migration Gate (Stage 3 — Extraction)

### Core Engineering Objective
Prevent off-platform extraction to unmoderated channels (e.g., Discord, Snapchat) by pairing historical state flags with any external routing attempt.

### Evaluated Variables
- `Initiator_State`: Requires prerequisite flag (`STATE:SENTRY_WATCH_STAGE_1` OR `STATE:SENTRY_WATCH_STAGE_2`).
- `Migration_Intent_Signal (MIS)`: Boolean flag triggered by syntax degradation patterns (leetspeak, spaced characters), QR code/image uploads, or external platform tokens.
- `Temporal_Proximity`: Time elapsed between Stage 2 economic conditioning activity and the detected MIS.

### Deterministic IF/THEN Logic
```
IF
  Initiator_State IN ("STATE:SENTRY_WATCH_STAGE_1", "STATE:SENTRY_WATCH_STAGE_2")
  AND
  Session.Migration_Intent_Signal == TRUE
  AND
  Temporal_Proximity <= 72_HOURS
THEN
  EXECUTE SENTRY_ACTION: STAGE_3_SEVERANCE

```

### Sentry Actions (STAGE_3_SEVERANCE)
- **Action 01 (Hard Severance)**: Immediately terminate the active session and permanently sever the communication graph between Initiator and Target.
- **Action 02 (Quarantine)**: Lock Initiator account from all social, trading, and co-play features pending forensic audit.
- **Action 03 (Target Shielding)**: Purge Initiator's messages from Target's chat history to remove obfuscated contact handles or instructions.

---

## 6. Gate Collision & Priority Override Logic

In the event of concurrent or highly compressed gate triggers ($\Delta t < 300\text{ seconds}$ across multiple stages), the governance engine enforces deterministic priority override rules to prevent race conditions:

1.  **Terminal Preemption (The "Highest Severity Wins" Rule)**: Terminal interdiction gates (STAGE_3_SEVERANCE) automatically override and suppress progressive state actions (STAGE_1_TRAP, STAGE_2_INTERDICTION).
2.  **Velocity Escalation**: Simultaneous multi-gate breaches indicate automated scripting or bot-driven payload delivery rather than human-driven operant conditioning.
3.  **Escalated Payload**: When Gate 01, Gate 02, and Gate 03 trigger concurrently, standard quarantine is upgraded to a Hardware-Level Device Ban and immediate purge of all active network sessions tied to the originating fingerprint.

---

## 7. Orthogonal Gate Invariants & Out-of-Order Execution

Legacy pipelines often fail when attackers execute steps out of sequence. HCAG Conformance Gates operate as orthogonal boundary invariants — each gate evaluates independent structural conditions that do not strictly require prerequisite sequential flags to trigger interdiction:

1.   **Unearned Trust Anomaly** (Skipping Gate 01 $\to$ Gate 02): Executing high-value asymmetric asset transfers (`Asset_Transfer_Value >= VALUE_THRESHOLD_HIGH`) with zero prior communication or co-play history applies a multiplier ($\mathcal{M}_{seq} = 2.5$) to the anomaly score, triggering STAGE_2_INTERDICTION immediately without Gate 01 firing.

2.   **Cold-Call Solicitation** (Skipping Gates 01/02 $\to$ Gate 03): When `Migration_Intent_Signal == TRUE` occurs between an asymmetric account pair ($\text{AI} \ge 0.75$) lacking established social or economic tenure, the system bypasses prerequisite state checks and executes a standalone `STAGE_3_SEVERANCE`.

3.   **The Principle of Social Invariance**: Normal human social progression requires interaction tenure before significant wealth transfer or external channel migration. Reversing or omitting these steps is treated as a deterministic indicator of adversarial evasion.

---

## 8. Architectural Summary Table


| Gate | Stage Intercepted | Primary Trigger Metric | Enforcement Mechanism | Bypass Difficulty |
| :--- | :--- | :--- | :--- | :--- |
| **Gate 01** | Stage 1: Targeting | Asymmetry Index + Contact Velocity | Silent Private Channel Rate-Limit | **High:** Requires abandoning rapid filtering and playing normally. |
| **Gate 02** | Stage 2: Conditioning | Reciprocity Ratio (<0.10) + Isolation Spike | Gifting Freeze + Public Lobby Forcing | **High:** Eliminates the ability to create unearned psychological debt. |
| **Gate 03** | Stage 3: Extraction | Prior State Flag + Migration Intent Signal | Instant Session Sever + Account Quarantine | **Absolute:** Severance occurs automatically before contact info is exchanged. |