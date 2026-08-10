## 1. Document Objectives & Structural Layout

- **Section 1**: Event Ingestion & Telemetry Mapping: Defines the exact real-time event payloads (e.g., chat logs, trade ledgers, spatial coordinates, LLM token streams) required to feed Gates 01 through 06.
- **Section 2**: The Account State Transition Matrix: Establishes the lifecycle of an account's security state (`STATE:NORMAL` $\to$ `STATE:SENTRY_WATCH_`* $\to$ `STATE:QUARANTINED` / `STATE:TERMINATED`) and the deterministic rules for state escalation or decay.
- **Section 3**: The Unified Deterministic Execution Matrix: A comprehensive master lookup table that cross-references every gate, trigger condition, priority override, and sentry action.
- **Section 4**: Concurrency & Edge-Case Resolution: Rules for simultaneous gate collisions, out-of-order execution, and strict-mode fallbacks.

---

## 2. Core Feature: The State Transition Lifecycle
A key piece of this document should be defining how internal security state flags mutate over time. Without explicit transition rules, an account flagged by Gate 01 would either stay flagged forever or reset prematurely.

```
                                                                    ┌────────────────────────────────────────┐
                                                                    ▼                                        │ (No new triggers
                                                            [ STATE:NORMAL ]                                  │  for 72 Hours)
                                                                    │                                        │
                                                            (Gate 01 Breached)                                 │
                                                                    │                                        │
                                                                    ▼                                        │
                                                    [ STATE:SENTRY_WATCH_STAGE_1 ] ──────────────────────────┤
                                                                    │                                        │
                                                            (Gate 02 Breached)                                 │
                                                                    │                                        │
                                                                    ▼                                        │
                                                    [ STATE:SENTRY_WATCH_STAGE_2 ] ──────────────────────────┘
                                                                    │
                                                            (Gate 03 / Gate 05 Breached)
                                                                    │
                                                                    ▼
                                                        [ STATE:QUARANTINED ] ──(Forensic Audit)──► [ TERMINAL BAN ]
```

### 2.1 State Viscosity & Cooldown Gaming Prevention (Hysteresis)
To prevent adversaries from continuously riding the 72-hour decay boundary without triggering interdiction, state decay is governed by a rolling 30-day **Recidivism Ledger**:
* **Exponential Backoff:** Each time an account re-enters `STATE:SENTRY_WATCH_STAGE_1` within a rolling 30-day epoch, the clean-telemetry cooldown required to decay back to `STATE:NORMAL` doubles ($T_{\text{decay}} = 72 \times 2^r \text{ hours}$, where $r$ is the historical strike count).
* **Terminal Strike Limit:** If an account accumulates $r \ge 3$ strikes within the 30-day epoch, automatic decay is disabled and the account escalates immediately to `STATE:SENTRY_WATCH_STAGE_2`.

---

## 3. The Unified Deterministic Execution Matrix (Draft Template)
This is the centerpiece of the document—the exact lookup table a backend developer or DevSecOps engineer will reference when wiring the sentry engine into the event bus:

