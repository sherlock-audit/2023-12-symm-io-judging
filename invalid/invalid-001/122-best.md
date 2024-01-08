Amusing Parchment Cottonmouth

medium

# Array Overflow in acceptWithdrawRequest Function

## Summary
The `requestWithdraw` function in the `SolverVault` contract may be susceptible to precision loss due to rounding during the conversion of decimals. This vulnerability could result in unintended consequences, such as incorrect state changes, when subtracting the converted value from the currentDeposit variable.

## Vulnerability Detail
The `requestWithdraw` function converts the withdrawal amount from SolverVaultTokens to collateral tokens, considering different decimal precisions. However, the conversion logic may introduce precision loss due to rounding, potentially leading to inaccurate values. The subtraction operation `(currentDeposit -= amountInCollateralDecimals;)` may result in unintended consequences, especially when `currentDeposit` is close in value to `amountInCollateralDecimals.`

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

#### POC
An external caller initiates a withdrawal request with amount set to 1000 SolverVaultTokens.
The requestWithdraw function converts this amount to collateral tokens, considering decimal precisions (e.g., collateralTokenDecimals is 18, and solverVaultTokenDecimals is 10).
The conversion results in amountInCollateralDecimals equal to 1000000000000000000000000.
The current value of currentDeposit is 1000000000000000000000001.
The subtraction operation (currentDeposit -= amountInCollateralDecimals;) is performed, causing a precision loss due to rounding.
The resulting value of currentDeposit is 1, which may lead to incorrect state changes and unintended consequences in subsequent operations.

## Impact

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L201C5-L235C1
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L219C3-L219C54

## Tool used

Manual Review

## Recommendation
Consider using alternative approaches, such as fixed-point arithmetic or external libraries for decimal calculations, to mitigate precision loss during conversions. 