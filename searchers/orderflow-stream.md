---
label: Orderflow Stream
icon: broadcast
order: 90
description: "Subscribing to the pending-transaction stream: filters, scope, message contents, and delivery semantics."
---

# Orderflow Stream

The Block Engine broadcasts pending transactions from participating validators to every subscribed searcher. This is Flowra's open mempool: the same flow the auction runs on, visible to every bidder through one interface.

## Subscribing

Open a server-streaming call on `SearcherService` with your access token:

```text
SubscribePendingTransactions {
  accounts: []                      // full stream
}

SubscribePendingTransactions {
  accounts: ["<pool>", "<market>"]  // only transactions touching these accounts
}
```

The stream delivers pending transaction packets from **all** participating validators in your region; one subscription covers the network. Upstream, the Relayer receives the raw flow on the validators' TPU ports, deduplicates it, and forwards it to the Block Engine, which fans it out to subscribers.

### Server-side account filters

`accounts` is a list of base58 account addresses. When it is non-empty, the engine only forwards a transaction if at least one of its account keys is in your list; batches with no matching transaction are skipped entirely. An empty list, or a list containing `"*"`, subscribes to the full firehose.

Filtering happens before the packet leaves the engine, so a scoped subscription costs you no bandwidth for flow you would discard anyway. Use it: the pool, market, and program accounts your strategy watches make a short list.

Today the Relayer still forwards the full flow to the engine (accounts-of-interest is wildcard); feeding the Relayer's AOI/POI from live subscriptions is on the [roadmap](../resources/roadmap.md#phase-4-institutional-expansion-q4-2026-and-beyond), and at that point concrete filters may become mandatory.

### Mempool scope

Your key's **mempool scope** ([set at registration](getting-started.md#registering-your-key)) decides which slots you see flow for:

- **Full** — every batch, all the time.
- **Leader scope** — batches only while one of the validators in your scope is leading or about to lead (within the next 8 slots). Outside those windows the stream stays open but silent.

The scope is re-checked on every batch, so a change made by an operator takes effect mid-stream without reconnecting. If your key is set to enforcement level 3 or above, or is no longer approved, the stream ends with `PERMISSION_DENIED` and the reason.

## What's in a stream message

Messages carry packet batches with a timestamp header:

Field | Description
--- | ---
`server_side_ts` | When the engine generated the batch
`expiration_time` | 30 seconds after `server_side_ts`; the flow is stale past that
`transactions[].data` | The pending transaction in wire format
`transactions[].meta` | Packet metadata: size, source address, flags, sender stake weight

Parse `data` with any standard Solana transaction decoder to inspect accounts, programs, and instructions.

## Delivery semantics

- **Best-effort, low-latency.** The stream is optimized for freshness over completeness; this is a bidding signal, not a ledger. Design strategies to tolerate brief gaps.
- **No replay.** Messages are not persisted for re-delivery. If you disconnect, you resume from *now*, not from where you left off.
- **Keep the receive path fast.** The engine broadcasts to every subscriber from one channel; a subscriber that cannot keep up is lagged and skips batches. Do parsing and strategy work off the receive thread.

## Regional strategy

Subscribe to the region nearest your infrastructure ([endpoints](../validators/endpoints.md)). Auctions settle every 10&nbsp;ms per region, so detection and bidding from the same region minimizes the gap between seeing an opportunity and landing a bid inside the same tick. Use `GetConnectedLeaders` to see which validators your region serves and `GetNextScheduledLeader` to time submissions around upcoming Flowra leader slots.

## Fair access by design

Every searcher receives the same stream through the same interface. There are no premium feeds, no early-access tiers, and no private forwards inside the Flowra pipeline. The auction is the only way to convert observation into position.

[!ref Next: build and submit bundles](bundles.md)
