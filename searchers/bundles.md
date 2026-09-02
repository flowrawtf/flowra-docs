# Bundles

A **bundle** is an ordered group of signed transactions submitted as one atomic unit into the auction. Bundles are how searchers convert an observed opportunity into a bid.

## Bundle rules

Property | Rule
--- | ---
Size | Up to **5 transactions** per bundle
Ordering | Executed strictly in the order submitted
Atomicity | **All-or-nothing**: if any transaction fails, the entire bundle is dropped
Tip | At least one transaction transfers lamports to a **tip account** (see below); **1,000 lamports minimum** per bundle
Signing | Every transaction fully signed before submission; each transaction's first signature identifies it

### Atomic execution means no revert risk

Before a bundle can enter the auction, the Block Engine simulates it against the leader's bank. If any leg would fail, the bundle is rejected with a `SimulationFailure` result carrying the failing transaction's error and logs, and nothing lands, so you pay nothing. Tight multi-leg strategies (arbitrage, liquidation plus hedge) are safe to attempt aggressively.

## Tipping

Your tip is your **bid** in the auction:

- Fetch the current tip accounts with `GetTipAccounts`, then include a lamport transfer to one of them inside your bundle.
- Your bid equals the **sum of transfers to tip accounts** across the bundle, as measured in simulation. Misreported tips are caught there and the bundle is dropped.
- **Minimum tip: 1,000 lamports** per bundle (the same floor Jito uses). The engine reads the tip from the transactions at submission and rejects a bundle under the floor with `INVALID_ARGUMENT` before it costs a simulation. Priority fees do not count toward the floor. Operators can set a different floor for a key, or exempt it.
- Ranking within a tick is by tip, subject to conflict-aware selection and the validator's policy.
- After the **5% protocol fee**, tips flow to the validator and its stakers through the on-chain tip distribution mechanism.

### How much to tip

There is no universal answer; the auction discovers it. Practical guidance:

1. **Start from opportunity value, not from fee floors.** Bid a fraction of your expected profit. As competition on an opportunity type matures, margins compress, and your bid has to reflect what the opportunity is worth.
2. **Remember conflict-awareness.** You are only bidding against bundles that touch *your* accounts. A modest tip can win an uncontested opportunity; a hot pool during volatility is a different auction entirely.
3. **Read your simulation failures.** A `SimulationFailure` result carries the program error and logs of the leg that failed; most mispriced or stale bundles show up there before they cost you a tick.

## Submitting

Submit through `SearcherService` with your access token:

```text
SendBundle {
  bundle: {
    packets: [<tx1>, <tx2>, <tx3>]   // signed, in execution order
  }
}
--> { uuid: "<server-assigned bundle id>" }
```

`SendBundle` returns as soon as the bundle is accepted for simulation; the simulation and auction run asynchronously. Track outcomes on the results stream:

```text
SubscribeBundleResults {}
--> stream of BundleResult { bundle_id, accepted | rejected | dropped }
```

A bundle that loses a tick is not gone: it re-enters the following ticks and keeps competing until it wins or its blockhash expires, so a near-miss can still land in a later tick without resubmission.

### Overlapping bundles

If several of your bundles share a transaction, the engine forwards them in order but puts every bundle that repeats an already-sent signature **behind** the bundles that do not, and forwards at most **three** per shared signature; further repeats are dropped with `Dropped / PartiallyProcessed`. A repeated transaction executes once, so only one of them can land. Send the variants you mean, not copies.

## Timing your submissions

Bundles only land while a Flowra validator is the leader. Use `GetNextScheduledLeader` for the next connected leader slot and `GetConnectedLeaders` for every connected validator's remaining slots this epoch, and shape your submission timing around those windows. With no connected leader upcoming, a bundle waits in the auction until one is.

## What gets you sanctioned

**The validator's policy.** Each validator sets its own [block policy](../concepts/programmable-block-policy.md) — blocked addresses and programs, searcher allowlists, and its stance on aggressive MEV. A bundle the policy disallows is rejected at submission with `PERMISSION_DENIED: PBP violation: <reason>`.

**Network-level sanctions.** Abusive strategies and bundle spamming (sustained submission far above your key's limit) can get a key restricted or suspended. Restrictions are applied by the Flowra operators and shown on the portal, with the reason.

[!ref Full API reference](api-reference.md)
