# Delta-Neutral Stability

"Delta" measures a portfolio's sensitivity to price changes in the underlying asset. A delta of +1 means the portfolio gains $1 for every $1 the asset rises. A delta of -1 means it gains $1 for every $1 the asset falls.

A **delta-neutral** portfolio has a delta of 0. It is not exposed to price movement in either direction.

## How Phalanx Achieves Delta Neutrality

Every Phalanx vault holds two matched positions:

| Leg | Position | Delta |
|-----|----------|-------|
| Spot | Long equity token on FELIX | +1 |
| Perp | Short perpetual on HyperLiquid | -1 |
| **Net** | | **0** |

The positions are opened simultaneously in matched unit quantities. If the vault buys 100 NVDA spot tokens, it shorts exactly 100 NVDA perpetual contracts. The unit counts are always equal.

## Price Scenarios

**NVDA rises from $140 to $160 (+14.3%)**

| Leg | P&L |
|-----|-----|
| Spot (long 100 @ $140) | +$2,000 |
| Perp (short 100 @ $140) | -$2,000 |
| **Net** | **$0** |

**NVDA falls from $140 to $100 (-28.6%)**

| Leg | P&L |
|-----|-----|
| Spot (long 100 @ $140) | -$4,000 |
| Perp (short 100 @ $140) | +$4,000 |
| **Net** | **$0** |

In both cases, the vault's USD value is unchanged. The only P&L comes from funding rate payments.

## No Rebalancing Required

A common misconception is that delta-neutral positions "drift" over time and require rebalancing. This is not the case for Phalanx vaults.

The vault holds matched **unit quantities** — 100 spot tokens and 100 perp contracts. Price changes affect both legs equally. Funding payments add USDC to the vault's margin balance but do not change the unit counts.

The only time the vault trades is:

- **New deposit** → Open additional matched position
- **Withdrawal** → Close proportional matched position

There is no ongoing rebalancing, no delta monitoring loop, and no execution cost between deposits and withdrawals.

## Comparison to Ethena

Ethena's USDe uses the same delta-neutral mechanism applied to crypto assets (ETH, BTC). Phalanx applies it to equity and commodity perpetuals on HyperLiquid's HIP-3 markets.

| | Ethena (USDe) | Phalanx (ct- tokens) |
|---|---|---|
| Underlying | ETH, BTC | Equities, commodities |
| Venue | CEXs (Binance, Bybit, etc.) | HyperLiquid |
| Custody | Off-exchange (Copper, Ceffu) | On-chain vault contracts |
| Yield source | Crypto perp funding rates | Equity/commodity perp funding rates |
| Output token | sUSDe (staked USDe) | ct-GOOGL, ct-NVDA, etc. |
