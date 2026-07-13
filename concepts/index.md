# Concepts

This section explains what Flowra is and how the system is designed. If you are evaluating Flowra as an institution, validator, searcher, or integrator, start here before moving to the operational guides.

## In this section

[!ref icon="verified" text="Institutional-Grade Validation"](institutional-validation.md)
[!ref icon="shield-check" text="Programmable Block Policy"](programmable-block-policy.md)
[!ref icon="north-star" text="Open Orderflow & Accountability"](open-orderflow-auction.md)
[!ref icon="law" text="Auction Mechanics"](auction-mechanics.md)
[!ref icon="stack" text="Architecture"](architecture.md)
[!ref icon="git-compare" text="Orderflow on Solana"](orderflow-on-solana.md)
[!ref icon="alert" text="The Problem"](the-problem.md)

## The short version

Solana's Gulf Stream design skips the public mempool: transactions go straight to upcoming leaders. That makes the chain fast, and it makes block production a black box. Nobody outside can see what flow a validator received, what it filtered, or why a bundle landed. For institutional operators with compliance obligations, a black box is disqualifying; for everyone else, it hides fee overpayment and unmeasurable MEV harm.

Flowra's response has two parts that only work together:

- **Programmable Block Policy**: validators declare enforceable rules over block composition (screening, program filters, MEV posture), enforced in the pipeline and re-verified by the validator's own client.
- **Open Orderflow Auction**: pending transactions are streamed openly and bundle competition happens in transparent 50&nbsp;ms auctions, so policy enforcement can be checked rather than taken on faith.

Control without accountability is a promise; accountability without control is a spectator sport. Flowra ships both, and the open competition it requires also raises validator returns.
