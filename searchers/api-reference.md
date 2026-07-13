# API Reference

The searcher surface is gRPC, defined in Flowra's `mev-protos` [!badge variant="warning" text="Public repo TBD"]. The layout is compatible with the proto conventions most Solana MEV tooling already speaks, so existing client code ports with minimal changes.

## AuthService (`auth.proto`)

Method | Type | Description
--- | --- | ---
`GenerateAuthChallenge` | unary | Returns a challenge string for a `(role, pubkey)` pair. Searchers use role `SEARCHER`.
`GenerateAuthTokens` | unary | Exchanges the signed challenge (sign the exact string `"{pubkey}-{challenge}"`, Ed25519, 64-byte signature) for tokens.
`RefreshAccessToken` | unary | Renews an access token using the refresh token.

Tokens are JWTs with explicit expiry timestamps. Attach the access token as gRPC bearer metadata on all other calls; refresh with the longer-lived refresh token rather than re-signing.

## SearcherService (`searcher.proto`)

Method | Type | Description
--- | --- | ---
`SubscribePendingTransactions` | server-streaming | The open orderflow stream: pending transaction packet batches from participating validators. See [Orderflow Stream](orderflow-stream.md).
`SendBundle` | unary | Submits a signed bundle (up to 5 transactions) into the auction. Returns the server-assigned bundle `uuid`.
`SubscribeBundleResults` | server-streaming | Pushes `BundleResult` state transitions for your submitted bundles as they occur.
`GetTipAccounts` | unary | Returns the tip accounts to transfer lamports to; the transfer sum is your auction bid.
`GetNextScheduledLeader` | unary | Current slot plus the next connected leader's slot, identity, and region.
`GetConnectedLeaders` | unary | Leader slots for connected validators in the current epoch (this region).
`GetConnectedLeadersRegioned` | unary | Same, across the requested regions.
`GetRegions` | unary | The region you are connected to and all available regions.

## Bundle results (`bundle.proto`)

A submitted bundle moves through a explicit state machine, streamed via `SubscribeBundleResults`:

State | Meaning
--- | ---
`Accepted` | Won its auction and was forwarded to a validator (includes slot and validator identity). Emitted once per forwarded validator.
`Rejected / StateAuctionBidRejected` | Outbid within its state auction (the contest among bundles touching the same accounts). Includes the auction id and your simulated bid in lamports.
`Rejected / WinningBatchBidRejected` | Won its state auction but did not make the tick's final winning set. Includes auction id and simulated bid.
`Rejected / SimulationFailure` | A transaction in the bundle failed simulation; includes the offending signature.
`Rejected / DroppedBundle` | Dropped before auction, for example when no connected leader is upcoming.
`Rejected / InternalError` | Engine-side error; safe to retry.
`Processed` | Landed on-chain (validator identity, slot, and index within the block).
`Finalized` | Reached finalized commitment.
`Dropped` | Forwarded but never landed: `BlockhashExpired`, `PartiallyProcessed` (another copy of a transaction landed first, invalidating atomicity), or `NotFinalized` (fork).

The two rejection types with bid values are your primary market-feedback signal; see [Bundles](bundles.md#how-much-to-tip).

## Error semantics

Transport-level errors follow standard gRPC status codes:

Code | Meaning | Typical causes
--- | --- | ---
`UNAUTHENTICATED` | Token missing or expired | Refresh your access token
`INVALID_ARGUMENT` | Request rejected at validation | Unsigned packet, more than 5 transactions, malformed bundle
`UNAVAILABLE` | Transient transport failure | Reconnect with exponential backoff

`INVALID_ARGUMENT` responses carry a human-readable reason; log them verbatim.

## Limits

Limit | Value
--- | ---
Transactions per bundle | 5
Concurrent result subscriptions | One per authenticated identity is sufficient; results are keyed by bundle uuid
Per-identity rate limits | [Planned](../resources/roadmap.md#phase-4-institutional-expansion-q4-2026-and-beyond); sized so honest strategies are unaffected

### Retry policy

On `UNAVAILABLE`: exponential backoff starting at 100&nbsp;ms, doubling to a 5&nbsp;s cap, with jitter. Do not tight-loop resubmit a rejected bundle; losing bundles already re-enter subsequent ticks automatically for a few seconds.

## SDKs

Language SDKs (Rust, TypeScript, Python) wrapping auth, streaming, and submission are planned for GA [!badge variant="warning" text="TBD"]. Until then, the protos compile cleanly with standard `tonic`, `grpc-js`, and `grpcio` toolchains.
