# Bundles

A **bundle** is an ordered group of signed transactions submitted as one atomic unit into the auction. Bundles are how searchers convert an observed opportunity into a bid.

## Bundle rules

Property | Rule
--- | ---
Size | Up to **5 transactions** per bundle
Ordering | Executed strictly in the order submitted
Atomicity | **All-or-nothing**: if any transaction fails, the entire bundle is dropped
Tip | At least one transaction transfers lamports to a **tip account** (see below)
Signing | Every transaction fully signed before submission

### Atomic execution means no revert risk

Before a bundle can enter the auction, the Block Engine simulates it against recent state. If any leg would fail, the bundle is rejected with a `SimulationFailure` result and nothing lands, so you pay nothing. Tight multi-leg strategies (arbitrage, liquidation plus hedge) are safe to attempt aggressively.

## Tipping

Your tip is your **bid** in the 50&nbsp;ms auction:

- Fetch the current tip accounts with `GetTipAccounts`, then include a lamport transfer to one of them inside your bundle.
- Your bid equals the **sum of transfers to tip accounts** across the bundle, as measured in simulation. Misreported tips are caught there and the bundle is dropped.
- Ranking within a tick is by tip, subject to conflict-aware selection and the validator's policy.
- After the **5% protocol fee**, tips flow to the validator and its stakers through the on-chain tip distribution mechanism.

### How much to tip

There is no universal answer; the auction discovers it. Practical guidance:

1. **Start from opportunity value, not from fee floors.** Bid a fraction of your expected profit. As competition on an opportunity type matures, margins compress, and your bid has to reflect what the opportunity is worth.
2. **Read your rejections.** `StateAuctionBidRejected` and `WinningBatchBidRejected` results include your simulated bid, telling you exactly how the market priced you out.
3. **Remember conflict-awareness.** You are only bidding against bundles that touch *your* accounts. A modest tip can win an uncontested opportunity; a hot pool during volatility is a different auction entirely.

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

Track outcomes on the results stream:

```text
SubscribeBundleResults {}
--> stream of BundleResult { bundle_id, accepted | rejected | processed | finalized | dropped }
```

A losing bundle is not gone immediately: it re-enters subsequent 50&nbsp;ms ticks and keeps competing for a few seconds before expiring, so a near-miss can still land in the next window without resubmission.

## Timing your submissions

Bundles only land while a Flowra validator is the leader. Use `GetNextScheduledLeader` for the next connected leader slot and `GetConnectedLeaders` for the epoch's full schedule, and shape your submission timing around those windows. Bundles submitted with no upcoming leader are rejected with a `DroppedBundle` reason.

## Best practices

### Simulate before you submit

The engine will catch failing bundles for free, but a submission that was never viable wastes your latency window. Run `simulateTransaction` against fresh state for each leg first.

### Guard your assumptions in-program

Where possible, encode pre- and post-checks into the transactions themselves (balance assertions, slippage bounds). Atomicity protects you from partial execution; assertions protect you from *successful* execution under moved markets.

### Keep the tip inside the core transaction

Attach the tip transfer within a transaction that is essential to the bundle's success, not as a detached final transfer. This prevents any path where your tip could land while your strategy legs do not.

### Treat rejections as market data

`Rejected` results carry structured reasons and your simulated bid. They are bid-calibration signals, not errors.

[!ref Full API reference](api-reference.md)
