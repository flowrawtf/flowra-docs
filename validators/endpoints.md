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
Frankfurt | `https://frankfurt.mainnet.blockengine.flowra.wtf:8003` | `https://frankfurt.mainnet.relayer.flowra.wtf:11226`
London | `https://london.mainnet.blockengine.flowra.wtf:8003` | `https://london.mainnet.relayer.flowra.wtf:11226`

!!!
A second Frankfurt-region relayer runs in Lithuania (`https://lithuania.mainnet.relayer.flowra.wtf:11226`) and feeds the Frankfurt Block Engine — validators in the Baltics and Poland may see lower latency pointing `--relayer-url` there while keeping the Frankfurt Block Engine.
!!!

Further regions are added as validator demand clusters; the [roadmap](../resources/roadmap.md) lists what is planned.

## Searcher endpoints

Searchers connect to the same Block Engine hosts on the searcher ports:

Region | AuthService | SearcherService
--- | --- | ---
Frankfurt | `https://frankfurt.mainnet.blockengine.flowra.wtf:8005` | `https://frankfurt.mainnet.blockengine.flowra.wtf:8234`
London | `https://london.mainnet.blockengine.flowra.wtf:8005` | `https://london.mainnet.blockengine.flowra.wtf:8234`

Keys are registered once at [portal.flowra.wtf](https://portal.flowra.wtf) and work in every region.

## Service ports

The Block Engine exposes separate gRPC services per audience. All are gRPC over TLS with publicly trusted certificates: use `https://` URLs, no custom root certificate required.

Service | Consumer | Port
--- | --- | ---
BlockEngineValidator + AuthService | Validator clients | 8003
AuthService (standalone) | Searchers and external tooling | 8005
SearcherService | Searchers | 8234
BlockEngineRelayer | Flowra Relayer (internal) | 11228

The Relayer serves validators on `11226` (its gRPC port; the validator fetches the relayer's TPU sockets from it). Relayer TPU ports are advertised to the validator over that connection and do not need to be configured by hand.

## Testnet

There is currently no separate public testnet: the London and Frankfurt regions above are mainnet. Validators wanting a staging environment before pointing a mainnet node at Flowra, and searchers wanting to integrate without tipping, can request one at [info@flowra.wtf](mailto:info@flowra.wtf).

## On-chain addresses

Flowra runs its own tip programs on mainnet. Set them on the validator with `--tip-payment-program-pubkey` and `--tip-distribution-program-pubkey` ([configuration](configuration.md)).

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

Pick one at random per bundle to spread write locks, and prefer reading the list from `GetTipAccounts` at startup so a rotation never breaks you. See the [API reference](../searchers/api-reference.md).

## Verifying connectivity

Verify reachability before restarting your validator:

```bash
# Block Engine: TLS handshake + service list on the validator port
grpcurl frankfurt.mainnet.blockengine.flowra.wtf:8003 list

# Relayer
grpcurl frankfurt.mainnet.relayer.flowra.wtf:11226 list

# Round-trip latency matters for auction competitiveness
ping -c 10 frankfurt.mainnet.blockengine.flowra.wtf
```

Searchers can do the same against `:8005` and `:8234`.

!!!success Staying current
Watch [flowra.wtf/blog](https://flowra.wtf/blog) or the registered-participant channel for endpoint announcements. This page is the source of truth once values are live.
!!!
