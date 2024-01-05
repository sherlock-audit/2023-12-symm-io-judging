Decent Hazel Spider

medium

# SolverVaults.requestWithdraw() does not check if the collateral token is paused

## Summary
The contract is meant to be used with USDT and USDC tokens on BNB, Arbitrum, Polygon, Base, opBNB, zkEVM, Optimism. Some tokens, for example USDC on Aribtrum, can be paused, meaning that their functionality is frozen. At the same time, USDT supports blacklisting accounts. `SolverVaults.requestWithdraw()` is neither checking if the collateral token is paused, nor if the user that makes the request is blacklisted, which burns the vault tokens from the user but does not allow them to get back their collateral tokens.

## Vulnerability Detail
Let's have user A call [`requestWithdraw`](https://github.com/SYMM-IO/solver-vaults/blob/4bdebbecb66e29ac18e5a5c9eda42e4cb44cdd65/contracts/SolverVaults.sol#L201-L234) when at least one of two conditions are true:
 - user A is in the collateral token's blacklist
 - the collateral token's functionality is currently paused
 
Since `requestWithdraw` does not check for these conditions, the user's vault tokens will be successfully burned. 

```solidity
        SolverVaultToken(solverVaultTokenAddress).burnFrom(msg.sender, amount);
```

The request will be pushed in the queue, but it won't be processed successfully, since transferring and approving will not be available in [`acceptWithdrawRequest`](https://github.com/SYMM-IO/solver-vaults/blob/4bdebbecb66e29ac18e5a5c9eda42e4cb44cdd65/contracts/SolverVaults.sol#L236-L280)

```solidity
        IERC20(collateralTokenAddress).safeTransferFrom(
            msg.sender,
            address(this),
            providedAmount
        );
```

## Impact
Users lose their vault tokens, but do not receive any collateral tokens in return. This is a problem, because the vault tokens are used for staking. This means that these users will lose their staking power for as long as the aforementioned conditions are true. In case that users are blacklisted, the fault is theirs, but it is possible for a user to not know that the token is paused and make a request, thus Medium severity is appropriate.

## Code Snippet
```solidity
    function requestWithdraw(
        uint256 amount,
        address receiver
    ) external whenNotPaused {
        require(
            SolverVaultToken(solverVaultTokenAddress).balanceOf(msg.sender) >=
                amount,
            "SolverVault: Insufficient token balance"
        );
        SolverVaultToken(solverVaultTokenAddress).burnFrom(msg.sender, amount);

        uint256 amountInCollateralDecimals = collateralTokenDecimals >=
            solverVaultTokenDecimals
            ? amount *
                (10 ** (collateralTokenDecimals - solverVaultTokenDecimals))
            : amount /
                (10 ** (solverVaultTokenDecimals - collateralTokenDecimals));

        currentDeposit -= amountInCollateralDecimals;

        withdrawRequests.push(
            WithdrawRequest({
                receiver: receiver,
                amount: amountInCollateralDecimals,
                status: RequestStatus.Pending,
                acceptedRatio: 0
            })
        );
        emit WithdrawRequestEvent(
            withdrawRequests.length - 1,
            receiver,
            amountInCollateralDecimals
        );
    }
```

```solidity
    function acceptWithdrawRequest(
        uint256 providedAmount,
        uint256[] memory _acceptedRequestIds,
        uint256 _paybackRatio
    ) external onlyRole(BALANCER_ROLE) whenNotPaused {
        IERC20(collateralTokenAddress).safeTransferFrom(
            msg.sender,
            address(this),
            providedAmount
        );
        require(
            _paybackRatio >= minimumPaybackRatio,
            "SolverVault: Payback ratio is too low"
        );
        uint256 totalRequiredBalance = lockedBalance;

        for (uint256 i = 0; i < _acceptedRequestIds.length; i++) {
            uint256 id = _acceptedRequestIds[i];
            require(
                id < withdrawRequests.length,
                "SolverVault: Invalid request ID"
            );
            require(
                withdrawRequests[id].status == RequestStatus.Pending,
                "SolverVault: Invalid accepted request"
            );
            totalRequiredBalance +=
                (withdrawRequests[id].amount * _paybackRatio) /
                1e18;
            withdrawRequests[id].status = RequestStatus.Ready;
            withdrawRequests[id].acceptedRatio = _paybackRatio;
        }
```
## Tool used

Manual Review

## Recommendation
Change `SolverVaults.requestWithdraw()` to check if the user is blacklisted and if the collateral token is paused. If any of these is true, revert the transaction, not allowing the user to lose their tokens.
