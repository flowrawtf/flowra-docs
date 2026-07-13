# Endpoints & Addresses

This page is the canonical reference for Flowra's network endpoints and on-chain addresses. Values are published here as infrastructure goes live.

!!!warning Not yet published
Flowra is finalizing its regional rollout. All values on this page are [!badge variant="warning" text="TBD"] until announced. Registered validators and searchers are notified directly when values go live, and this page is updated at the same time.
!!!

## Mainnet regions

All of a validator's regional settings must point at the **same region**. Choose the region closest to your validator.

Region | Block Engine (`--block-engine-url`) | Relayer (`--relayer-url`)
--- | --- | ---
Amsterdam | `TBD` | `TBD`
Frankfurt | `TBD` | `TBD`
New York | `TBD` | `TBD`
Singapore | `TBD` | `TBD`
Tokyo | `TBD` | `TBD`

The final region list will be announced with mainnet launch; the table reflects planned coverage.

## Service ports

The Block Engine exposes separate gRPC services per audience:

Service | Consumer | Reference port
--- | --- | ---
BlockEngineValidator + AuthService | Validator clients | 8003
SearcherService | Searchers | 8234
AuthService (standalone) | External tooling | 8005
BlockEngineRelayer | Flowra Relayer (internal) | 11228

Reference ports are from the current deployment configuration; production endpoints will publish exact host:port pairs.

## Testnet

Environment | Block Engine | Relayer | Searcher endpoint
--- | --- | --- | ---
Testnet | `TBD` | `TBD` | `TBD`

Testnet access is granted during validator and searcher onboarding: contact [info@flowra.wtf](mailto:info@flowra.wtf).

## On-chain addresses

Purpose | Network | Address
--- | --- | ---
Tip payment program | Mainnet | `TBD`
Tip distribution program | Mainnet | `TBD`
Merkle root upload authority | Mainnet | `TBD`
Tip payment program | Testnet | `TBD`
Tip distribution program | Testnet | `TBD`

Tip accounts for searchers are served dynamically by the `GetTipAccounts` RPC; see the [API reference](../searchers/api-reference.md).

## Verifying connectivity

Once endpoints are published, verify reachability before restarting your validator:

```bash
# TLS handshake / reachability check against your region's Block Engine
grpcurl -v <REGION_BLOCK_ENGINE_HOST>:<PORT> list 2>&1 | head

# Round-trip latency matters for auction competitiveness
ping -c 10 <REGION_BLOCK_ENGINE_HOST>
```

!!!success Staying current
Watch [flowra.wtf/blog](https://flowra.wtf/blog) or the registered-participant channel for endpoint announcements. This page is the source of truth once values are live.
!!!
