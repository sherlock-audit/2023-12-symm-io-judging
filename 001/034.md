Orbiting Mocha Vulture

medium

# Users can lose small amounts during deposit due to rounding error when converting collateral amount decimals to vault token decimals

## Summary

When users deposit funds to `SolverVault`, the deposit amount is taken from users in collateral decimals, but internally recorded in `SolverVaultToken` decimals (by minting this amount of vault token to the user). If `SolverVaultToken` decimals is significantly less than collateral decimals, user might deposit some funds, but will be minted 0 vault tokens for this due to rounding error, leading to user's loss of funds.

## Vulnerability Detail

`SolverVaults.deposit` transfers `amount` collateral tokens from user, but converts `amount` into `SolverVaultToken` decimals to mint vault tokens to user:
```solidity
    IERC20(collateralTokenAddress).safeTransferFrom(
        msg.sender,
        address(this),
        amount
    );
    uint256 amountInSolverVaultTokenDecimals = solverVaultTokenDecimals >=
        collateralTokenDecimals
        ? amount *
            (10 ** (solverVaultTokenDecimals - collateralTokenDecimals))
        : amount /
            (10 ** (collateralTokenDecimals - solverVaultTokenDecimals));
```

The only requirement for decimals of both collateral and vault token is for each of them to be 18 or less. This means that vault token can have significantly smaller decimals than collateral token. In extreme case, vault token can have `decimals = 0` and collateral token can have `decimals = 18`. In such case, any fractional part of the collateral deposited will be lost by the user. This can add up to significant amounts lost from many deposit transactions.

## Impact

Any time `SolverVaultToken` decimals are smaller than collateral's decimals, some amount can be lost by the user during deposit. In extreme cases the amount lost can be up to 1 USD of value per deposit.

## Code Snippet

`SolverVault.deposit` converts collateral amount's decimals into vault decimals to mint vault tokens:
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L168-L173

## Tool used

Manual Review

## Recommendation

Consider adding a restriction to have `SolverVaultToken` decimals greater or equal to collateral's decimals, or maybe a lower bound for `SolverVaultToken` decimals, to avoid loss of user funds during user deposit due to rounding.