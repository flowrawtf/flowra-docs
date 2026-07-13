---
label: Configuration
icon: gear
order: 90
description: "Flowra Validator Client flags and the Programmable Block Policy file."
---

# Configuration

The Flowra Validator Client is configured through startup flags on `agave-validator` (the same pattern as any Agave-lineage client) plus a **Programmable Block Policy** file referenced by environment variable.

## Flowra-related flags

Flag | Required | Description
--- | --- | ---
`--block-engine-url` | Yes | Your regional Block Engine endpoint. See [Endpoints](endpoints.md).
`--relayer-url` | Yes | Your regional Relayer endpoint. The validator fetches the relayer's TPU sockets and advertises them in gossip.
`--tip-payment-program-pubkey` | Yes | On-chain tip payment program address [!badge variant="warning" text="Mainnet address TBD"]
`--tip-distribution-program-pubkey` | Yes | On-chain tip distribution program address [!badge variant="warning" text="Mainnet address TBD"]
`--merkle-root-upload-authority` | Yes | Authority for tip distribution merkle root uploads
`--commission-bps` | Yes | Your MEV commission in basis points (0 to 10000), applied to tips before staker distribution
`--bundle-cu-reserve-pct` | No | Share of block compute units reserved for bundles. Default `15`
`--bundle-reserve-release-pct` | No | Share of that reservation released back to regular flow when no bundles are pending. Default `70`
`--disable-block-engine-autoconfig` | Recommended | Use the configured Block Engine URL as-is instead of RTT-probing for the closest region
`--trust-block-engine-packets` | Optional | Skip redundant signature verification for packets arriving from the Block Engine

Both regional endpoints must point at the **same region**.

## The PBP policy file

Set the policy file path through the environment:

```bash
export FLOWRA_PBP_CONFIG=/etc/flowra/flowra-pbp.toml
```

The validator is the policy authority: it pushes this file to the Block Engine on connect and re-pushes automatically whenever the file changes. Full field reference and a complete example: [Programmable Block Policy](../concepts/programmable-block-policy.md).

```toml
# flowra-pbp.toml (minimal example)
[policy]
allow_aggressive_mev = false

[address_blacklist]
addresses = []
```

If `FLOWRA_PBP_CONFIG` is unset, the default policy applies: open access, standard inclusion, no filters.

### Policy semantics

- **Authority-push.** Your file is the source of truth. The engine enforces whatever you last pushed.
- **Hot reload.** Save the file and the client re-pushes it within seconds. No restart required.
- **Logged.** Policy pushes and bundle rejections appear in client and engine logs.

## Commission and distribution

Auction tips are lamport transfers to per-validator tip accounts, claimed through the on-chain tip distribution mechanism. After the 5% protocol fee, your `--commission-bps` determines the validator/staker split.

```text
tip (100%) -> protocol fee (5%)
          \-> validator commission (your bps)
          \-> stakers (remainder)
```

!!!info Tip crank
Each leader slot, the client's BundleStage runs the tip program crank: it initializes the tip payment config once, then sets the tip receiver and block builder for the epoch. If the crank fails (look for `tip_programs_error=1` in logs), bundles cannot execute; verify the tip program addresses match your network's deployment.
!!!

## Operational notes

- **Logs.** Bundle and Block Engine activity is logged by the `BundleStage` and `BlockEngineStage` modules; run with your usual Agave log configuration.
- **Ledger replay.** On restart the validator replays the ledger before going live, and the Block Engine connection is established only after replay completes. Budget for this in maintenance windows.
- **Upgrades.** Client releases track upstream Agave/Jito-Solana versions; registered validators are notified through the operator channel.
