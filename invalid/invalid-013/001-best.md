Unique Punch Corgi

medium

# The collateral token cannot be changed

## Summary 

The `updateCollateral` function [here](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L128-L137) cannot be changed.

## Details

The `updateCollateral` function is meant to change the collateral token of the of the pool and was called when updating the `setSymmioAddress` , but there is a require check that ensures the `decimals` is less than 18 
````
require(collateralTokenDecimals <= 18, "SolverVault: Collateral decimals should be lower than 18");

````


I assume the protocol to work with  USDT and USDC as clarified here 

```

Which ERC20 tokens do you expect will interact with the smart contracts?

USDT, USDC

```

so this means that once the token has been set to USDC which is of 6 decimals, the contract will no longer accept any token thats of 18 decimals like the DAI with 18 decimals  which might be in the future 

## Impact

The contract can never accept any token that's of 18 decimals, this will affect the functionality of the protocol when there is need of more token collateral like `DAI` token which is of 18 decimals 

## Recommendations

Allow token that are with 18 decimal for more flexibility, because of how most of the tokens in the space are with `18` decimals 