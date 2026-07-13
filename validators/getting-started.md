# Getting Started

This guide takes a working Solana validator to a Flowra-connected validator. The switch is a client swap plus configuration, comparable to moving between Agave and Jito-Solana; budget one maintenance window.

## Prerequisites

- A running mainnet or testnet validator on **Agave or Jito-Solana**. The Flowra client is built on the Jito-Solana codebase, so operators of either will find the setup familiar.
- Hardware meeting standard Agave validator recommendations. Flowra adds no additional hardware requirements.
- Outbound connectivity from your validator to your region's Block Engine and Relayer endpoints (gRPC/TLS). See [Endpoints](endpoints.md).
- Your validator **identity pubkey**, for registration.

## Onboarding steps

>>> Register your validator identity

Send your identity pubkey and preferred region to [info@flowra.wtf](mailto:info@flowra.wtf) (a self-service registration portal is [!badge variant="warning" text="TBD"]). You will receive confirmation along with the current endpoint values for your region.

>>> Install the Flowra Validator Client

Download or build the client [!badge variant="warning" text="Public repo TBD"]:

```bash
git clone <flowra-validator-repo>
cd flowra-validator
cargo build --release --bin agave-validator
```

The result is a drop-in `agave-validator` binary with Flowra's bundle pipeline built in.

>>> Configure the client

Add the Flowra flags to your validator startup (full reference: [Configuration](configuration.md)):

```bash
export FLOWRA_PBP_CONFIG=/etc/flowra/flowra-pbp.toml   # optional policy file

agave-validator \
    # ... your existing flags ...
    --block-engine-url <REGION_BLOCK_ENGINE_URL> \
    --relayer-url <REGION_RELAYER_URL> \
    --tip-payment-program-pubkey <TIP_PAYMENT_PROGRAM> \
    --tip-distribution-program-pubkey <TIP_DISTRIBUTION_PROGRAM> \
    --merkle-root-upload-authority <MERKLE_AUTHORITY> \
    --commission-bps <YOUR_MEV_COMMISSION> \
    --bundle-cu-reserve-pct 15
```

Endpoint and program values are listed in [Endpoints & addresses](endpoints.md). All regional values must point at the **same region**.

>>> Restart and verify

Restart on the new binary during your normal procedure (ledger-safe restart, wait for your leader window to pass). Note that the Block Engine connection is established only after ledger replay completes.

Verify in the logs:

```bash
grep -iE "block_engine|bundle" validator.log | tail -50
# Expect: auth handshake complete, packet and bundle subscriptions active,
#         PBP policy pushed (if configured), tip crank healthy
```

>>> Confirm auction participation

Once your next leader slots pass, confirm bundles are landing and tips are accruing to your tip distribution account. A public dashboard for per-validator auction stats is [!badge variant="warning" text="TBD"].

>>>

## Rollback

Rolling back is the reverse of step 3: restart on your previous binary with your previous flags. There is no on-chain state to unwind; Flowra participation is purely an infrastructure-layer choice.

!!!success No lock-in
Your validator remains a fully standard Solana validator throughout. If the Block Engine or Relayer is unreachable, the client reclaims its TPU and produces blocks exactly as before.
!!!

## Next

[!ref Full flag & PBP reference](configuration.md)
[!ref Regional endpoints](endpoints.md)
