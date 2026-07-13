---
label: Validator FAQ
icon: question
order: 70
description: "Common questions from validator operators evaluating Flowra."
---

# Validator FAQ

### What happens to my blocks if Flowra goes down?

Nothing. The Flowra client always retains full local block production. If the Relayer is unreachable, the validator reclaims its TPU ports and receives transactions directly; if the Block Engine degrades, the validator simply stops receiving bundles and builds blocks with standard in-client scheduling. The worst case is a normal block. See [Architecture](../concepts/architecture.md#failure-behavior-the-validator-never-depends-on-flowra).

### Do I give up control over what goes into my blocks?

The opposite. You are the policy authority: your [Programmable Block Policy](../concepts/programmable-block-policy.md) file lives on your machine, your client pushes it to the Block Engine, and the engine can only auction what the policy allows. Edits to the file take effect within seconds, without a restart. Screening lists, program filters, MEV posture, priority pinning, and compute quotas are all yours to set.

### Is the Flowra client compatible with my current setup?

The client is built on the **Jito-Solana codebase** (Agave lineage), so anything that runs Jito-Solana runs Flowra with the same hardware and operational playbook. Agave operators face the same migration effort as moving to Jito-Solana.

### Can I run Flowra alongside Harmonic, Rakurai, or Raiku?

Systems that bind your block production pipeline (Flowra, Harmonic, Rakurai, and similar block-engine or scheduler stacks) are exclusive at any given moment: each takes over the TPU path and bundle injection, so a validator identity runs one at a time and switching between them is a restart with different flags. Tools that operate outside the block pipeline (for example, sidecar analytics or shred-distribution services) are generally unaffected. Raiku's slot-reservation model is architecturally different, and potential interoperability is something we are happy to discuss for specific setups: [info@flowra.wtf](mailto:info@flowra.wtf).

### How are tips split?

Auction tips arrive as lamport transfers to tip accounts and are distributed through the on-chain tip distribution mechanism: 5% protocol fee, then your MEV commission (`--commission-bps`), then the remainder to your stakers. See [Configuration](configuration.md#commission-and-distribution).

### What revenue uplift should I expect?

It scales with searcher adoption and market conditions. Open competition consistently prices inclusion higher than closed channels, and the modeling anchored to comparable open-auction transitions ranges from +60% to +261% on the MEV component; see [Auction Mechanics](../concepts/auction-mechanics.md) for the assumptions.

### Does running Flowra affect SFDP eligibility?

SFDP treatment of specific clients is determined by the Solana Foundation, not by Flowra. Confirm current policy with the Foundation before switching a delegation-dependent validator. [!badge variant="warning" text="Status TBD"]

### What does participation cost?

No fixed fees. Flowra's revenue is the 5% protocol fee on tips; if the auction earns you nothing, you pay nothing.

### How do I register?

Email [info@flowra.wtf](mailto:info@flowra.wtf) with your validator identity pubkey and region. See [Getting Started](getting-started.md).
