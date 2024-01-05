Slow Cloth Mongoose

medium

# Decimal adjustment misbehave with small values

## Summary

There is no minimum amount requirement when depositing or withdrawing tokens, so small amounts can result in truncation of value when accounting for decimal discrepancies between Vault and Collateral tokens.

## Vulnerability Detail

The requirement for Vault and Collateral tokens is to have no more than 18 decimals:
```solidity
require(
    solverVaultTokenDecimals <= 18,
    "SolverVault: SolverVaultToken decimals should be lower than 18"
);
```
```solidity
require(
    collateralTokenDecimals <= 18,
    "SolverVault: Collateral decimals should be lower than 18"
);
```

When depositing into Vault, it adjusts the decimals:
```solidity
uint256 amountInSolverVaultTokenDecimals = solverVaultTokenDecimals >=
    collateralTokenDecimals
    ? amount *
        (10 ** (solverVaultTokenDecimals - collateralTokenDecimals))
    : amount /
        (10 ** (collateralTokenDecimals - solverVaultTokenDecimals));
```

The collateral token can have a different number of decimals across chains. For example, USDC is 6 decimals on Ethereum, but 18 decimals on BSC.

Let's say `solverVaultTokenDecimals` < `collateralTokenDecimals`, e.g. `solverVaultTokenDecimals` = 2, `collateralTokenDecimals` = 8.
Then, 0.001 collateral tokens = 1e5.
`amountInSolverVaultTokenDecimals = 1e5 / (1e(8 - 2)) = 1e5 / 1e6 = 0`.
This results in a value truncation and the user receives 0 shares in return.


And accordingly when requesting the withdrawal:
```solidity
uint256 amountInCollateralDecimals = collateralTokenDecimals >=
    solverVaultTokenDecimals
    ? amount *
        (10 ** (collateralTokenDecimals - solverVaultTokenDecimals))
    : amount /
        (10 ** (solverVaultTokenDecimals - collateralTokenDecimals));
```

Let's say `solverVaultTokenDecimals` > `collateralTokenDecimals`, e.g. `solverVaultTokenDecimals` = 2, `collateralTokenDecimals` = 8.
Then, 0.001 solver tokens = 1e5.
`amountInCollateralDecimals` = 1e5 / (1e(8 - 2)) = 1e5 / 1e6 = 0.
Again user gets 0 collateral tokens in return.



Introducing a minimum amount requirement for deposits and withdrawals also solves an additional problem. Now anyone can spam deposits or withdrawal requests even with 0 tokens, this spams the system storage and might confuse the receiver (anyone can be the receiver of the request of 0 tokens). The Balancer needs to do extra work when indexing and filtering legitimate withdrawal requests.

## Impact

The issue is not practically significant when the collateral is stablecoins (USDC, USDT) because a fraction of the value is not considered a big loss. However, technically, the protocol should be robust enough to handle such situations and prevent value leakage in all cases.

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L132-L135

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L145-L148

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L168-L173

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L212-L217

## Tool used

Manual Review

## Recommendation

Consider either introducing a minimum amount when depositing or withdrawing. Or minting Vault tokens not in a 1:1 ratio but with some added precision, e.g. scaling by WAD.
