# Integration Guide

Phalanx vaults implement ERC-4626. Any protocol supporting ERC-4626 can integrate ct- tokens natively for lending, yield tokenization, or DEX trading.

Key view functions: `sharePrice()` (1e18 scaled), `totalAssets()` (USDC 6 decimals), `availableBuffer()` (instant redemption capacity).
