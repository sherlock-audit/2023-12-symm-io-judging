Main Stone Hamster

medium

# Phishing Risk in `solverVault::requestWithdraw`

## Summary
`requestWithdraw` allows the user to withdraw to a user-provided address. An attacker could phish the users of the protocol to use the attacker's address for withdrawals unknowingly.

## Vulnerability Detail
Example:
USER A deposits. He is now publically visible by checking `collateralTokenAddress` for the address. Now the attacker can target known users token balance and phish/trick the user to call `requestWithdraw(with attacker address, user amount)`. When the `balancer` uses `acceptWithdrawRequest` it only checks if the calling user has the amount needed. After that the attacker can see if it was approved and call `claimForWithdrawRequest` with the public id of the `withdrawRequests[requestId]` and claim the withdrawal because `receiver=attacker`

## Impact
Leaving such an option in the code can expose the users to unnecessary attack attempts. Successful attack = loss of funds for users

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L201-L234

## Tool used
Manual Review

## Recommendation
I cant see any reason why the `msg.sender` has the option to set the `receiver` . Just remove it and use `msg.sender` as the receiver in `requestWithdraw`. It would solve the problem. This fix only involves adding `address receiver = msg.sender;` and removing one input parameter. Other functions use the `receiver` variable so it only needs the right value to be set here.

```javascript
        // ********** FIX **********
        function requestWithdraw(
            uint256 amount,
            // remove this parameter
            // address receiver,
    )
...
        // add this line before `withdrawRequests.push`...
        address receiver = msg.sender;
        // ********** FIX **********
```