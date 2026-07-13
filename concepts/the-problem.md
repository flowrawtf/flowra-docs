# The Problem

Solana's block production works, but it is not *accountable*, it is not *policy-controllable*, and it depends heavily on a single infrastructure stack. For institutional operators those are disqualifying; for everyone else they are a hidden tax. This page lays out the problems Flowra is built to address.

## Problem 1: Block production is unaccountable

There is no public record of the orderflow a validator saw, what it filtered, or why a bundle landed. The consequences compound from there:

- **Ordering manipulation is unmeasurable.** MEV searchers operate through private channels, so front-running and induced slippage cannot be quantified from outside. On Ethereum, researchers can at least observe the mempool. On Solana the opacity is structural; see [Orderflow on Solana](orderflow-on-solana.md).
- **Policy claims are unverifiable.** A validator asserting "we screen sanctioned addresses" or "we exclude sandwiches" offers a promise no third party can check.
- **Restrictions relocated the market instead of shrinking it.** When public mempool services on Solana were shut down in 2024 over sandwich-attack concerns, private mempools re-emerged among select validators, off-market orderflow deals spread, and dozens of validators lost Foundation delegation over exploitative flows. The activity survived; the visibility did not.

You cannot police what you cannot see, and you cannot *comply* with what you cannot control.

## Problem 2: Validators have no policy control

Today's MEV pipelines hand validators a take-it-or-leave-it role: run the client, accept the bundles, collect the tips. There is no mechanism to say *these addresses never enter my blocks* or *this bundle shape is unacceptable to my compliance team*, which is exactly what regulated staking providers, custodians, and exchanges operate under. Block composition control lives with the pipeline, not with the operator who carries the legal exposure.

## Problem 3: The infrastructure is concentrated

Metric | Rough scale
--- | ---
Solana stake running a single MEV client stack | ~90% of the network
Validator tips paid through that stack in 2024 | Over $600M
Peak month (November 2024) | Over $200M
Typical MEV contribution to staking yield | Fractions of a percent in quiet months, several times that in volatile ones

*Figures are rounded estimates from public dashboards and reporting; they move with market conditions.*

Concentration this deep creates two risks:

- **Operational fragility.** One stack's outage, policy change, or degradation touches essentially the whole network's block economics at once.
- **No negotiating power.** When terms or services change, validators have nowhere practical to go. Terms are taken, not negotiated.

A healthy market needs credible alternatives. Flowra is built to be one.

## Problem 4: Users pay a blindness tax

With no visibility into the going rate for inclusion, users defensively over-attach priority fees even when blocks are far from full. The overpayment flows to validators and operators not as payment for scarce space but as a tax on missing information. It is the retail-facing symptom of the same root cause: a dark pipeline.

## Summary

Problem | Flowra's response
--- | ---
Unaccountable production, unverifiable policy claims | An open stream and transparent auction, checkable end to end
No validator policy control | [Programmable Block Policy](programmable-block-policy.md), enforced twice
Single-stack dependence | An independent pipeline with guaranteed local fallback
Blind-auction fees | Observable, real-time fee levels from an open market

[!ref What institutional-grade actually requires](institutional-validation.md)
