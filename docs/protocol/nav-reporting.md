# NAV Reporting

The NAV (Net Asset Value) system bridges off-chain HyperLiquid position data to the on-chain vault contract.

## How It Works

Phalanx uses an **operator-reported NAV** model. The operator is a trusted EOA that calls `updateNav()` directly on the vault contract:

1. The operator bot reads the vault's subaccount equity from the HyperLiquid API
2. This value represents the total USD value of all positions plus margin balance
3. The operator calls `updateNav(totalDeployedAssets)` on the PhalanxVault contract
4. The vault calculates `totalAssets() = USDC buffer + totalDeployedAssets`
5. Share price updates: `sharePrice = totalAssets × 1e18 / totalSupply`

```
totalAssets = asset.balanceOf(vault) + totalDeployedAssets
sharePrice  = totalAssets × 1e18 / totalSupply
```

## Performance Fee Charging

Performance fees are calculated and charged atomically inside `updateNav()`:

1. If new share price exceeds the high water mark:
2. Calculate yield per share = currentPrice - highWaterMark
3. Fee per share = yieldPerShare × 10% (performanceFeeBps / 10000)
4. Mint fee shares to feeRecipient via share dilution
5. Update high water mark to post-fee share price

This ensures fees are only charged on **new** yield above the all-time high. Depositors never pay fees during flat or negative periods.

## Staleness Protection

The vault enforces NAV freshness:

- `maxNavStaleness` is set to 24 hours by default
- If `block.timestamp - lastNavUpdate > maxNavStaleness`, deposits revert with `NavStale()`
- This prevents depositors from entering at a stale (potentially incorrect) exchange rate
- The operator must maintain regular NAV updates to keep the vault operational

## Trust Assumptions

This is the primary trust assumption in Phalanx — identical to the model used by Ethena (sENA/USDe):

- **The operator reports NAV honestly.** The vault cannot independently verify off-chain positions.
- **Mitigation:** NAV values are publicly auditable against HyperLiquid's API. Anyone can verify the operator's subaccount equity matches the reported value.
- **Mitigation:** Large deviations trigger automated alerts and investigation.
- **Mitigation:** Historical NAV data is stored on-chain via `NavUpdated` events.

## Future: Trustless NAV via CoreWriter

HyperEVM's L1Read precompiles enable on-chain verification of HyperCore state. The upgrade path:

**Phase 1 (Current):** Operator-reported NAV via `updateNav()`

**Phase 2 (Planned):** Replace `updateNav()` with precompile reads:
- `POSITION_PRECOMPILE (0x800)` — Read perp positions and unrealized PnL
- `SPOT_BALANCE_PRECOMPILE (0x801)` — Read spot token balances
- `ORACLE_PRICE_PRECOMPILE (0x807)` — Read oracle prices for mark-to-market

This would make NAV calculation fully trustless and verifiable on-chain, removing the operator trust assumption entirely.

See [CoreWriter Integration](corewriter.md) for technical details.
