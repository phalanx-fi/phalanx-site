# Redemptions

Phalanx uses a hybrid redemption model: instant redemptions from a USDC buffer, with ERC-7540 asynchronous fallback for large withdrawals.

## How Redemptions Work

### Instant Redemptions (From Buffer)

Each vault maintains a USDC buffer — liquid reserves sitting in the vault contract for immediate withdrawals.

1. User calls `redeem(shares, receiver, owner)` on the vault contract
2. Vault calculates USDC owed at current exchange rate via `convertToAssets(shares)`
3. If the buffer covers the amount, USDC is sent in the same transaction
4. Shares are burned, `Withdraw` event emitted
5. The operator replenishes the buffer by closing proportional positions in the background

Check available instant redemption with `maxInstantRedeem(owner)`.

### Async Redemptions (ERC-7540)

When a redemption exceeds the buffer:

1. User calls `requestRedeem(shares, receiver, owner)`
2. Shares are **transferred to the vault contract** and locked (not burned yet)
3. A unique `requestId` is assigned and `RedeemRequest` event emitted
4. The operator detects the event and unwinds a proportional position on HyperLiquid
5. Operator returns USDC to the vault contract
6. Operator calls `fulfillRedeem(requestId)` — vault burns locked shares, sends USDC to receiver at **current NAV**
7. `RedeemFulfilled` event emitted

Typical async redemption time: **15–30 minutes** depending on position size and market depth.

### Cancelling Requests

Users can cancel pending async requests:

1. Call `cancelRedeemRequest(requestId)`
2. Locked shares are returned to the owner
3. `RedeemRequestCancelled` event emitted

The operator can also cancel requests if needed (e.g., market conditions make unwinding unsafe).

## Proportional Unwinding

Withdrawals are processed proportionally across the vault's positions. If a vault holds $5M in matched GOOGL positions and a user redeems $550K (11%):

- Operator closes 11% of the spot position
- Operator closes 11% of the perp position
- Resulting USDC returns to the vault
- Redeemer receives their share

This ensures no single withdrawal distorts the vault's position structure.

## Batch Fulfillment

The operator can settle multiple pending requests in a single transaction using `batchFulfillRedeem(requestIds[])`, reducing gas costs during high-redemption periods.

## Exchange Rate at Redemption

**Instant redemptions** are priced at the exchange rate at the moment of the transaction.

**Async redemptions** are priced at the exchange rate when the operator calls `fulfillRedeem()` — not when the request was submitted. This means the redeemer gets the most current NAV, which could be higher or lower than when they requested.

## Tracking Requests

- `pendingRedeemRequest(requestId)` — Returns shares still locked (0 if fulfilled/cancelled)
- `claimableRedeemRequest(requestId)` — Returns USDC amount after fulfillment
- `getOwnerRequests(owner)` — Returns all request IDs for an owner

All request data is stored on-chain in the vault contract — no claim tokens, no NFTs.