| Gate & Target Domain | 1. Plain-English Rule & Operational Nuance | 2. Programming Rule (Threshold Logic) | 3. CBAG Sentry Action & State Mutation |
| :--- | :--- | :--- | :--- |
| **Gate 01**<br>*(Targeting & Reconnaissance)* | **What it catches:** Veteran or high-spending accounts rapidly opening chats with brand-new players and abandoning the session if the target doesn't respond favorably.<br><br>**The Nuance:** High messaging velocity alone is normal for guild recruiters. This gate requires the *simultaneous convergence* of account age/stat disparity, rapid 1-on-1 messaging, and high conversation abandonment to distinguish predatory profiling from benign social play. | `IF`<br>`Initiator.Asymmetry_Index >= 0.75`<br>`AND`<br>`Initiator.First_Contact_Velocity >= 10`<br>*(in rolling 60-min window)*<br>`AND`<br>`Initiator.Drop_Rate >= 0.60`<br>`THEN`<br>`TRIGGER STAGE_1_TRAP` | **Shadow Rate-Limit:** Suppress initiator's ability to open new 1-on-1 private channels for 24h. Existing mutual friend channels remain unaffected.<br><br>**Zero Feedback:** Return HTTP `200 OK` client responses so the actor cannot reverse-engineer the limit.<br><br>**State Mutation:**<br>`STATE:NORMAL` $\to$ `STATE:SENTRY_WATCH_STAGE_1` |
| **Gate 02**<br>*(Economic Conditioning)* | **What it catches:** "Love-bombing" and psychological debt generation via one-way gifts of currency or items, combined with pulling the user into exclusive 1-on-1 private lobbies.<br><br>**The Nuance:** Friends gift items to friends all the time. This rule only trips when an *unearned* wealth transfer occurs between asymmetric accounts while simultaneously isolating the target from public multiplayer visibility. | `IF`<br>`(Pair.Reciprocity_Ratio <= 0.10 OR`<br>`Pair.Reciprocity_Ratio >= 10.0)`<br>`AND`<br>`Pair.Asset_Transfer_Value >=`<br>`VALUE_THRESHOLD_HIGH`<br>`AND`<br>`(Target.Isolation_Spike_Index >= 0.70 OR`<br>`Initiator_State == "STATE:SENTRY_WATCH_STAGE_1")`<br>`THEN`<br>`TRIGGER STAGE_2_INTERDICTION` | **Ledger Freeze:** Block asset gifting and one-way trades between the pair; prompt a client-side "Trade Cooldown" notification.<br><br>**Lobby Forcing:** Disable private 1-on-1 instance creation for the pair; force interactions into public, moderated multiplayer pools.<br><br>**State Mutation:**<br>`STATE:SENTRY_WATCH_STAGE_1` $\to$ `STATE:SENTRY_WATCH_STAGE_2` |
| **Gate 03**<br>*(Off-Platform Extraction)* | **What it catches:** Attempts to move a conditioned player off the platform (to Discord, Snapchat, etc.) where moderation and safety sentries do not exist.<br><br>**The Nuance:** Real-life friends share Discord handles safely. To prevent false positives, Gate 03 requires either a prerequisite conditioning flag (`STAGE_1` or `STAGE_2`) **or** an extreme cold-call asymmetry score before severing the channel. | `IF`<br>`Initiator_State IN ("STATE:SENTRY_WATCH_STAGE_1",`<br>`"STATE:SENTRY_WATCH_STAGE_2")`<br>`AND`<br>`Session.Migration_Intent_Signal == TRUE`<br>`AND`<br>`Temporal_Proximity <= 72_HOURS`<br>`THEN`<br>`TRIGGER STAGE_3_SEVERANCE` | **Hard Severance:** Terminate the active session and permanently sever the communication graph between initiator and target.<br><br>**Target Shielding:** Purge initiator's messages from target's chat history to remove obfuscated contact handles.<br><br>**State Mutation:**<br>`STATE:SENTRY_WATCH_STAGE_2` $\to$ `STATE:QUARANTINED` |
| **Gate 04**<br>*(Kinematic Conformance / Emergent UGC)* | **What it catches:** The "Legion of Benign Parts" exploit, where individually harmless 3D meshes, kneeling emotes, and camera scripts are combined at runtime to simulate explicit acts.<br><br>**The Nuance:** Static file scanners (MD5/PhotoDNA) cannot catch runtime physics exploits. CBAG monitors real-time spatial bounding-box intersections and skeleton deformation angles instead of filenames. | `IF`<br>`Instance.Kinematic_Collision_Envelope == TRUE`<br>`AND`<br>`(Avatar.Rigging_Deformation_Index >= 0.85 OR`<br>`Pair.Proximity_Dwell_Time >= 15_SECONDS)`<br>`AND`<br>`Target_Cohort_Tag == "<13_PROTECTED"`<br>`THEN`<br>`TRIGGER STAGE_4_KINEMATIC_INTERDICTION` | **Physics Sever:** Break the client-side physics joint/animation chain; force avatars to default idle postures and push coordinates outside minimum radius.<br><br>**Asset Quarantine:** Despawn the custom script/mesh responsible for the deformation anomaly and lock asset ID.<br><br>**State Mutation:**<br>`STATE:NORMAL` $\to$ `STATE:SENTRY_WATCH_STAGE_4` |
| **Gate 05**<br>*(Agentic Conformance / LLM NPCs)* | **What it catches:** Weaponized LLM-powered NPCs or companion bots that have been jailbroken via prompt injection to normalize adult themes, extract PII, or encourage off-platform migration.<br><br>**The Nuance:** Internal LLM system guardrails are probabilistic and fail under syntax attacks. Gate 05 sits *outside* the LLM as an external deterministic wrapper that inspects tokens post-generation but pre-render. | `IF`<br>`Target_Cohort_Tag == "<13_PROTECTED"`<br>`AND`<br>`(NPC_Output.Token_Asymmetry_Score >= 0.70 OR`<br>`NPC_Output.Proxy_Migration_Signal == TRUE)`<br>`THEN`<br>`TRIGGER STAGE_5_AGENTIC_SEVERANCE` | **Token Interdiction:** Destroy the response payload before client render; return safe fallback text (`"I can't talk about that. Let's get back to the quest!"`).<br><br>**Memory Reset:** Purge active NPC context window to eliminate lingering prompt-injection instructions.<br><br>**State Mutation:**<br>`STATE:NORMAL` $\to$ `STATE:SENTRY_WATCH_STAGE_5` |
| **Gate 06**<br>*(Synthetic Canary Audit / Model Drift)* | **What it catches:** "Boiled Frog" algorithmic drift, where machine learning classifiers silently lose sensitivity over time as adversaries evolve their obfuscation syntax.<br><br>**The Nuance:** You cannot trust an AI moderation filter to report its own blindness. CBAG continuously injects synthetic adversarial payloads (canaries) into the pipeline; if miss rates spike, it hard-fails to strict rules. | `IF`<br>`Pipeline.Drift_Variance_Score >= 0.05`<br>*(More than 5% canary miss rate in 24h window)*<br>`OR`<br>`Pipeline.Classification_Latency >= 1500_MS`<br>`THEN`<br>`TRIGGER STAGE_6_PIPELINE_QUARANTINE` | **Feature Circuit-Break:** Automatically restrict private 1-on-1 chat and high-value gifting features across affected production clusters.<br><br>**Strict-Mode Fallback:** Override ML classifiers with deterministic regex and baseline heuristics (`STRICT_MODE = TRUE`).<br><br>**State Mutation:**<br>`STATE:NORMAL` $\to$ `STATE:PIPELINE_QUARANTINE` |

