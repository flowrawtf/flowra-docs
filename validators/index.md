# Validators

Flowra gives validators first-class control over how their blocks are composed, operational independence, and a second source of block revenue, without ever taking block production out of their hands.

## Why run Flowra

### Your blocks, your policy

The [Programmable Block Policy](../concepts/programmable-block-policy.md) puts composition rules in your hands: sanctions and denylist screening, program filters, MEV posture (including a no-sandwich stance), priority pinning, and compute quotas. You own the policy file, your client pushes it to the pipeline, and changes apply within seconds without a restart.

### Independence without risk

The Flowra client is built on the Jito-Solana codebase: operationally familiar, and always capable of building blocks locally. If any Flowra component is unreachable, your validator produces blocks exactly as it would without Flowra. Participation is additive, never a dependency, and leaving is just a restart. See [Architecture](../concepts/architecture.md#failure-behavior-the-validator-never-depends-on-flowra).

### Revenue from open competition

Open auctions compress searcher margins and push tips up relative to closed channels. Every 50&nbsp;ms, searchers bid for your block space; the highest-tip, non-conflicting set wins and lands first in your block, inside a compute budget you control. Modeling anchored to comparable open-auction transitions puts the MEV-component uplift between +60% and +261%; see [Auction Mechanics](../concepts/auction-mechanics.md).

### Built for institutional requirements

Screening enforced before inclusion, a policy trail in your own logs, and a reporting suite on the [roadmap](../resources/roadmap.md#phase-4-institutional-expansion-q4-2026-and-beyond) designed for risk teams and delegators. See [Institutional-Grade Validation](../concepts/institutional-validation.md).

## What's involved

Aspect | Summary
--- | ---
Client | Flowra Validator Client, built on Jito-Solana (Agave lineage) [!badge variant="warning" text="Repo TBD"]
Hardware | Same class as your existing Agave/Jito-Solana setup
Registration | Validator identity registration with the Flowra team before connecting
Policy | Optional PBP file (`FLOWRA_PBP_CONFIG`); permissive defaults if omitted
Commission | You set your own MEV commission in basis points
Fallback | Automatic: local block production continues if Flowra is unreachable

## Guides

[!ref icon="rocket" text="Getting started"](getting-started.md)
[!ref icon="gear" text="Configuration"](configuration.md)
[!ref icon="globe" text="Endpoints & addresses"](endpoints.md)
[!ref icon="question" text="Validator FAQ"](faq.md)

!!!info Early access
Flowra is onboarding an initial validator cohort ahead of full mainnet availability. To join the early partner group, contact [info@flowra.wtf](mailto:info@flowra.wtf).
!!!
