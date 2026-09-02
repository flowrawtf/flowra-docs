---
label: Searchers
icon: zap
order: 700
expanded: false
description: "Open access to Solana orderflow: a standardized stream, open auctions, atomic bundles."
---

# Searchers

Flowra gives searchers what Solana has never had: **open, standardized access to orderflow**. No privileged relationships, no private deals. Register a key, subscribe to the stream, find your edge, and win by bidding.

## What you get

### An open, standardized stream

Pending transactions from all participating validators, deduplicated and delivered over a single gRPC interface. One integration covers the whole Flowra validator set, and you can narrow it server-side to the accounts you care about. See [Orderflow Stream](orderflow-stream.md).

### Auctions every 10 ms

Auction ticks close every 10&nbsp;ms, roughly forty per Solana slot. Within a tick, the highest tip wins among non-conflicting bundles; you are racing bids, not nanoseconds. Mechanics: [Auction Mechanics](../concepts/auction-mechanics.md).

### Atomic, all-or-nothing bundles

Bundles execute in order and in full, or not at all. Failed bundles never land, so multi-leg strategies carry no partial-execution risk and no wasted fees on reverts. See [Bundles](bundles.md).

### A free dry run

`SendBundle` with `simulate_only` simulates your bundle against the leader's live state and returns the full report (logs, compute units, token balances, pre/post account state) without entering the auction. Nothing is spent and nothing can land, so you can verify a pricing formula as often as you like.

### Strategy confidentiality

The *orderflow* is open; your *bundles* are not. Submissions go point-to-point to the Block Engine and are never exposed to other searchers or a public mempool.

### Simple economics

The protocol fee is **5% of tips**. No subscriptions, no per-seat access fees, no paid data tiers.

## How it fits together

![The searcher loop: subscribe, bid, win or re-bid](../static/diagrams/searcher-flow.svg)

## Requirements

Requirement | Detail
--- | ---
Keypair | An Ed25519 keypair identifying your searcher. It is registered once at [portal.flowra.wtf](https://portal.flowra.wtf) and then used for authentication and bundle signing.
Connectivity | Plain gRPC (HTTP/2, no TLS) to your nearest Flowra region ([endpoints](../validators/endpoints.md))
Tips | SOL for auction tips, paid as lamport transfers to tip accounts (`GetTipAccounts`)
Bundle size | Up to 5 transactions per bundle

## Guides

[!ref icon="rocket" text="Getting started: registration & authentication"](getting-started.md)
[!ref icon="broadcast" text="Subscribing to the orderflow stream"](orderflow-stream.md)
[!ref icon="package" text="Building and submitting bundles"](bundles.md)
[!ref icon="code" text="API reference"](api-reference.md)
