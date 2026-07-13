# FAQ

General questions about Flowra. Operator-specific questions: [Validator FAQ](../validators/faq.md).

## About Flowra

### What is Flowra, in one sentence?

**Institutional-grade validator infrastructure for Solana**: validators enforce their own block composition policies ([PBP](../concepts/programmable-block-policy.md)), an open orderflow auction makes that enforcement checkable, and open competition improves validator returns along the way.

### Why open the mempool if the goal is compliance?

Because a policy nobody can check is a promise, not a control. When the orderflow, the auction rules, and the resulting blocks are all observable, a validator's blocks can be checked for consistency with its declared policy. Openness is the accountability mechanism. See [Open Orderflow & Accountability](../concepts/open-orderflow-auction.md).

### What is Flowra built on?

The validator client is built on the **Jito-Solana codebase** (Agave lineage), the practical foundation for bundle-capable block production on Solana today. The Block Engine, the auction, the Programmable Block Policy, and the open orderflow stream are Flowra's own systems, purpose-built for policy-controlled, accountable validation.

### Doesn't opening orderflow make MEV worse?

Flowra's premise is that the MEV market already exists, just closed. When public mempool services were restricted in 2024, extraction did not stop; it moved into private mempools and off-market deals where nobody can measure it. Opening the flow makes fees observable, margins competitive, and ordering checkable, which is precisely what makes protection *enforceable*. See [Orderflow on Solana](../concepts/orderflow-on-solana.md).

### Does Flowra enable sandwich attacks?

Open access is not unconditional access. Validators can set `allow_aggressive_mev = false` in their [block policy](../concepts/programmable-block-policy.md), and the Block Engine then rejects bundle shapes matching the sandwich pattern before they ever enter the auction. Policy is the validator's choice, enforced in infrastructure rather than promised in marketing.

## For users and traders

### What do I get out of Flowra as a regular user?

Two things. **Fair fees**: an open auction produces visible, real-time fee levels, so you stop defensively overpaying into a blind auction. **Enforceable protection**: validators that adopt user-protective policies have them enforced at the infrastructure layer.

### Do I need to change anything to benefit?

No. Users transact normally; the improvements land at the infrastructure layer of validators that adopt Flowra.

## For stakers

### How does Flowra affect my staking yield?

Auction tips (minus the 5% protocol fee and the validator's commission) are distributed to stakers, like MEV commissions today. Open competition is designed to raise the tip pool itself; modeled scenarios anchored to comparable open-auction transitions range from +60% to +261% on the MEV component. Assumptions are in [Auction Mechanics](../concepts/auction-mechanics.md#what-this-means-for-revenue).

### How do I stake toward Flowra validators?

Delegate to a validator running the Flowra client. A public registry of participating validators is planned alongside mainnet [!badge variant="warning" text="TBD"].

## Economics

### What does Flowra charge?

A **5% protocol fee** on auction tips. No subscriptions, no per-seat access fees, no paid data tiers.

### Where does the other 95% go?

To the validator whose block included the bundle, split with its stakers according to the validator's MEV commission.

## Project

### Who is behind Flowra?

Flowra is developed by the Flowra team with advisors from across the ecosystem. Introductions, funding announcements, and team updates are published on the [blog](https://flowra.wtf/blog).

### When is mainnet?

Mainnet launch is targeted for **Q3 2026** with an initial validator cohort; see the [Roadmap](roadmap.md).

### How do I get in touch?

[info@flowra.wtf](mailto:info@flowra.wtf), or watch [flowra.wtf](https://flowra.wtf) and the [blog](https://flowra.wtf/blog) for announcements.
