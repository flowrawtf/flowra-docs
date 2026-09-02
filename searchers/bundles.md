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

Your tip is your **bid** in the 10&nbsp;ms auction:

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

The engine remembers every transaction signature it has sent to a leader for the last **2 seconds**. A later bundle that repeats one of those signatures can only land if the earlier bundle fails — a signature executes once — so the engine does not drop it, but it does not let it compete on equal terms either:

- It is forwarded **behind every bundle that carries no repeated signature** in that tick. If the earlier bundle executed, it fails cheaply at the leader; if the earlier bundle died on a leg of its own, it is the one that lands.
- At most **3 bundles per signature** are forwarded within the window, best reward first. Further repeats are dropped with `Dropped / PartiallyProcessed` and the shared signature named in the result.

This is what makes hedged variants work — `[A, C]` and `[A, B, C]` for the same opportunity both go out, and whichever fits lands — while keeping one hot transaction from filling the leader's bundle window with copies. Keep it to a few deliberate variants; a resubmission of the same bundle gains nothing.

## Timing your submissions

Bundles only land while a Flowra validator is the leader. Use `GetNextScheduledLeader` for the next connected leader slot and `GetConnectedLeaders` for every connected validator's remaining slots this epoch, and shape your submission timing around those windows. With no connected leader upcoming, a bundle waits in the auction until one is.

## What gets you sanctioned

Flowra keeps the stream open by holding keys accountable rather than by gating access. Two mechanisms act on a key:

**The validator's policy (PBP).** Each validator sets its own [block policy](../concepts/programmable-block-policy.md). A bundle that violates it — a blacklisted address or program, a searcher outside a whitelist, or a sandwich pattern where `allow_aggressive_mev` is off — is rejected at submission with `PERMISSION_DENIED: PBP violation: <reason>`. The engine classifies a bundle as a sandwich when it has three or more transactions, the first and last share a fee payer, and a middle transaction has a different one.

**Network-level sanctions.** Aggressive MEV strategies such as sandwiching, and bundle spamming (sustained submission far above your key's limit), can get a key restricted — from a lower bundle limit to losing the mempool stream or being suspended. Restrictions are applied by the Flowra operators and shown on the portal, with the reason.

## Best practices

### Simulate before you submit

The engine will catch failing bundles for free, but a submission that was never viable wastes your latency window. Run `simulateTransaction` against fresh state for each leg first.

### Guard your assumptions in-program

Where possible, encode pre- and post-checks into the transactions themselves (balance assertions, slippage bounds). Atomicity protects you from partial execution; assertions protect you from *successful* execution under moved markets.

### Keep the tip inside the core transaction

Attach the tip transfer within a transaction that is essential to the bundle's success, not as a detached final transfer. This prevents any path where your tip could land while your strategy legs do not.

### A few variants, not a spray

Overlapping variants of one opportunity are allowed and go out behind the fresh bundles, up to three per shared signature. Beyond that they are dropped, and none of them can beat a bundle that carries no repeat. Submit the two or three variants you actually mean, with the bids you actually want; a bundle that loses a tick competes again by itself, so there is nothing to resubmit.

### Treat results as market data

`Rejected` and `Dropped` results carry structured reasons, the failing signature, and simulation logs; a rejection at `SendBundle` names the exact number that was short. They are calibration signals, not errors.

[!ref Full API reference](api-reference.md)
