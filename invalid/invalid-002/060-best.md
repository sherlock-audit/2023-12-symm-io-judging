Joyous Purple Squid

medium

# SolverVaults.sol - no storage gap for upgradability

## Summary
Storage of the vault, meant to be upgradable, might face issues due to lack of storage gap.

## Vulnerability Details
There is no storage ``__gap`` variable defined to ensure proper upgradability of the vault contract. The repo README specifies the meant upgradability of the vaults.
If these contract in any scenario, get inherited, the issue would impact the children as well.

## Impact
Corrupted storage

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L66-L79

## Tool used

Manual Review

## Recommendation
Introduce the standard ``uint256 __gap[50]`` variable to reserve new empty storage for  upgrades
