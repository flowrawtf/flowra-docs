---
label: Bundles
icon: package
order: 80
description: "Bundle structure, tipping, dry runs, auction submission, and what gets a key throttled."
---

# Bundles

A **bundle** is an ordered group of signed transactions submitted as one atomic unit into the auction. Bundles are how searchers convert an observed opportunity into a bid.

## Bundle rules

Property | Rule
--- | ---
Size | Up to **5 transactions** per bundle
Ordering | Executed strictly in the order submitted
Atomicity | **All-or-nothing**: if any transaction fails, the entire bundle is dropped
Tip | At least one transaction transfers lamports to a **tip account** (see below)
Signing | Every transaction fully signed before submission; each transaction's first signature identifies it

### Atomic execution means no revert risk

Before a bundle can enter the auction, the Block Engine simulates it against the leader's bank. If any leg would fail, the bundle is rejected with a `SimulationFailure` result carrying the failing transaction's error and logs, and nothing lands, so you pay nothing. Tight multi-leg strategies (arbitrage, liquidation plus hedge) are safe to attempt aggressively.

## Tipping

Your tip is your **bid** in the 10&nbsp;ms auction:

- Fetch the current tip accounts with `GetTipAccounts`, then include a lamport transfer to one of them inside your bundle.
- Your bid equals the **sum of transfers to tip accounts** across the bundle, as measured in simulation. Misreported tips are caught there and the bundle is dropped.
- Ranking within a tick is by tip, subject to conflict-aware selection and the validator's policy.
- After the **5% protocol fee**, tips flow to the validator and its stakers through the on-chain tip distribution mechanism.

### How much to tip

There is no universal answer; the auction discovers it. Practical guidance:

1. **Start from opportunity value, not from fee floors.** Bid a fraction of your expected profit. As competition on an opportunity type matures, margins compress, and your bid has to reflect what the opportunity is worth.
2. **Remember conflict-awareness.** You are only bidding against bundles that touch *your* accounts. A modest tip can win an uncontested opportunity; a hot pool during volatility is a different auction entirely.
3. **Use the dry run to price.** `simulate_only` shows you exactly what your bundle does against live state, including the tip the engine will measure, before you commit it.

## Dry run

`SendBundle` accepts a Flowra-only flag, `simulate_only`. The bundle is simulated against the leader's bank exactly as a real submission would be, and the full report comes back inline — but it never enters the auction and is never broadcast. Nothing is spent and nothing can land.

```text
SendBundle { bundle: {...}, simulate_only: true }
--> {
  uuid,
  simulated_ok: true | false,
  simulation_err: "<bundle-level failure, JSON>",   // empty when ok
  simulation: [                                   // one per transaction, in order
    { logs, units_consumed, pre/post token balances, pre/post write-locked accounts }
  ],
  simulation_skipped: true | false                // set when simulation could not run at all
}
```

Dry runs count against your key's bundle rate limit like any other `SendBundle`.

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

A bundle that loses a tick is not gone: it re-enters the following ticks and keeps competing until it wins, is superseded, or its blockhash expires, so a near-miss can still land in a later tick without resubmission. Resubmitting the same transactions yourself does the opposite — see *superseded* below.

### Superseded bundles

The engine remembers every transaction signature it has emitted to a leader for the last **2 seconds**. A bundle that repeats one of those signatures cannot land atomically (the earlier copy is already on its way), so it is dropped immediately with `Dropped / PartiallyProcessed` and the conflicting signature named in the result. In practice this is almost always the searcher's own overlapping variant arriving second: build one bundle per opportunity and let it compete, rather than firing variants with shared legs.

## Timing your submissions

Bundles only land while a Flowra validator is the leader. Use `GetNextScheduledLeader` for the next connected leader slot and `GetConnectedLeaders` for every connected validator's remaining slots this epoch, and shape your submission timing around those windows. With no connected leader upcoming, a bundle waits in the auction until one is.

## What gets you throttled

Flowra keeps the stream open by holding keys accountable rather than by gating access. Two mechanisms act on a key:

**The validator's policy (PBP).** Each validator sets its own [block policy](../concepts/programmable-block-policy.md). A bundle that violates it — a blacklisted address or program, a searcher outside a whitelist, or a sandwich pattern where `allow_aggressive_mev` is off — is rejected at submission with `PERMISSION_DENIED: PBP violation: <reason>`. The engine classifies a bundle as a sandwich when it has three or more transactions, the first and last share a fee payer, and a middle transaction has a different one.

**Network enforcement levels.** The control plane raises a key's level when network rules trip, and an operator lowers it:

Level | Effect
--- | ---
L0 | Normal
L1 | Warning; no change in service
L2 | Bundle submission limited
L3 | Mempool stream withheld (subscriptions end with `PERMISSION_DENIED`)
L4 | Suspended; authentication fails

A single detected sandwich sets a key to **L3**. Sustained request-rate abuse raises the level one step per window. Your current level and its reason are shown on the portal.

## Best practices

### Dry-run before you submit

The engine will catch failing bundles for free, but a submission that was never viable wastes your latency window. `simulate_only` against the leader's live state is cheaper than a wasted tick.

### Guard your assumptions in-program

Where possible, encode pre- and post-checks into the transactions themselves (balance assertions, slippage bounds). Atomicity protects you from partial execution; assertions protect you from *successful* execution under moved markets.

### Keep the tip inside the core transaction

Attach the tip transfer within a transaction that is essential to the bundle's success, not as a detached final transfer. This prevents any path where your tip could land while your strategy legs do not.

### One bundle per opportunity

Overlapping variants supersede each other within the 2-second window. Pick your bid and let the bundle re-enter ticks on its own.

### Treat results as market data

`Rejected` and `Dropped` results carry structured reasons, the failing signature, and simulation logs. They are calibration signals, not errors.

[!ref Full API reference](api-reference.md)
