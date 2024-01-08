Unique Punch Corgi

medium

# setDepositLimit() function should be more robust

## Summary

The `setDepositLimit` function [here](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L151-156) should be more  robust.

## Vulnerability Detail
The `setDepositLimit`  serves as the main function of the contract, as it specify the amount of deposit to be accepted in to the contract, the issue here is there is a lack of check to whether a certain values are set.

Lets say the one with `SETTER_ROLE` role decided to set or change the deposit limit to certain values, there is nothing preventing him to mistakenly set it to `zero` (0), and that will break the function (`main`)of the contract.

## Impact

There will be no deposit allow even when the contract is `unPaused` and not intended to be `limited`

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L151-156

## Tool used

Manual Review

## Recommendation

Add a require statement to ensure that its never set to some certain values like 0 or any number that's out of question to be set in as the limit  like :

```solidity
function setDepositLimit(uint256 _depositLimit) public onlyRole(SETTER_ROLE) {
    // Require that the deposit limit is not set to zero
    require(_depositLimit > 0, "SolverVault: Deposit limit must be greater than zero");

    // Set the deposit limit
    depositLimit = _depositLimit;

    // Emit an event to indicate that the deposit limit has been updated
    emit DepositLimitUpdatedEvent(_depositLimit);
}
```
