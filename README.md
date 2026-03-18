# PL-Genesis Genomics Safety Co-Pilot

> **PL Genesis: Frontiers of Collaboration Hackathon** | Track: Flow · NEAR · Crypto · Infrastructure & Digital Rights · Agents With Receipts

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![AGI Corporation](https://img.shields.io/badge/org-AGI--Corporation-blue)](https://github.com/AGI-Corporation)
[![Hackathon](https://img.shields.io/badge/hackathon-PL%20Genesis-purple)](https://pl-genesis-frontiers-of-collaboration-hackathon.devspot.app)

## Overview

A **safety and explainability layer for on-chain financial operations in genomics labs and DeSci orgs**. Every transaction is simulated, risk-scored, and linked back to a concrete study or grant before being signed. Full cryptographically-verifiable receipts are stored for every action.

**Core repos integrated:**
- [`CMMC`](https://github.com/AGI-Corporation/CMMC) — regulatory constraints, risk policies, compliance rules
- [`Route.X`](https://github.com/AGI-Corporation/Route.X) — simulation & approval workflow pipeline
- [`genomic-go-platform`](https://github.com/AGI-Corporation/genomic-go-platform) — links transactions to studies/cohorts
- [`Sapient.x`](https://github.com/AGI-Corporation/Sapient.x) — program metadata for human-readable context
- [`AGI-Framework`](https://github.com/AGI-Corporation/AGI-Framework) — "Safety Copilot" agent explaining calldata

---

## Goals

- Add a safety and explainability layer to every on-chain financial action for genomics labs
- Tie every transaction back to actual studies and grant programs
- Demonstrate policy-driven, auditable agent guidance using CMMC
- Generate tamper-evident receipts linking funds to concrete research
- Support Flow and NEAR for meta-transactions and intent-based DeFi

---

## Hackathon Tracks & Sponsors

| Sponsor / Track | How this project qualifies |
|---|---|
| **Flow** | Meta-transactions and walletless UX for lab finance operations |
| **NEAR** | Intent-based transactions; AI that works for you |
| **Crypto** | Consumer DeFi safety; risk management for public goods orgs |
| **Infrastructure & Digital Rights** | Policy governance and verifiability for all financial actions |
| **Agents With Receipts** | Every agent action produces a cryptographically-signed receipt |
| **Fresh/Existing Code** | Reuses CMMC, Route.X, genomic-go-platform, Sapient.x |

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│     Web UI / Browser Extension / CLI                    │
│  (paste tx payload + choose linked study/grant)          │
└──────────────────────┬──────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────┐
│           Safety Co-Pilot Backend (FastAPI)             │
│                                                         │
│   Route.X Pipeline:                                     │
│   decode_intent → simulate → policy_check →             │
│   explanation → (approve | reject | suggest_change)     │
│                                                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  │
│  │  CMMC       │  │ genomic-go  │  │ AGI-Framework │  │
│  │  Risk rules │  │ Study/Grant │  │ Explanation   │  │
│  │  Policies   │  │ Linking     │  │ Agent         │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  │
└──────────────────────┬──────────────────────────────────┘
                       │
        ┌────────────┴────────────┐
        ▼                            ▼
  Flow / NEAR                   Receipt Store
  (meta-tx,                     (signed JSON +
   on-chain logic)               on-chain anchor)
```

**Route.X Pipeline:**
```
decode_intent → simulate_outcomes → policy_check (CMMC) →
generate_explanation → sign_receipt → broadcast (or block)
```

---

## Data Schema

```typescript
UserAccount {
  id: string
  org_id: string
  risk_profile: RiskTier  // conservative | moderate | aggressive
  jurisdictions: string[]
  protocols_allowed: string[]
  max_single_tx_usd: number
}

Intent {
  id: string
  user_id: string
  natural_language_text: string
  parsed_action: "swap" | "lend" | "pay_vendor" | "grant_payout" | "yield"
  target_protocol: string
  chain: "flow" | "near" | "evm"
  linked_study_id?: string
  linked_grant_program_id?: string
  raw_calldata?: string
}

Simulation {
  id: string
  intent_id: string
  protocol_state_snapshot_uri: string
  projected_outcomes: {
    treasury_after: number
    pct_volatile: number
    estimated_slippage: number
    liquidity_risk: string
  }
  risk_score: number  // 0.0 - 1.0
  risk_label: "low" | "medium" | "high" | "blocked"
}

SafetyPolicy {
  id: string
  owner_id: string
  condition_expr: string  // e.g. "risk_score > 0.7"
  action: "block" | "warn" | "require_2fa" | "require_human_approval"
  scope: "per_user" | "global" | "per_protocol"
}

Receipt {
  id: string
  tx_hash?: string
  simulation_id: string
  intent_id: string
  safety_policies_applied: string[]
  narrative_explanation: string
  risk_score: number
  outcome: "approved" | "blocked" | "modified"
  signatures: string[]  // agent + user
  timestamp: string
}
```

---

## Repo Structure

```
pl-genesis-genomics-safety-copilot/
├── README.md
├── docs/
│   ├── architecture.md
│   ├── policy-spec.md
│   └── demo-script.md
├── backend/
│   ├── app/
│   │   ├── routes/
│   │   │   ├── intents.py
│   │   │   ├── policies.py
│   │   │   └── receipts.py
│   │   ├── services/
│   │   │   ├── simulation.py
│   │   │   ├── cmmc_policy.py
│   │   │   ├── routex_pipeline.py
│   │   │   ├── genomicgo_linker.py
│   │   │   └── explanation_agent.py
│   │   └── models/
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── src/
│   │   ├── pages/
│   │   │   ├── IntentSubmit.tsx      # Paste tx + link study
│   │   │   ├── RiskDashboard.tsx     # Risk score + policies
│   │   │   └── ReceiptViewer.tsx     # Tamper-evident receipts
│   │   └── components/
│   └── package.json
├── routex/
│   └── workflows/
│       └── finance-safety-pipeline.yml
├── cmmc/
│   └── policies/
│       ├── finance-risk-rules.rego
│       └── jurisdiction-constraints.rego
└── docker-compose.yml
```

---

## API Endpoints (MVP)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/intents` | Submit tx payload + lab/study identifiers |
| `GET` | `/intents/{id}` | Simulation results, risk score, explanation, recommended action |
| `POST` | `/policies` | Add/update a JSON safety rule |
| `GET` | `/policies` | List all active policies |
| `GET` | `/receipts/{id}` | Retrieve signed receipt for a completed intent |
| `GET` | `/receipts?study_id={id}` | All receipts for a given study |

---

## User Stories

1. **Lab Admin** — Initiate a DeFi action (swap, pay vendor) and see clear explanation + risk score before signing
2. **Compliance Officer** — Define policies around max exposures, forbidden protocols, and jurisdiction constraints
3. **Donor / DAO** — Verify tamper-evident receipts that link funds to specific studies and grants
4. **Researcher** — See that a payout from the treasury is correctly attributed to my study cohort

---

## Demo Script

1. Define a policy: *"No more than 20% of treasury in volatile assets"*
2. Paste a transaction representing a risky token swap
3. Show simulation summary: *"Treasury after tx: 45% volatile"*
4. UI labels it **High Risk** — explanation cites the policy + linked genomic-go/Sapient.x program
5. Show the signed Receipt JSON with agent + user signatures
6. Optionally: submit a safe tx, show it passes and generates a green receipt

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Pipeline Orchestration | Route.X (`decode → simulate → policy → explain → receipt`) |
| Policy Engine | CMMC (OPA/Rego) |
| Domain Linking | genomic-go-platform + Sapient.x |
| Explanation Agent | AGI-Framework ("Safety Copilot" eliza bot) |
| On-chain | Flow Cadence + NEAR smart contracts |
| Backend | FastAPI (Python) |
| Frontend | React + TailwindCSS |
| Receipt Storage | IPFS / Storacha (content-addressed, immutable) |

---

## Getting Started

```bash
git clone https://github.com/AGI-Corporation/pl-genesis-genomics-safety-copilot
cd pl-genesis-genomics-safety-copilot
cp .env.example .env
docker-compose up
```

Visit `http://localhost:3000` for the UI and `http://localhost:8000/docs` for the API.

---

## License

MIT — see [LICENSE](LICENSE)

---

*Built for the [PL Genesis: Frontiers of Collaboration Hackathon](https://pl-genesis-frontiers-of-collaboration-hackathon.devspot.app) by AGI Corporation*
