---
label: Open Orderflow & Accountability
icon: north-star
order: 90
description: "Why Flowra opens the orderflow: verifiable policy enforcement first, better markets second."
---

# Open Orderflow & Accountability

The **Open Orderflow Auction (OOA)** is the mechanism that makes Flowra's promises checkable. Pending transactions from participating validators are streamed openly, searchers compete for inclusion in transparent auctions, and outcomes land on-chain where anyone can inspect them.

It is worth being precise about the order of goals. Flowra does not open the orderflow *because openness earns more* (though it does). Flowra opens the orderflow **because a closed pipeline cannot be audited**, and unauditable infrastructure cannot be institutional-grade.

## Accountability first

A validator that says "we screen sanctioned addresses" or "we don't include sandwiches" is making a claim about a process nobody outside can observe. On today's Solana, that claim is structurally unverifiable: there is no public record of what flow the validator saw, what it filtered, or why a given bundle landed.

Openness changes the equation:

Layer | Closed pipeline | Flowra
--- | --- | ---
What flow existed | Unknowable to outsiders | Open, standardized stream, observable by any subscriber
How winners were chosen | Operator's word | Published auction rules: highest tip among non-conflicting, policy-conforming bundles
What landed | On-chain | On-chain
Can the layers be checked against each other? | No | Yes, and that is the point

This is the property that [Programmable Block Policy](programmable-block-policy.md) builds on: enforcement happens inside a market that participants can see. The [roadmap](../resources/roadmap.md#phase-4-institutional-expansion-q4-2026-and-beyond) adds signed auction records and packaged reports to make that checking systematic.

## Design principles

1. **Open streams.** Transaction streams are broadcast in a standardized format to all subscribed searchers. No privileged feeds, no side deals.
2. **Open bidding.** Any searcher can participate. Competition, not access, determines who wins.
3. **Validator sovereignty.** The validator's policy constrains the auction, and the validator's client re-verifies every winning set. Flowra proposes; the validator disposes.
4. **Transparent outcomes.** Published rules in, blocks out, checkable by anyone who cares to look.

## The market effects

Openness engineered for accountability also produces a healthier market:

### Fees users can see

Priority fees on Solana work like a blind auction: users overpay defensively because they cannot observe the going rate. An open auction generates real-time, observable price levels, letting fees settle where the market actually clears.

### Competition instead of privilege

When one searcher has exclusive access to an opportunity, it tips the minimum needed to land. When many compete, tips rise toward the opportunity's value, and the validator and its stakers capture the difference. The mechanics and modeling are in [Auction Mechanics](auction-mechanics.md).

### Permissionless entry for searchers

Access is standardized and open: a keypair and a strategy, not a relationship. See [Searchers](../searchers/index.md).

## Protocol economics

Flowra charges a **5% protocol fee** on auction tips. The remainder flows to the validator and, through its MEV commission, to stakers. There are no subscriptions, no per-seat fees, and no paid data tiers.

## What the OOA is *not*

- **Not a sandwiching venue.** Open flow is not unconditional flow: validator policies reject bundle shapes consistent with sandwiching, and enforcement is dual-layer. See [PBP](programmable-block-policy.md).
- **Not a replacement for the validator.** If Flowra infrastructure is unreachable, the validator builds blocks locally. Participation is additive. See [Architecture](architecture.md).
- **Not a data business.** Every searcher sees the same stream through the same interface.

## Next

[!ref The auction, tick by tick](auction-mechanics.md)
[!ref The institutional case this serves](institutional-validation.md)
