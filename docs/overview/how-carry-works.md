# How Carry Works

A carry trade profits from the difference between two related instruments. In traditional finance, the most common example is borrowing in a low-interest-rate currency and investing in a high-interest-rate currency. The "carry" is the spread between the two rates.

## Carry in Perpetual Futures

Perpetual futures contracts have no expiry date. To keep the contract price anchored to the spot price, exchanges use a **funding rate mechanism**. Every funding interval (hourly on HyperLiquid), one side of the trade pays the other:

- **When funding is positive**: Longs pay shorts
- **When funding is negative**: Shorts pay longs

Funding rates are positive most of the time because speculative demand to go long typically exceeds demand to go short. Traders want leveraged upside exposure. This structural imbalance creates a persistent premium — and a persistent yield opportunity for anyone willing to take the other side.

## The Phalanx Carry Trade

Phalanx captures this yield by holding a **delta-neutral position**:

```
Long 100 NVDA spot tokens on FELIX     = +$14,000 exposure
Short 100 NVDA perpetual on HyperLiquid = -$14,000 exposure
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Net exposure                            =  $0 (delta neutral)
Funding received                        =  Hourly yield
```

- If NVDA goes up 10%, the spot position gains $1,400 and the perp position loses $1,400. Net P&L from price: $0.
- If NVDA goes down 10%, the spot position loses $1,400 and the perp position gains $1,400. Net P&L from price: $0.
- In both cases, the vault collects funding payments from leveraged longs.

The vault has **zero directional risk**. It earns yield regardless of which direction the underlying asset moves.

## Why Are Equity Funding Rates So High?

HyperLiquid's HIP-3 equity perps consistently show higher funding rates than crypto perps. This is because:

1. **Demand asymmetry** — On-chain users overwhelmingly want *long* equity exposure (NVDA, GOOGL, TSLA). Very few want to short equities on-chain. This imbalance pushes funding rates higher.
2. **Limited supply of shorts** — Traditional basis traders and market makers haven't fully entered HIP-3 markets yet. The carry opportunity persists because not enough capital is arbitraging it away.
3. **Access premium** — On-chain equity exposure is novel. Users are willing to pay a premium (via funding) for 24/7 permissionless access to tokenized equities.

As more capital enters these markets, funding rates will compress toward equilibrium. But even in mature markets, the carry opportunity doesn't disappear — it shrinks. Ethena's crypto basis trade still yields 8–15% annually even at $6B+ scale.

## Historical Context

Funding rate carry is one of the oldest and most well-understood strategies in derivatives markets:

| Source | Yield | Notes |
|--------|-------|-------|
| Ethena (USDe) | 8–15% APY | Crypto perp basis trade, $6B+ TVL |
| HIP-3 equity perps | 20–75% APY | Early-stage market, less capital competing |
| TradFi basis trades | 3–8% APY | Mature markets, institutional competition |

Phalanx operates in the middle column — a market with structural yield that has not yet been arbitraged to institutional levels.
