# What is Phalanx

Phalanx is a tokenized carry protocol built on HyperEVM. It operates automated delta-neutral ERC-4626 vaults that capture funding rate yield from equity and commodity perpetual futures on HyperLiquid.

## The Problem

On-chain equity perpetual futures (via HyperLiquid's HIP-3 standard) consistently trade at a premium to spot prices. Leveraged longs pay a recurring funding rate to shorts — typically 20–75% annualized on equity names. This creates a persistent, extractable yield opportunity known as a **carry trade**.

Sophisticated traders and institutions have run carry trades for decades. But the trade has a fundamental limitation: **your capital is trapped**.

Whether you're running a basis trade at a bank, on a CEX, or manually on HyperLiquid, the capital deployed in a carry position sits locked on that platform for the duration of the trade. It earns yield, but it can't do anything else. You can't borrow against it. You can't sell the yield stream separately. You can't use it as collateral. It's dead capital — productive but immobile.

## The Solution

Phalanx solves both problems — execution and capital efficiency — by **tokenizing the carry trade**.

Phalanx vaults accept USDC deposits and execute a matched carry trade:

- **Long leg**: Buy the spot equity token on FELIX
- **Short leg**: Short the matching perpetual future on HyperLiquid

The two positions cancel out all price exposure (delta = 0). What remains is the funding rate — paid hourly by leveraged longs to the vault.

Depositors receive **ct- tokens** (e.g., ct-GOOGL) — ERC-4626 vault shares whose exchange rate appreciates as yield accrues. These tokens are transferable, composable, and redeemable for USDC at any time.

## Why Tokenization Matters

A ct- token is not just a receipt. It's a **liquid, composable representation of a live carry trade**. Because ct- tokens are standard ERC-20s on HyperEVM, you can:

- **Use as collateral** — Borrow against your ct-GOOGL on lending markets (HyperLend, Morpho) while it continues earning yield underneath
- **Split yield on Pendle** — Separate ct-NVDA into a fixed-rate Principal Token (PT) and a variable Yield Token (YT), letting you lock in fixed returns or speculate on funding rate direction
- **Trade on DEXs** — Buy or sell ct- tokens on secondary markets without waiting for vault redemption
- **Stack strategies** — Deposit ct-GOLD as collateral, borrow USDC, deposit into ct-MU for leveraged carry exposure across multiple names

None of this is possible when your carry trade sits in a CEX account or a prime brokerage. Phalanx turns an illiquid trading strategy into a programmable DeFi primitive.

## Why HyperLiquid

HyperLiquid's HIP-3 standard enables perpetual futures on real-world assets (equities, commodities, indices) with deep liquidity. Felix Protocol launched TSLA as the first HIP-3 equity perp, with more names following. HyperLiquid processes over $10B in daily volume as the largest perp DEX by open interest.

Phalanx deploys exclusively on HyperEVM — HyperLiquid's EVM-compatible execution layer — giving ct- tokens native composability with Pendle, lending markets, and DEXs on the same chain. No bridging. No fragmented liquidity. One ecosystem.

The HyperEVM's CoreWriter precompiles allow smart contracts to read positions, balances, and oracle prices directly from HyperCore — enabling trustless NAV verification without external oracles.

## Protocol Design

- **No VCs.** Phalanx is bootstrapped with team capital.
- **ERC-4626 vaults.** Industry-standard tokenized vault interface, compatible with any protocol that supports ERC-4626.
- **ERC-7540 async redemptions.** Large withdrawals handled via request/fulfill pattern — no capital lockups.
- **10% performance fee.** Charged only on positive yield above a high water mark. No management fee. No deposit/withdrawal fee.
- **On-chain NAV.** Operator reports vault equity on-chain. Future upgrade: trustless NAV via CoreWriter L1Read precompiles.
- **PhalanxFactory.** One-click deployment of new vault instances for any carry trade name.
