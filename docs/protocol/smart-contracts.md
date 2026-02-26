# Smart Contracts

Phalanx Protocol consists of three Solidity contracts deployed on HyperEVM.

## PhalanxFactory.sol

The factory deploys new vault instances. Each vault corresponds to a single carry trade name (ct-HYPE, ct-NVDA, etc.).

**Key functions:**

| Function | Access | Description |
|----------|--------|-------------|
| `deployVault()` | Admin only | Deploy a new PhalanxVault with asset, name, symbol, operator, feeRecipient, depositCap |
| `vaultBySymbol()` | Public | Look up vault address by symbol (e.g., "ct-HYPE") |
| `getAllVaults()` | Public | Return all deployed vault addresses |
| `totalVaults()` | Public | Count of deployed vaults |

**Design:** One vault per symbol. The factory prevents duplicate symbol deployment via `SymbolExists()` revert. Admin can be transferred.

## PhalanxVault.sol

ERC-4626 compliant vault with hybrid instant/async redemption.

**Standards implemented:**
- ERC-4626 (Tokenized Vault Standard)
- ERC-7540 (Asynchronous Redemption, subset)
- ERC-20 (ct- share token)
- OpenZeppelin ReentrancyGuard, Pausable

**User functions:**

| Function | Description |
|----------|-------------|
| `deposit(assets, receiver)` | Deposit USDC, receive ct- shares at current NAV |
| `mint(shares, receiver)` | Specify shares desired, deposit required USDC |
| `redeem(shares, receiver, owner)` | Instant redeem from USDC buffer |
| `withdraw(assets, receiver, owner)` | Withdraw specific USDC amount from buffer |
| `requestRedeem(shares, receiver, owner)` | Async redemption request (ERC-7540) |
| `cancelRedeemRequest(requestId)` | Cancel pending async request |

**Operator functions:**

| Function | Description |
|----------|-------------|
| `updateNav(totalDeployedAssets)` | Report off-chain subaccount equity, charge performance fee |
| `deployAssets(amount, to)` | Move USDC from vault to HyperLiquid subaccount |
| `fulfillRedeem(requestId)` | Settle an async redemption request |
| `batchFulfillRedeem(requestIds)` | Settle multiple requests in one tx |

**View functions:**

| Function | Description |
|----------|-------------|
| `totalAssets()` | On-chain buffer + off-chain deployed |
| `sharePrice()` | Current price per share (1e18 scaled) |
| `availableBuffer()` | USDC available for instant redemptions |
| `maxInstantRedeem(owner)` | Max shares redeemable instantly |
| `convertToShares(assets)` | Preview deposit: USDC → shares |
| `convertToAssets(shares)` | Preview redeem: shares → USDC |

**Key parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `performanceFeeBps` | 1000 (10%) | Fee on yield above high water mark |
| `MAX_PERFORMANCE_FEE` | 2000 (20%) | Hard cap on fee changes |
| `maxNavStaleness` | 24 hours | Deposits auto-pause if NAV stale |
| `depositCap` | Set at deploy | Maximum total deposits |
| `highWaterMark` | 1e18 | Tracks all-time high share price |

## IERC7540.sol

Interface for asynchronous redemption:

**Events:**
- `RedeemRequest(controller, owner, requestId, sender, shares)` — New async request
- `RedeemFulfilled(requestId, controller, shares, assets)` — Operator settled request
- `RedeemRequestCancelled(requestId, controller, shares)` — Request cancelled

**How async redemption works:**

1. User calls `requestRedeem()` — shares transferred to vault (locked, not burned)
2. Operator unwinds proportional position on HyperLiquid
3. Operator returns USDC to vault, calls `fulfillRedeem(requestId)`
4. Vault burns locked shares, sends USDC to receiver at current NAV

Users or operator can cancel pending requests via `cancelRedeemRequest()`, which returns shares to the owner.

## Dependencies

- Solidity 0.8.20
- OpenZeppelin Contracts v5 (ERC20, SafeERC20, Math, ReentrancyGuard, Pausable)
- Foundry (forge build, forge test, forge deploy)
- HyperEVM (Cancun hardfork, EIP-1559, Chain ID 998)

## Deployment

Contracts are deployed via Foundry:

```bash
# Deploy factory
forge create src/PhalanxFactory.sol:PhalanxFactory \
  --constructor-args <ADMIN_ADDRESS> \
  --rpc-url https://rpc.hyperliquid-testnet.xyz/evm \
  --private-key <KEY>

# Deploy vault through factory
cast send <FACTORY> \
  "deployVault(address,string,string,address,address,uint256)" \
  <USDC> "Phalanx Carry HYPE" "ct-HYPE" <OPERATOR> <FEE_RECIPIENT> <CAP> \
  --rpc-url https://rpc.hyperliquid-testnet.xyz/evm \
  --private-key <KEY>
```

Verification via Sourcify on [PurrSec Explorer](https://testnet.purrsec.com/).
