# CoreWriter Integration

HyperEVM provides two mechanisms for smart contracts to interact with HyperCore: **L1Read precompiles** for reading state, and **CoreWriter** for writing transactions. Phalanx plans to integrate both to achieve trustless vault operations.

## L1Read Precompiles

Special contracts at fixed addresses allow EVM contracts to query HyperCore data directly:

| Precompile | Address | Data Available |
|-----------|---------|---------------|
| Position | `0x...0800` | Perp positions, sizes, entry prices, unrealized PnL |
| Spot Balance | `0x...0801` | Spot token balances per user |
| Vault Equity | `0x...0802` | Vault equity values |
| Oracle Price | `0x...0807` | Current oracle prices for any asset |

Values are guaranteed to match the latest HyperCore state at the time the EVM block is constructed.

### How This Applies to Phalanx

Currently, the operator reports NAV via `updateNav()`. With precompiles, the vault could calculate NAV trustlessly:

```
NAV = spotBalance(operator, HYPE) × oraclePrice(HYPE)
    + perpPosition(operator, HYPE).unrealizedPnL
    + perpPosition(operator, HYPE).fundingAccrued
    + marginBalance(operator)
```

This removes the trust assumption entirely — no operator honesty required for NAV accuracy.

## CoreWriter

The CoreWriter system contract at `0x3333333333333333333333333333333333333333` enables EVM contracts to submit transactions to HyperCore:

- Place limit orders on HyperCore orderbooks
- Transfer spot assets between HyperEVM and HyperCore
- Manage vault positions programmatically
- Stake HYPE

### Relevant Actions for Phalanx

| Action | Use Case |
|--------|----------|
| Place order | Vault contract executes carry trades directly |
| Spot transfer | Bridge USDC between EVM vault and HyperCore |
| USD class transfer | Move margin between spot and perp accounts |

## Important Caveats

**CoreWriter is not atomic.** Actions submitted via CoreWriter are delayed by a few seconds to prevent MEV/frontrunning. This means:

- You cannot verify the result of a CoreWriter action within the same transaction
- Actions may silently fail — CoreWriter is an event emitter only, it does not revert on HyperCore failure
- Your EVM transaction will succeed even if the HyperCore action fails

**Precompile data is block-start state.** Within a single block, precompiles return state from when the block was constructed, not mid-execution state. CoreWriter actions in the same block won't be reflected in precompile reads until the next block.

**Decimal mismatch.** HyperEVM uses 18 decimals for HYPE, HyperCore uses 8. Careful conversion is required to avoid inflation bugs.

## Integration Roadmap

### Phase 1: Testnet (Current)
- Operator-reported NAV via `updateNav()`
- Operator deploys/returns assets manually
- Validates ERC-4626 mechanics, fee math, async redemption

### Phase 2: Precompile Read Integration
- Replace `updateNav()` with on-chain precompile reads
- Vault reads perp positions, spot balances, oracle prices directly
- Trustless NAV — no operator honesty required for pricing
- Operator still executes trades off-chain

### Phase 3: Full CoreWriter Integration
- Vault contract executes trades directly via CoreWriter
- Deposits trigger automatic carry trade execution from EVM
- Redemptions trigger automatic position unwind
- Fully trustless — no operator key needed for trade execution

## Developer Resources

- [hyper-evm-lib](https://github.com/hyperliquid-dev/hyper-evm-lib) — Official Solidity library with CoreWriterLib, PrecompileLib, and local Foundry test simulator
- [Hyperliquid Docs: Interacting with HyperCore](https://hyperliquid.gitbook.io/hyperliquid-docs/for-developers/hyperevm/interacting-with-hypercore)
- [Ambit Labs: Demystifying Precompiles](https://medium.com/@ambitlabs/demystifying-the-hyperliquid-precompiles-and-corewriter-ef4507eb17ef)
