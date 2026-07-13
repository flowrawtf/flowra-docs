# Glossary

Atomic execution
: The guarantee that a bundle's transactions land **in full and in order, or not at all**. A bundle with any failing transaction is dropped entirely.

Auction tick
: One 50&nbsp;ms auction round in the Block Engine. Roughly eight fit into each Solana slot. See [Auction Mechanics](../concepts/auction-mechanics.md).

Block Engine
: Flowra's central service. It broadcasts the orderflow stream to searchers, validates and simulates bundles, enforces the validator's policy, runs the 50&nbsp;ms auctions, and forwards winning bundles to the leader. See [Architecture](../concepts/architecture.md).

Bundle
: An ordered group of up to 5 signed transactions submitted as one atomic unit into the auction, carrying a tip. See [Bundles](../searchers/bundles.md).

BundleStage
: The validator client module that executes winning bundles atomically at the front of the block and runs the tip program crank each leader slot.

Conflict-aware selection
: Auction logic that detects overlapping account access between candidate bundles (write-write and read-write conflict; read-read does not) and selects the highest-value combination of non-conflicting bundles within the compute budget.

CU reservation
: The share of a block's compute units the validator reserves for auction bundles (configurable, 15% by default), released back to regular flow when no bundles are pending.

Flowra Relayer
: A TPU proxy that receives transactions on the validator's behalf, deduplicates them, and forwards them to the Block Engine and the validator, with no added packet delay.

Flowra Validator Client
: A validator client built on the Jito-Solana codebase (Agave lineage), extended with Flowra's bundle pipeline, CU reservation flags, and PBP policy push.

Gulf Stream
: Solana's mempool-less transaction forwarding design: transactions are pushed directly to upcoming leaders instead of gossiped publicly. The structural reason Solana's orderflow is dark by default.

Leader / leader slot
: The validator currently assigned to produce blocks, and its ~400&nbsp;ms production window.

MEV (Maximal Extractable Value)
: Value extractable by controlling transaction inclusion and ordering: arbitrage, liquidations, and (in harmful forms) front-running and sandwiching.

OOA (Open Orderflow Auction)
: Flowra's market mechanism: a transparent, permissionless auction over pending-transaction flow. See [Open Orderflow & Accountability](../concepts/open-orderflow-auction.md).

Orderflow
: The stream of pending user transactions (swaps, liquidations, transfers) before finalization. See [Orderflow on Solana](../concepts/orderflow-on-solana.md).

PBP (Programmable Block Policy)
: Flowra's core feature: validator-defined block composition rules (address screening, program filters, MEV posture, priority pinning, compute quotas) owned by the validator as a policy file, pushed to the Block Engine, and enforced before the auction. See [PBP](../concepts/programmable-block-policy.md).

PFOF (Payment for Order Flow)
: Monetizing orderflow by routing it to paying market makers through exclusive channels; the model Flowra's open market replaces.

Priority fee
: The fee users attach to compete for inclusion. On Solana today, effectively a blind auction and a key source of systematic overpayment.

Protocol fee
: Flowra's 5% fee on auction tips; the remainder flows to validators and stakers.

Sandwich attack
: Harmful MEV where an attacker brackets a victim's trade with its own transactions to profit from induced price movement. Flowra's policy engine detects the wrapping pattern and lets validators reject it.

Searcher
: A participant running strategies over the orderflow stream and bidding tip-bearing bundles into the auction. See [Searchers](../searchers/index.md).

SFDP (Solana Foundation Delegation Program)
: The Foundation's stake delegation program; its rules influence which infrastructure validators can run without losing delegation.

Slot
: Solana's ~400&nbsp;ms block production interval.

State auction
: The contest among bundles whose account footprints overlap. Each 50&nbsp;ms tick resolves state auctions and assembles the winning batch from their winners.

Tip
: The searcher's auction bid, paid as lamport transfers to designated tip accounts inside the bundle and measured during simulation. Split 5% protocol / 95% validator and stakers.

Tip account
: One of the addresses returned by `GetTipAccounts`. Transfers to these accounts inside a bundle constitute the bundle's bid, claimed through the on-chain tip distribution mechanism.
