# HYPE Staking & Fee Optimization

Phalanx stakes HYPE tokens to reduce trading fees on HyperLiquid, directly increasing net yield for depositors.

## Fee Tiers

HyperLiquid offers fee discounts based on staked HYPE:

| HYPE Staked | Discount | Perp Taker | Spot Taker |
|-------------|----------|-----------|-----------|
| 0 | 0% | 0.0450% | 0.0700% |
| 10,000 | 20% | 0.0360% | 0.0560% |
| 100,000 | 30% | 0.0315% | 0.0490% |
| 500,000 | 40% | 0.0270% | 0.0420% |

## Strategy

**Launch**: Stake 10,000 HYPE for the 20% discount tier. This provides meaningful fee reduction without excessive capital commitment.

**Ongoing**: Allocate 50% of protocol fee revenue to continuously purchase and stake additional HYPE. This creates a compounding flywheel:

```
More revenue → More HYPE staked → Lower fees → Higher net yield → More TVL → More revenue
```

## Impact on Depositor Yield

At the 10K HYPE tier (20% discount), round-trip execution costs are approximately 0.18% of notional. At the 500K tier (40% discount), this drops to approximately 0.14%.

The fee savings flow directly to depositors through higher net yield, since execution costs reduce the gross funding captured by the vault.

## Additional Benefits

- **Staking yield**: Staked HYPE earns native staking rewards, providing a secondary revenue stream
- **Ecosystem alignment**: Publicly staking HYPE signals commitment to the HyperLiquid ecosystem
- **Partnership leverage**: Being a significant HYPE staker strengthens relationships with HyperLiquid team and ecosystem projects
