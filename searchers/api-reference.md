# API Reference

The searcher surface is gRPC, defined in [`flowrawtf/mev-protos`](https://github.com/flowrawtf/mev-protos). The layout is the Jito proto layout with Flowra extensions marked inline, so existing client code ports with minimal changes.

Transport is plain gRPC over HTTP/2 (no TLS). `AuthService` listens on port `8005`, `SearcherService` on `8234`; see [endpoints](../validators/endpoints.md) for hosts.

## AuthService (`auth.proto`)

Method | Type | Description
--- | --- | ---
`GenerateAuthChallenge` | unary | Returns a challenge string for a `(role, pubkey)` pair. Searchers use role `SEARCHER`. Fails with `PERMISSION_DENIED` if the key is not registered, not yet approved, or suspended.
`GenerateAuthTokens` | unary | Exchanges the signed challenge (sign the exact string `"{pubkey}-{challenge}"`, Ed25519, 64-byte signature) for tokens.
`RefreshAccessToken` | unary | Renews an access token using the refresh token.

Tokens are RS256 JWTs with explicit expiry timestamps: the access token lives **1 hour**, the refresh token **24 hours**. Attach the access token as `authorization: Bearer <token>` metadata on all other calls; refresh with the longer-lived refresh token rather than re-signing. Tokens remain valid across engine restarts.

## SearcherService (`searcher.proto`)

Method | Type | Description
--- | --- | ---
`SubscribePendingTransactions` | server-streaming | The open orderflow stream. `accounts` narrows it server-side to transactions touching those accounts; empty or `"*"` is the full stream. See [Orderflow Stream](orderflow-stream.md).
`SendBundle` | unary | Submits a signed bundle (up to 5 transactions) into the auction and returns the server-assigned `uuid`. With `simulate_only: true` it returns the full simulation report instead and never enters the auction ([dry run](bundles.md#dry-run)).
`SubscribeBundleResults` | server-streaming | Pushes `BundleResult` transitions for bundles submitted by the authenticated key.
`GetTipAccounts` | unary | Returns the tip accounts to transfer lamports to; the transfer sum is your auction bid.
`GetNextScheduledLeader` | unary | Current slot plus the next connected leader's slot and identity.
`GetConnectedLeaders` | unary | Remaining leader slots this epoch for every validator connected to this engine.

## Bundle results (`bundle.proto`)

`SubscribeBundleResults` streams the states below. A bundle that is still competing in the auction produces nothing until it wins, fails, or is superseded.

State | Meaning
--- | ---
`Accepted` | Won a tick and was forwarded to a leader. Carries the slot and the validator identity.
`Rejected / SimulationFailure` | A transaction in the bundle failed simulation. Carries the failing signature and, in `SimulatedTransaction`, the program error, logs, and compute units.
`Dropped / PartiallyProcessed` | Superseded: a bundle emitted within the last 2 seconds already carried one of this bundle's signatures, so this copy could not land atomically. Carries the conflicting signature.

Whether an `Accepted` bundle then lands in the block is visible on-chain from its signatures; the engine does not stream `Processed` / `Finalized` states.

## Error semantics

Transport-level errors follow standard gRPC status codes, and every non-OK status carries a human-readable reason; log them verbatim.

Code | Meaning | Typical messages
--- | --- | ---
`PERMISSION_DENIED` | The key may not do this | `Searcher key is not registered. Register it at portal.flowra.wtf.` · `Searcher key is pending. Approval is granted by the Flowra admins.` · `Searcher key is suspended (enforcement level 4).` · `PBP violation: <reason>` · `mempool stream withheld at enforcement level 3` · `Token has expired.`
`UNAUTHENTICATED` | Missing or unusable credentials | `Challenge has expired.` · `bundle results require an authenticated searcher`
`RESOURCE_EXHAUSTED` | Over a limit | `bundle rate limit of <n>/s exceeded` · `System overloaded.` (challenge cache full — back off and retry)
`INVALID_ARGUMENT` | Request rejected at validation | `Bundle is required.` · `Bundle has too many transactions (<n>), max is 5` · undeserializable packet · `Signature must be 64 bytes.`
`UNAVAILABLE` | Transient transport failure | Reconnect with exponential backoff

## Limits

Limit | Value
--- | ---
Transactions per bundle | 5
`SendBundle` calls per second | Per key, set at registration (default 20). Dry runs count.
Stream expiry | Each batch carries `expiration_time` = generation time + 30 s
Supersede window | 2 s: a signature already sent to a leader cannot be re-submitted in another bundle
Result subscriptions | One per key is sufficient; results are keyed by bundle uuid

### Retry policy

On `UNAVAILABLE`: exponential backoff starting at 100&nbsp;ms, doubling to a 5&nbsp;s cap, with jitter. Do not tight-loop resubmit a bundle: losing bundles already re-enter subsequent ticks automatically, and a resubmitted copy is superseded by the original. On `RESOURCE_EXHAUSTED` from the rate limit, wait out the second; sustained overrun raises your enforcement level.

## SDKs

No Flowra-specific SDK is required. The protos compile with standard `tonic`, `grpc-js`, and `grpcio` toolchains, and Jito-compatible searcher clients (for example `jito-rs` / `jito-ts` searcher examples) work by pointing them at a Flowra endpoint and using the same auth keypair; only `simulate_only` needs the Flowra proto.
