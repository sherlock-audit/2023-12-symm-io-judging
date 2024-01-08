Daring Ash Canary

medium

# Lack of Reentrancy Protection in Withdrawal Process

## Summary
The contract lacks proper protection against reentrancy attacks during the withdrawal process, which could potentially lead to unexpected and harmful behavior.
## Vulnerability Detail
In the `claimForWithdrawRequest` function, the contract transfers tokens using the `safeTransfer` function without implementing checks for reentrancy. Reentrancy occurs when an external contract can repeatedly call back into the vulnerable contract during the execution of a function, potentially leading to unexpected consequences.
```solidity
function claimForWithdrawRequest(uint256 requestId) external whenNotPaused {
    // ... (other checks and actions)
    IERC20(collateralTokenAddress).safeTransfer(request.receiver, amount);
    emit WithdrawClaimedEvent(requestId, request.receiver);
}
```
The absence of reentrancy protection in this context may expose the contract to potential reentrancy attacks, where an attacker could deploy a malicious contract that repeatedly calls the `claimForWithdrawRequest` function before it completes, manipulating the contract's state.

## Impact
The lack of reentrancy protection could result in unexpected behavior during token transfers, potentially leading to loss of funds or manipulation of the contract's state.


## Code Snippet
[Link](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L282-L299)
## Tool used

Manual Review

## Recommendation
Consider implementing reentrancy protection by adding a nonReentrant modifier. 
```solidity
bool private reentrancyLock;

modifier nonReentrant() {
    require(!reentrancyLock, "SolverVault: Reentrant call");
    reentrancyLock = true;
    _;
    reentrancyLock = false;
}

function claimForWithdrawRequest(uint256 requestId) external whenNotPaused nonReentrant {
    // ... (other checks and actions)
    IERC20(collateralTokenAddress).safeTransfer(request.receiver, amount);
    emit WithdrawClaimedEvent(requestId, request.receiver);
}
```