---
label: Endpoints & Addresses
icon: globe
order: 650
description: "Regional Flowra endpoints, service ports, and on-chain program addresses."
---

# Endpoints & Addresses

This page is the canonical reference for Flowra's network endpoints and on-chain addresses. It is updated the moment a value changes; registered validators and searchers are also notified directly.

## Mainnet regions

All of a validator's regional settings must point at the **same region**. Choose the region closest to your validator.

Region | Block Engine (`--block-engine-url`) | Relayer (`--relayer-url`)
--- | --- | ---
Frankfurt | `https://frankfurt.mainnet.blockengine.flowra.wtf` | `https://frankfurt.mainnet.relayer.flowra.wtf`
London | `https://london.mainnet.blockengine.flowra.wtf` | `https://london.mainnet.relayer.flowra.wtf`

!!!
A second Frankfurt-region relayer runs in Lithuania (`https://lithuania.mainnet.relayer.flowra.wtf`) and feeds the Frankfurt Block Engine — validators in the Baltics and Poland may see lower latency pointing `--relayer-url` there while keeping the Frankfurt Block Engine.
!!!

Further regions are added as validator demand clusters; the [roadmap](resources/roadmap.md) lists what is planned.

## Searcher endpoints

Searchers use the same Block Engine URL for `AuthService` and `SearcherService`:

Region | Endpoint
--- | ---
Frankfurt | `https://frankfurt.mainnet.blockengine.flowra.wtf`
London | `https://london.mainnet.blockengine.flowra.wtf`

Keys are registered once at [portal.flowra.wtf](https://portal.flowra.wtf) and work in every region.

## Services

Every endpoint is gRPC over TLS on the standard port `443`, with publicly trusted certificates — use `https://` URLs with no port and no custom root certificate. One hostname serves all of a component's gRPC services; the service name in the request path selects the service, so a client library needs nothing beyond the URL.

Host | gRPC services | Consumer
--- | --- | ---
`*.blockengine.flowra.wtf` | `auth.AuthService`, `block_engine.BlockEngineValidator` | Validator clients
 | `auth.AuthService`, `searcher.SearcherService` | Searchers
 | `block_engine.BlockEngineRelayer` | Flowra Relayer (internal)
`*.relayer.flowra.wtf` | `auth.AuthService`, `relayer.Relayer` | Validator clients

The numbered per-service ports of earlier deployments (8003, 8005, 8234, 11226, 11228) are no longer reachable from outside; they exist only on the hosts' loopback behind the TLS front. Relayer TPU ports are advertised to the validator over the `Relayer` connection and do not need to be configured by hand.

## Testnet

There is currently no separate public testnet: the London and Frankfurt regions above are mainnet. Validators wanting a staging environment before pointing a mainnet node at Flowra, and searchers wanting to integrate without tipping, can request one at [info@flowra.wtf](mailto:info@flowra.wtf).

## On-chain addresses

Flowra runs its own tip programs on mainnet. Set them on the validator with `--tip-payment-program-pubkey` and `--tip-distribution-program-pubkey` ([configuration](validators/configuration.md)).

Purpose | Network | Address
--- | --- | ---
Tip payment program | Mainnet | `2viWvGRQaQuTiLDVWqm6R1EtXNTQXP8pkhs44a8JYXqh`
Tip distribution program | Mainnet | `AAuXLgmQDUHKTDovmKStkG9yciYjoe1YcRmgmYLkPy3R`
Merkle root upload authority | Mainnet | `5LTodMSRDDKBDvvgUtSxo17gk8dv6cdnAcKFYnMnybV7`

### Tip accounts

Searchers pay tips by transferring lamports to any of the tip payment program's eight tip accounts. `GetTipAccounts` returns the current list; the addresses are:

```text
BtAM76RgBUfijsMkpCehxbW6iRqdabxazcaek2jQXvZ5
BgNHgADf8PV5fNhTVEgJnyYeNv3veBHicz5vMqhRVsJV
Bq4jckjxQS1igGvNK8ct6KU6phcs81bMw4F2KqNBYbsD
46ba4KqsD25nAwNK8uoQ5cvaAKf4v46DZZHTmJ8TDKwT
HRbTWnz5P3RHQH8KPL2478mm2URC3CShiTdPSm1smmgg
2AFp51z4GQ41BLpNqYBLWEJta1Daw6vAz4HkFCjNeipo
Gg2VMr5DuuHiaTrbQMaNnY51Ut46HSTCi1U9EriEHqRy
SH98FYfX3XmEKtNCTvwpcX56Jj3ZtCpYaLwFdQq6Y48
```

Pick one at random per bundle to spread write locks, and prefer reading the list from `GetTipAccounts` at startup so a rotation never breaks you. See the [API reference](searchers/api-reference.md).

## Verifying connectivity

Verify reachability before restarting your validator:

```bash
# Block Engine: TLS handshake + a gRPC round-trip (expects a gRPC error, which proves routing)
grpcurl frankfurt.mainnet.blockengine.flowra.wtf:443 block_engine.BlockEngineValidator/GetBlockEngineEndpoints

# Relayer
grpcurl frankfurt.mainnet.relayer.flowra.wtf:443 relayer.Relayer/GetTpuConfigs

# Round-trip latency matters for auction competitiveness
ping -c 10 frankfurt.mainnet.blockengine.flowra.wtf
```

Searchers can do the same with `searcher.SearcherService/GetTipAccounts` (it answers with an auth error until you hold a token, which is enough to prove connectivity). The engines do not expose gRPC reflection, so `list` will not work; pass the proto files with `-proto` if you want to call methods with payloads.

!!!success Staying current
Watch [flowra.wtf/blog](https://flowra.wtf/blog) or the registered-participant channel for endpoint announcements. This page is the source of truth once values are live.
!!!
