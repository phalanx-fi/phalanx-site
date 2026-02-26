# Phalanx Protocol

**The first tokenized carry protocol on HyperEVM.**

Phalanx operates delta-neutral ERC-4626 vaults that capture funding rate yield on equity and commodity perpetuals traded on HyperLiquid. Depositors receive `ct-` tokens (e.g., ct-GOOGL, ct-NVDA) representing their share of vault assets. Yield accrues automatically through an appreciating exchange rate.

## Quick Links

| Resource | Link |
|----------|------|
| App | [app.phalanx.trading](https://app.phalanx.trading) |
| Website | [phalanx.trading](https://phalanx.trading) |
| Twitter | [@phalanx\_fi](https://x.com/phalanx_fi) |
| GitHub | [github.com/phalanx-fi](https://github.com/phalanx-fi) |

## How It Works in 30 Seconds

1. **Deposit USDC** into any ct- vault
2. **Vault opens a matched trade** — long spot equity on FELIX, short the matching perp on HyperLiquid
3. **Funding rates accrue** every hour, increasing the vault's NAV
4. **Withdraw anytime** — burn ct- tokens, receive USDC at the current exchange rate

No price exposure. No leverage. No lock-ups. The vault earns the carry — the structural premium that leveraged longs pay to hold equity exposure on-chain.

## Launch Vaults

| Vault | Underlying | Type | Annualized Yield |
|-------|-----------|------|-----------------|
| ct-HYPE | Hyperliquid | Crypto | Variable |
| ct-NVDA | NVIDIA Corp. | Equity | ~28% |
| ct-TSLA | Tesla Inc. | Equity | ~33% |
| ct-GOOGL | Alphabet Inc. | Equity | ~41% |
| ct-GOLD | Gold Spot | Commodity | ~6% |

*Yields are variable and based on prevailing funding rates. Past performance does not guarantee future results.*

## Smart Contracts

Phalanx vaults are built on industry standards:

- **ERC-4626** — Tokenized vault standard for deposit/withdraw/share accounting
- **ERC-7540** — Asynchronous redemption extension for large withdrawals
- **PhalanxFactory** — Deploys new vault instances, one per carry trade symbol
- **PhalanxVault** — Individual vault contract with hybrid instant/async redemption

All contracts are deployed on HyperEVM (Chain ID: 998 testnet / mainnet) and verified on [PurrSec Explorer](https://testnet.purrsec.com/).
