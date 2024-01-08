Daring Ash Canary

medium

# Missing Approval to Zero in depositToSymmio Function

## Summary
The `depositToSymmio` function in the `SolverVault` contract lacks an approval to zero before the subsequent approval for the deposit amount. This omission can lead to unexpected behaviors and potential issues during token transfers.
## Vulnerability Detail
In the `depositToSymmio` function, the code is missing an approval to zero before approving the deposit amount for the symmio contract. The relevant code snippet is as follows:
```solidity
function depositToSymmio(uint256 amount) external onlyRole(DEPOSITOR_ROLE) whenNotPaused {
    uint256 contractBalance = IERC20(collateralTokenAddress).balanceOf(address(this));
    require(contractBalance - lockedBalance >= amount, "SolverVault: Insufficient contract balance");

    // Missing approval to zero before the subsequent approval
    require(IERC20(collateralTokenAddress).approve(address(symmio), amount), "SolverVault: Approve failed");
    symmio.depositFor(solver, amount);
    emit DepositToSymmio(msg.sender, solver, amount);
}
```
The `depositToSymmio` function is designed to allow depositors to contribute collateral directly to an external contract (`symmio`). The missing approval to zero can result in unexpected behavior, as it may not reset the previous approval state before setting a new allowance. This could potentially lead to allowance inconsistencies and impact the security of token transfers.
## Impact
unsafe ERC20 approve that do not handle non-standard erc20 behavior. 1.Some token contracts do not return any value. 2.Some token contracts revert the transaction when the allowance is not zero.
## Code Snippet
[Link](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L194)
## Tool used

Manual Review

## Recommendation
It is recommended to set the allowance to zero before increasing the allowance and use safeApprove/safeIncreaseAllowance.
