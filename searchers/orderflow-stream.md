---
label: Orderflow Stream
icon: broadcast
order: 90
description: "Subscribing to the pending-transaction stream: message contents and delivery semantics."
---

# Orderflow Stream

The Block Engine broadcasts pending transactions from participating validators to every subscribed searcher. This is Flowra's open mempool: the same flow the auction runs on, visible to every bidder through one interface.

## Subscribing

Open a server-streaming call on `SearcherService` with your access token:

```text
SubscribePendingTransactions {}
```

The stream delivers pending transaction packets from **all** participating validators; one subscription covers the network. Upstream, the Relayer receives the raw flow on the validators' TPU ports, deduplicates it, and forwards it to the Block Engine, which fans it out to subscribers.

## What's in a stream message

Messages carry packet batches with a timestamp header:

Field | Description
--- | ---
`header.ts` | Timestamp when the batch was generated
`batch.packets[].data` | The pending transaction in wire format
`batch.packets[].meta` | Packet metadata: size, source address, flags, sender stake weight

Parse `data` with any standard Solana transaction decoder to inspect accounts, programs, and instructions.

## Delivery semantics

- **Best-effort, low-latency.** The stream is optimized for freshness over completeness; this is a bidding signal, not a ledger. Design strategies to tolerate brief gaps.
- **No replay.** Messages are not persisted for re-delivery. If you disconnect, you resume from *now*, not from where you left off.
- **Keep the receive path fast.** Do your parsing and strategy work off the receive thread so the stream never backs up.

## Filters

Server-side subscription filters (by account or program) are on the [roadmap](../resources/roadmap.md#phase-4-institutional-expansion-q4-2026-and-beyond). Until then, subscribe to the full stream and filter client-side; the packet metadata and transaction contents give you everything needed for fast local matching.

## Regional strategy

Subscribe to the region nearest your infrastructure ([endpoints](../validators/endpoints.md)). Auctions settle every 50&nbsp;ms per region, so detection and bidding from the same region minimizes the gap between seeing an opportunity and landing a bid inside the same window. Use `GetRegions` to discover available regions and `GetNextScheduledLeader` to time submissions around upcoming Flowra leader slots.

## Fair access by design

Every searcher receives the same stream through the same interface. There are no premium feeds, no early-access tiers, and no private forwards inside the Flowra pipeline. The auction is the only way to convert observation into position.

[!ref Next: build and submit bundles](bundles.md)
