Lone Mango Hare

medium

# Deployer cumulates every single role at deployment, with the need of an upgrade to attribute roles.

## Summary

As currently designed, SolverVault implementation contract uses a `initialize` function , which sets `msg.sender` to all available roles. Also, there is no way to update or set these roles in this implementation contract.

## Vulnerability Detail

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L92-L97

SolverVault needs to be upgraded for a new implementation in order to set roles on the proxy, and this could be avoided.

## Impact

This impact is limited as it is mainly related to centralisation risks. Nevertheless, the design is not optimised, given the fact that an upgrade of the implementation (and its risks) is required to set roles. This should be avoided.

## Code Snippet

The `initialize` function of the SolverVault implementation contract grants all roles to `msg.sender`:
```solidity
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEPOSITOR_ROLE, msg.sender);
        _grantRole(BALANCER_ROLE, msg.sender);
        _grantRole(SETTER_ROLE, msg.sender);
        _grantRole(PAUSER_ROLE, msg.sender);
        _grantRole(UNPAUSER_ROLE, msg.sender);
```

This means until an upgrade occurs, the protocol is fully centralised, with only one address able to do all admin actions.

## Tool used

Manual Review

## Recommendation

The design should be modified, either : 
- by adding inputs in the `initialize` function, to allow the deployer to set roles to the right people
- by adding a setter function to allow role attribution throug `_grantRole` function, in the implementation contract

In both cases, no upgrade is required to set roles.
