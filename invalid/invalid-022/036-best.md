Dazzling Bronze Scorpion

medium

# SolverVaultToken has public burn capabilities

## Summary

The SolverVaultToken contains public accessible burn functions that would allow any holder or allowed third party to burn tokens.

## Vulnerability Detail

The SolverVaultToken contract inherits from OZ ERC20Burnable, which adds the `burn()` and `burnFrom()` public functions to allow burning of tokens.

This is presumably included to allow the SolverVault contract to burn user's tokens when they are redeeming their deposit. However, not only the SolverVault contract could burn tokens, any holder or allowed account can call these functions to burn tokens.

## Impact

Token holders or any third party that has been granted an allowance can burn SolverVaultToken.

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaultToken.sol#L11

## Tool used

Manual Review

## Recommendation

Remove ERC20Burnable from the inheritance chain and, similar to `mint()`, add a permissioned `burn()` function that only accounts with the `BURNER_ROLE` can call. Grant the `BURNER_ROLE` to the SolverVault contract.

```solidity
function burn(address from, uint256 amount) external onlyRole(BURNER_ROLE) {
    _burn(from, amount);
}
```
