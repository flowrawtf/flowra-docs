---
label: Endpoints & Addresses
icon: globe
order: 80
description: "Regional Flowra endpoints, service ports, and on-chain program addresses."
---

# Endpoints & Addresses

This page is the canonical reference for Flowra's network endpoints and on-chain addresses. It is updated the moment a value changes; registered validators and searchers are also notified directly.

## Mainnet regions

All of a validator's regional settings must point at the **same region**. Choose the region closest to your validator.

Region | Block Engine (`--block-engine-url`) | Relayer (`--relayer-url`)
--- | --- | ---
Frankfurt | `http://frankfurt.mainnet.blockengine.flowra.wtf:8003` | `http://frankfurt.mainnet.relayer.flowra.wtf:11226`
London | `http://london.testnet.blockengine.flowra.wtf:8003` | `http://london.testnet.relayer.flowra.wtf:11226`

!!!
The London hostnames carry `testnet` for historical reasons; both regions serve **mainnet**. A second Frankfurt-region relayer runs in Lithuania (`lithuania.mainnet.relayer.flowra.wtf:11226`) and feeds the Frankfurt Block Engine — validators in the Baltics and Poland may see lower latency pointing `--relayer-url` there while keeping the Frankfurt Block Engine.
!!!

Further regions are added as validator demand clusters; the [roadmap](../resources/roadmap.md) lists what is planned.

## Searcher endpoints

Searchers connect to the same Block Engine hosts on the searcher ports:

Region | AuthService | SearcherService
--- | --- | ---
Frankfurt | `http://frankfurt.mainnet.blockengine.flowra.wtf:8005` | `http://frankfurt.mainnet.blockengine.flowra.wtf:8234`
London | `http://london.testnet.blockengine.flowra.wtf:8005` | `http://london.testnet.blockengine.flowra.wtf:8234`

Keys are registered once at [portal.flowra.wtf](https://portal.flowra.wtf) and work in every region.

## Service ports

The Block Engine exposes separate gRPC services per audience. All are plain gRPC over HTTP/2 without TLS: use `http://` URLs.

Service | Consumer | Port
--- | --- | ---
BlockEngineValidator + AuthService | Validator clients | 8003
AuthService (standalone) | Searchers and external tooling | 8005
SearcherService | Searchers | 8234
BlockEngineRelayer | Flowra Relayer (internal) | 11228

The Relayer serves validators on `11226` (its gRPC port; the validator fetches the relayer's TPU sockets from it). Relayer TPU ports are advertised to the validator over that connection and do not need to be configured by hand.

## Testnet

There is currently no separate public testnet: the London and Frankfurt regions above are mainnet. For searchers, the [dry run](../searchers/bundles.md#dry-run) (`SendBundle` with `simulate_only`) simulates a bundle against live mainnet state without spending anything or entering the auction, which covers most integration testing. Validators wanting a staging environment before pointing a mainnet node at Flowra can request one at [info@flowra.wtf](mailto:info@flowra.wtf).

## On-chain addresses

Flowra uses the canonical tip programs that Jito-lineage validators already ship with, so a validator's `--tip-payment-program-pubkey` and `--tip-distribution-program-pubkey` do not change when it joins Flowra.

Purpose | Network | Address
--- | --- | ---
Tip payment program | Mainnet | `T1pyyaTNZsKv2WcRAB8oVnk93mLJw2XzjtVYqCsaHqt`
Tip distribution program | Mainnet | `4R3gSG8BpU4t19KYj8CfnbtRpnT8gtk4dvTHxVRwc2r7`
Merkle root upload authority | Mainnet | [!badge variant="warning" text="TBD"] — published before the first tip distribution epoch

Tip **accounts** for searchers are served dynamically by the `GetTipAccounts` RPC and may rotate; never hard-code them. See the [API reference](../searchers/api-reference.md).

## Verifying connectivity

Verify reachability before restarting your validator. The endpoints are plain HTTP/2, so use `-plaintext`:

```bash
# Block Engine: list services on the validator port
grpcurl -plaintext frankfurt.mainnet.blockengine.flowra.wtf:8003 list

# Relayer
grpcurl -plaintext frankfurt.mainnet.relayer.flowra.wtf:11226 list

# Round-trip latency matters for auction competitiveness
ping -c 10 frankfurt.mainnet.blockengine.flowra.wtf
```

Searchers can do the same against `:8005` and `:8234`.

!!!success Staying current
Watch [flowra.wtf/blog](https://flowra.wtf/blog) or the registered-participant channel for endpoint announcements. This page is the source of truth once values are live.
!!!
