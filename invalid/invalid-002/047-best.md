Dazzling Bronze Scorpion

medium

# Implementation contract can be initialized

## Summary

The implementation contract behind the proxy of SolverVault can be freely initialized by anyone.

## Vulnerability Detail

The SolverVault contract is an upgradeable contract that sits behind a proxy. In this pattern, the proxy contract delegatecalls to the implementation contract, and new implementations can be deployed and upgraded to by changing the implementation address in the proxy.

This pattern introduces the need of initializers, special functions that can be used to initialize the state of the contract after it is deployed.

Initializers are meant to be called on the proxies to initialize state, but they can still be called on the implementation contract (that is, a direct call to `initialize()` on the implementation address), allowing anyone to initialize the state of the contract.

## Impact

The implementation contract can be initialized after deployed, allowing anyone to take control over the contract.

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L80-L105

## Tool used

Manual Review

## Recommendation

Add a constructor to disable the initializers in the implementation contract.

```solidity
/// @custom:oz-upgrades-unsafe-allow constructor
constructor() {
    _disableInitializers();
}
```
