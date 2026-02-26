# Execution System

The Phalanx execution bot manages all trading operations — opening positions on deposits, closing positions on withdrawals, and collecting funding settlements.

## Simultaneous Execution

Both legs of the carry trade execute at the same time. There is no "perp first" or "spot first" sequencing. The bot fires the spot buy and perp sell simultaneously as a matched pair.

This eliminates any window of directional exposure between legs.

## Execution Tiers

Order size relative to visible order book depth determines the execution method:

| Order Size vs Book Depth | Method | Duration | Typical Slippage |
|--------------------------|--------|----------|-----------------|
| < 2% of visible depth | Market order, both legs | Seconds | 1–2 bps |
| 2–10% of depth | TWAP, 5–10 minute window | Minutes | 2–5 bps |
| > 10% of depth | Extended TWAP, 15–30 min, randomized | Minutes | 5–10 bps |

At initial vault sizes ($50K–$250K), most deposits fall into the market order tier against $26M–$119M OI names. TWAP execution becomes relevant at $5M+ individual deposit sizes.

## No Rebalancing

Once a position is opened, the bot does not trade again until a deposit or withdrawal occurs. There is no delta drift, no rebalancing loop, no ongoing execution cost.

The matched unit quantities (e.g., long 100 NVDA spot, short 100 NVDA perp) remain constant regardless of price movement. Funding payments add USDC to the margin balance but do not alter position sizes.

## Circuit Breakers

The bot implements safety checks for abnormal conditions:

- **Leg fill timeout** — If either the spot or perp leg fails to fill within the timeout window, both legs are cancelled. No partial positions.
- **Spread threshold** — If the bid-ask spread exceeds a configurable threshold (indicating thin liquidity), execution is delayed until conditions normalize.
- **Funding rate monitor** — If the trailing 48-hour average funding rate turns negative, the bot begins automated position unwinding to protect vault NAV.

## Bot Security

The execution bot operates with **restricted API keys**:

- ✅ Can open and close positions
- ✅ Can place and cancel orders
- ❌ Cannot withdraw funds from HyperLiquid
- ❌ Cannot transfer assets to external addresses

Vault admin keys (capable of contract upgrades and parameter changes) are stored on a hardware wallet and never exposed to the bot's runtime environment.

## Monitoring

- Real-time Telegram alerts for all trades, funding settlements, and error conditions
- Position reconciliation runs every 5 minutes
- NAV oracle updates published on-chain every 5 minutes
- Manual override capability for emergency situations
