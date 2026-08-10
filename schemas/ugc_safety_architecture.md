# Visual Architecture: The Execution Sequence

This diagram maps the real-time event pipeline, demonstrating how raw telemetry triggers deterministic state mutations before logging to the immutable forensic vault.

```mermaid
flowchart TD
    %% Define Styling
    classDef trigger fill:#1f2937,stroke:#4b5563,color:#f9fafb
    classDef gate fill:#0369a1,stroke:#0284c7,color:#f0f9ff,stroke-width:2px
    classDef stateNormal fill:#14532d,stroke:#22c55e,color:#f0fdf4
    classDef stateWarn fill:#78350f,stroke:#f59e0b,color:#fffbeb
    classDef stateBad fill:#7f1d1d,stroke:#ef4444,color:#fef2f2
    classDef vault fill:#4c1d95,stroke:#8b5cf6,color:#f5f3ff,stroke-width:2px

    subgraph L1 [1. CLIENT / UX LAYER]
        direction LR
        A1[Player Chat/Trade]:::trigger
        A2[NPC Token Generation]:::trigger
        A3[Avatar Movement]:::trigger
    end

    subgraph L2 [2. SENTRY PIPELINE: Deterministic Gates]
        direction TB
        INGEST[API Gateway / Telemetry Ingestion]:::trigger
        G1[Gate 01: Targeting & Recon]:::gate
        G2[Gate 02: Economic Conditioning]:::gate
        G3[Gate 03: Off-Platform Extraction]:::gate
        G4[Gate 04: Kinematic UGC]:::gate
        G5[Gate 05: LLM Token Wrapper]:::gate
        G6[Gate 06: Canary Drift Audit]:::gate
    end

    subgraph L3 [3. THE STATE MACHINE]
        direction TB
        S_NORM([STATE: NORMAL]):::stateNormal
        S_W1([STATE: SENTRY WATCH STAGE 1]):::stateWarn
        S_W2([STATE: SENTRY WATCH STAGE 2]):::stateWarn
        S_QUAR([STATE: QUARANTINED]):::stateBad
    end

    subgraph L4 [4. FORENSIC VAULT]
        direction TB
        SANDBOX[(Conformance Sandbox\nImmutable Ledger)]:::vault
    end

    %% Routing Logic
    A1 --> INGEST
    A2 --> INGEST
    A3 --> INGEST

    INGEST --> G1 & G2 & G3 & G4 & G5 & G6

    %% Gate Outcomes
    G1 -->|Breach| S_W1
    G2 -->|Breach| S_W2
    G3 -->|Breach| S_QUAR
    G4 -->|Breach| S_QUAR
    G5 -->|Breach| S_QUAR
    G6 -->|Drift Detected| S_QUAR

    %% Decay & Recidivism
    S_W1 -.->|Decay 72h| S_NORM
    S_W1 -->|Recidivism r>=3| S_W2
    S_W2 -->|Migration Intent| S_QUAR

    %% Vault Storage
    S_W1 ===>|Log Telemetry| SANDBOX
    S_W2 ===>|Log Telemetry| SANDBOX
    S_QUAR ===>|Forensic Snapshot & Lockout| SANDBOX
```