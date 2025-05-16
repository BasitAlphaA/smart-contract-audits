# Uniswap V2 - Smart Contract Audit

- **Auditor**: BasitAlphaA 
- **Date**: May 2024  
- **Contracts Reviewed**: UniswapV2Pair.sol, UniswapV2Factory.sol
- **Commit**: [placeholder for commit hash]  

---

## Overview

This audit reviews the core Uniswap V2 smart contracts, focusing on security vulnerabilities, logical flaws, and gas optimizations. These contracts are widely used for decentralized token swaps on Ethereum.

---

## Findings Summary

| ID | Severity | Title                           |
|----|----------|---------------------------------|
| 1  | High     | Price Manipulation via Sync     |
| 2  | Medium   | Slippage Mismatch in Swap       |
| 3  | Low      | Redundant Event Emission        |

---

## 1. Price Manipulation via `sync()`

**Severity**: High  
**Category**: Oracle Manipulation / Front-running

### Description

The `sync()` function in `UniswapV2Pair` allows anyone to update the reserves to match the current token balances. An attacker can flash-loan tokens, call `sync()` to inflate the price, and then abuse price oracles that read from reserves.

### Proof of Concept
```solidity
// Attacker inflates token0 reserve
pair.token0().transfer(address(pair), largeAmount);
pair.sync();
// Oracle reads inflated reserve
```

### Recommendation
Avoid using `sync()` in on-chain oracles. Use time-weighted average price (TWAP) mechanisms instead.

---

## 2. Slippage Mismatch in Swap Logic

**Severity**: Medium  
**Category**: Inaccurate Output Amounts

### Description

In certain edge cases, the formula in `getAmountOut()` may result in actual slippage that exceeds user expectations due to rounding errors and insufficient liquidity.

### Recommendation
Add checks for minimum output (`amountOutMin`) and simulate slippage impact in frontend tools to mitigate risks.

---

## 3. Redundant Event Emission in `_update()`

**Severity**: Low  
**Category**: Gas Optimization

### Description

In `UniswapV2Pair.sol`, the `_update()` function emits multiple events (`Sync`, `Transfer`, etc.) even when token balances have not changed significantly.

### Recommendation
Add conditionals to emit events only when changes cross meaningful thresholds.

---

## Final Notes

Uniswap V2 is generally well-written and battle-tested, but like all smart contracts, it can benefit from security hardening and optimization. Developers integrating Uniswap V2 logic should be cautious of oracle and slippage issues.

---

📬 Questions or want your dApp audited? DM @jani_audits on X
