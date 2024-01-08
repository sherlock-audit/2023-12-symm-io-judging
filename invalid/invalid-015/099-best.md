Slow Cloth Mongoose

medium

# Actions cannot be adjusted or cancelled

## Summary

Vault contract lacks functions to adjust previous actions, e.g. when initiating or accepting the requests. Possible errors cannot be corrected.

## Vulnerability Detail

1) When users request withdrawals, they lose shares (burned) with no collateral return. The collateral is returned only when the Balancer accepts their requests. However, there is no possibility to cancel the withdrawal request if the Balancer never accepts it. Users have no guarantees that Balancer ever includes their requests. If they can cancel the requests, at least they can exchange their vault tokens on a secondary market.
It would be fair to have a possibility to cancel and get it back at least after some time, e.g. if not accepted in 1 week.

2) Also, it would be nice to have a possibility for Balancer to correct `_paybackRatio` or reset withdrawal requests in case of an accidentally set wrong value.

3) When the Balancer accepts requests, it can send an arbitrary amount (`providedAmount`) to the Vault. If it sends too much of tokens upfront, the excess will be locked until enough withdrawal requests are accumulated or deposited to Symm. The Vault can have a sweep function for ERC20 tokens to transfer out excessive amounts (```IERC20(collateralTokenAddress).balanceOf(address(this) - lockedBalance```).

## Impact

An action cannot be undone and may result in funds lost (e.g. when a user withdrawal request is never accepted), or the wrong value paid (if the Balancer sets incorrect withdrawal parameters). 
I put it as a Medium issue because the contract is Upgradeable and Pausable so technically some of the cases can be fixed if necessary.

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L201-L234

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L236-L280

## Tool used

Manual Review

## Recommendation

Consider adding edit/cancel functions to handle the aforementioned cases.