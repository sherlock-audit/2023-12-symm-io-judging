Powerful Mustard Pelican

medium

# Users may risk losing their small withdrawals.

## Summary
Users may risk losing their small withdrawals.
## Vulnerability Detail
During the initialization, [minimumPaybackRatio](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L90) is set to <= 1e18, and the check for [paybackRatio](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L247) requires >= minimumPaybackRatio
Using  (amount * _paybackRatio) / 1e18 may result in precision loss.
The user's small withdrawal is calculated as 0.


## Impact
The lockedBalance cannot be transferred to Symmio.Due to precision loss, small deposits from users may be calculated as 0 and can be sent to Symmio without being withdrawable.
## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L262-L264
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L295
## Tool used

Manual Review

## Recommendation
