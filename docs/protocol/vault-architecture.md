# Vault Architecture

Phalanx vaults are implemented as ERC-4626 tokenized vaults with ERC-7540 asynchronous redemption extensions, deployed on HyperEVM via the PhalanxFactory contract.

## ERC-4626 Standard

ERC-4626 is the Ethereum standard for tokenized yield-bearing vaults. It defines a common interface for deposit, withdrawal, and share accounting. Every DeFi protocol that supports ERC-4626 — lending markets, yield aggregators, routers — can integrate ct- tokens natively.

Key properties:

- **Shares represent proportional ownership.** If you hold 1% of ct-GOOGL supply, you own 1% of the vault's assets.
- **Exchange rate appreciates with yield.** As funding payments increase the vault's total assets, each share becomes redeemable for more USDC.
- **ERC-20 compatible.** ct- tokens can be transferred, traded, and used as collateral like any standard token.

## Contract Architecture

```
┌─────────────────────────────────────────────────┐
│              PhalanxFactory                      │
│  Admin-controlled, deploys one vault per symbol  │
│  Registry: vaultBySymbol["ct-HYPE"] → address   │
├─────────────────────────────────────────────────┤
│              PhalanxVault (ERC-4626)             │
│                                                 │
│  deposit(USDC) → mint ct- shares                │
│  redeem(shares) → instant from USDC buffer      │
│  requestRedeem(shares) → async (ERC-7540)       │
│                                                 │
│  totalAssets() = buffer + totalDeployedAssets    │
│  sharePrice() = totalAssets * 1e18 / supply     │
│                                                 │
├─────────────────────────────────────────────────┤
│          Operator (trusted EOA)                  │
│  updateNav() → reports off-chain equity          │
│  deployAssets() → moves USDC to HL subaccount    │
│  fulfillRedeem() → settles async redemptions     │
├─────────────────────────────────────────────────┤
│          HyperLiquid Subaccount                  │
│  Spot + perp positions, funding accrues          │
│  Managed by operator bot via HL SDK              │
└─────────────────────────────────────────────────┘
```

## One Vault Per Name

Each underlying asset gets its own vault deployed through PhalanxFactory:

| Symbol | Token | Underlying |
|--------|-------|-----------|
| ct-HYPE | Phalanx Carry HYPE | Hyperliquid |
| ct-NVDA | Phalanx Carry NVDA | NVIDIA Corp. |
| ct-TSLA | Phalanx Carry TSLA | Tesla Inc. |
| ct-GOOGL | Phalanx Carry GOOGL | Alphabet Inc. |
| ct-GOLD | Phalanx Carry GOLD | Gold Spot |

This isolates risk — a funding rate spike on one name does not affect other vaults.

## Deposit Flow

1. User approves USDC spend on the vault contract
2. User calls `deposit(amount, receiver)` — vault mints ct- shares at current exchange rate
3. USDC sits in vault buffer until operator deploys it
4. Operator calls `deployAssets(amount, destination)` to move USDC to HyperLiquid subaccount
5. Operator opens matched spot+perp position on HyperLiquid
6. Operator calls `updateNav()` with new subaccount equity — share price updates

## NAV Calculation

```
totalAssets = USDC balance in vault contract + totalDeployedAssets (off-chain)
sharePrice  = totalAssets × 1e18 / totalSupply
```

The operator reports `totalDeployedAssets` via `updateNav()`. This is the primary trust assumption — the same model used by Ethena. NAV staleness is enforced: if `updateNav()` hasn't been called within 24 hours, deposits auto-pause.

## Performance Fee

Fees are charged only on yield above a high water mark:

1. Operator calls `updateNav()` with new deployed assets value
2. Contract calculates new price per share
3. If price exceeds high water mark, 10% of the yield-per-share is charged
4. Fee is implemented as share dilution — new shares minted to `feeRecipient`
5. High water mark updates to post-fee share price

This means depositors never pay fees on flat or negative performance.

## Admin Controls

The vault admin (Phalanx team multisig) can:

- **Pause/unpause** — Freeze deposits and redemptions in emergency
- **Set operator** — Change the execution operator address
- **Set fee parameters** — Adjust performance fee (capped at 20% max)
- **Set deposit cap** — Limit maximum TVL per vault
- **Emergency withdraw** — Only when paused, only for stuck tokens

The vault admin **cannot**:

- Withdraw USDC during normal operation
- Modify the share accounting math
- Override the high water mark
- Change the ERC-4626 interface
