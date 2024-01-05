Mean Candy Starling

medium

# Absence of Upgradeable SafeERC20 in Controller.sol

## Summary

The smart contract code in SolverVaults.sol neglects the use of the upgradeable SafeERC20 contract, opting for a standard ERC20 contract (ERC20Upgradeable.sol). This oversight poses potential security risks and hinders the upgradability features provided by the SafeERC20 contract.

## Vulnerability Detail

The absence of the upgradeable SafeERC20 contract in Controller.sol compromises the contract's resilience and upgradeability.

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L10-L11

## Impact

Without the implementation of the upgradeable SafeERC20 contract, the contract may face challenges in ensuring secure and seamless upgrades, potentially leading to vulnerabilities in the handling of ERC20 tokens.

## Code Snippet

## Tool used

Manual Review

## Recommendation

To enhance the security and upgradability of the codebase, it is strongly recommended to replace the use of SafeERC20.sol with the upgradeable SafeERC20 contract in the SolverVaults.sol file. This adjustment ensures that token operations are performed securely, aligning with best practices for upgradable smart contracts.