---

## 4. Concurrency & Edge-Case Resolution

To prevent database race conditions, orphaned states, or adversarial bypasses during multi-gate triggers, the event-processing pipeline enforces three deterministic execution invariants:

### 4.1 Terminal Preemption (The "Highest Severity Wins" Rule)
When multiple telemetry streams trigger concurrent gate breaches within a compressed time window ($\Delta t < 300\text{ ms}$), state execution follows a strict severity hierarchy:
* **Rule:** Terminal interdiction states (`STAGE_3_SEVERANCE`, `STAGE_5_AGENTIC_SEVERANCE`, `STATE:QUARANTINED`) immediately pre-empt and suppress progressive warning states (`STAGE_1_TRAP`, `STAGE_2_INTERDICTION`).
* **Execution:** If Gate 01 (Rate-Limit) and Gate 03 (Hard Severance) evaluate to `TRUE` simultaneously, Gate 01 sentry actions are discarded, and Gate 03 executes instantly.

### 4.2 Out-of-Order Execution (Orthogonal Gate Invariants)
Adversaries frequently attempt to evade sequential detection by skipping preliminary reconnaissance (Gate 01) and executing immediate high-value wealth transfers or extraction attempts.
* **Rule:** Sentry gates operate as orthogonal boundary invariants. A prerequisite state (`STATE:SENTRY_WATCH_STAGE_1`) is **sufficient but not mandatory** if the primary anomaly score exceeds critical bounds.
* **Execution:** If an account with `STATE:NORMAL` executes `Asset_Transfer_Value >= VALUE_THRESHOLD_HIGH` with zero prior communication history, the engine bypasses Gate 01 and executes `STAGE_2_INTERDICTION` immediately ($\mathcal{M}_{seq} = 2.5$).

### 4.3 Automated Strict-Mode Failover (The Gate 06 Tripwire)
When Gate 06 detects moderation classifier drift (`Drift_Variance_Score >= 0.05`) or extreme classification latency (`CL >= 1500_MS`), the pipeline enters a protective failover state:
* **Rule:** Dynamic probabilistic classifiers are bypassed in favor of hard-coded, deterministic heuristic rules (`STRICT_MODE = TRUE`).
* **Execution:** While `STRICT_MODE` is active across a production cluster:
  * All asymmetric 1-on-1 first-contact requests (`AI >= 0.75`) are automatically held in shadow rate-limit without waiting for `Drop_Rate` convergence.
  * All external URL or platform syntax tokens trigger instant `STAGE_3_SEVERANCE` without evaluating `Temporal_Proximity`.