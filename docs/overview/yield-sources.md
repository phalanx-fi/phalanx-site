# Yield Sources

Phalanx vaults generate yield from a single primary source with supplementary revenue streams contributing to protocol sustainability.

## Primary: Funding Rate Payments

Every hour, HyperLiquid settles funding payments between long and short position holders. The payment formula is:

```
Funding Payment = Position Size × Oracle Price × Funding Rate
```

Phalanx vaults hold short perpetual positions and collect funding when rates are positive — which they are the majority of the time on equity names due to structural long demand.

### Current Funding Rates (as of February 2026)

| Name | 1h Funding Rate | Annualized | Open Interest |
|------|----------------|------------|---------------|
| MU | 0.0552% | ~60.5% | $60M |
| GOOGL | 0.0373% | ~40.9% | $26M |
| TSLA | 0.0300% | ~32.9% | $45M |
| NVDA | 0.0252% | ~27.6% | $59M |
| GOLD | 0.0055% | ~6.0% | $119M |
| SILVER | 0.0050% | ~5.5% | $106M |

*Rates are variable and change with market conditions.*

## Yield Flow

```
Gross funding received
  └─ 10% → Protocol fee (Phalanx Labs)
  └─ 90% → Vault NAV increase → Exchange rate appreciation
```

The exchange rate between ct- tokens and USDC increases with every positive funding settlement. No rebasing, no reward claiming — yield is automatically reflected in the token's redemption value.

### Example

| Time | Exchange Rate | 10,000 ct-GOOGL Worth |
|------|-------------|----------------------|
| Day 0 | 1.000000 | $10,000.00 |
| Day 30 | 1.030500 | $10,305.00 |
| Day 60 | 1.061800 | $10,618.00 |
| Day 90 | 1.092273 | $10,922.73 |

## Negative Funding Periods

When funding rates turn negative, long positions pay short positions. In this scenario:

- The vault **absorbs the cost** — NAV decreases slightly
- **No fee is charged** on negative funding periods
- If funding remains persistently negative (trailing 48 hours), the vault begins automated unwinding to protect depositor capital

Historically, negative funding periods on equity names represent less than 15% of all intervals and are typically short-lived.

## Execution Drag

Entry and exit execution costs are the only friction:

| Component | Cost |
|-----------|------|
| Perp taker fee (10K HYPE staked) | 0.036% |
| Spot taker fee (10K HYPE staked) | 0.056% |
| Round trip (entry + exit) | ~0.18% |

At 30%+ annualized yield, the round-trip execution cost represents less than half a day's yield. There is no ongoing execution cost between deposits and withdrawals because no rebalancing is required.
