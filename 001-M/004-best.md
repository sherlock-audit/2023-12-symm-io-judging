Cuddly Tawny Stallion

medium

# Using solidity version ^8.0.20 will make the protocol incompatible with several L2 chains

## Summary

Using solidity version ^8.0.20 will make the protocol incompatible with several L2 chains

## Vulnerability Detail

Observe the following code 

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L5

The protocol uses solidity version ^0.8.20 which introduces a new opcode, push0. This is a significant change. The push0 opcode is a gas-optimized way to push the constant zero onto the stack. It can lead to more efficient smart contract execution in terms of gas usage, but as mentioned above, it's not yet supported by several L2 chains e.g. Base

https://docs.base.org/differences/


This will make the protocol incompatible with the chain, preventing it from being deployed 

## Impact

The protocol will not be able to be deployed on several chains

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L5

## Tool used

Manual Review

## Recommendation

To mitigate this issue and enhance the protocol's compatibility with a broader range of L2 chains, it is recommended to revert to an older version of Solidity that does not include the push0 opcode. Versions prior to ^0.8.20 would be more appropriate. This change would ensure greater compatibility with various L2 chains, including those that have not yet updated their opcode sets to align with the latest Ethereum mainnet versions.