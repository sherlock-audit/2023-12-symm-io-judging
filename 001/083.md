Prehistoric Burlap Falcon

high

# Users will be Disincentized from depositing into the vault thereby bricking the Symmio Hedge Funding.

## Summary
There is a known risk when depositing into the Solver vault, where users who deposit funds into the contract will never be able to withdraw the full amount of funds they deposited into the vault, this will lead users never depositing into the vault thereby breaking the functionality of the protocol as it disincentivizes users from funding, and will break the 1:1 peg for minted `SVT` per collateral `USDc`/`USDt`.

**Functionality**
- `This contract enables users to provide liquidity for a Hedger (solver) on the Symmio platform.`
*  [`README.md`](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/README.md)
## Vulnerability Detail
**THE RISK**
Whenever a user requests a withdrawal via `requestWithdraw()`, the contract burns the equivalent amount of solver vault tokens the user has for the amount of collateral token the user requests to withdraw.

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L158C1-L181C6
```solidity
    function requestWithdraw(
..SNIP..
        SolverVaultToken(solverVaultTokenAddress).burnFrom(msg.sender, amount);
..SNIP..
```

and creates a withdrawal request for the user, which is to be accepted by the `Balancer Role` on the call to accept `acceptWithdrawRequest()` with the withdrawal `ID` and and `payBackRatio`.

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L236C1-L267C10
```solidity
    function acceptWithdrawRequest(
        uint256 providedAmount,
        uint256[] memory _acceptedRequestIds,
        uint256 _paybackRatio
    ) external onlyRole(BALANCER_ROLE) whenNotPaused {
..SNIP..
        require(
            _paybackRatio >= minimumPaybackRatio,
            "SolverVault: Payback ratio is too low"
..SNIP..
        for (uint256 i = 0; i < _acceptedRequestIds.length; i++) {
..SNIP..
            withdrawRequests[id].status = RequestStatus.Ready;
            withdrawRequests[id].acceptedRatio = _paybackRatio;
        }
```
the issue arises from the fact that when users try to `claimForWithdrawRequest()` the amount of their collateral sent to them, will depend on their `request.acceptedRatio`.

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L282C1-L299C6
```solidity
    function claimForWithdrawRequest(uint256 requestId) external whenNotPaused {
..SNIP..
        WithdrawRequest storage request = withdrawRequests[requestId];
..SNIP..
        request.status = RequestStatus.Done;
        uint256 amount = (request.amount * request.acceptedRatio) / 1e18;
        lockedBalance -= amount;
        IERC20(collateralTokenAddress).safeTransfer(request.receiver, amount);
        emit WithdrawClaimedEvent(requestId, request.receiver);
    }
```
in a scenario where the ratio of the amount less than `100%` the remaining share of the amount the user requested to withdraw will be lost as `100%` of the solver vault token representing the amount of collateral the user requested has been burnt already on the call to `requestWithdraw()`.
`Example Scenario:`
 1)  User deposits `100 USDc` and receives `100 SVT` representing his `100 USDc` in the vault.
 2)  User requests to withdraw his `100 USDc` on the call to `requestWithdraw()` His `100 SVT` is burnt and a request is created.
 3)  Balancer `acceptWithdrawRequest()` with `payBackRatio` of `50%` as seen in the `Tests`(`5e17`).
 4)  User `claimForWithdrawRequest()` and will receive `50 USDc` because the `payBackRatio` was set to `50%`, the User loses his remaining `50 USDc` as all hie `100 SVT` tokens that represent his share of collateral has been burnt on the call to `requestWithdraw()`.

**As per the sponsors this is a known risk that the users will accept before the deposit into the vault.**

**THE ISSUE**
The main functionality of the contracts is to enable users to deposit funds into the vault to help fund Symmio Hedge(solver). For this to be possible Users are incentivized to deposit with the idea that, an equivalent amount of Solver Vault Tokens will be minted per the collateral the user deposits as we can see in the `README.md`
```
the contract issues vault tokens in a 1:1 ratio with the deposited amount. Users can stake these vault tokens in another
-
contract to earn returns. They also have the option to withdraw their funds anytime by returning their vault tokens.
```
So it is When a user has `1 SVT`  he can sell for `1 USDc`, and the can stake the `SVT` tokens somewhere else to earn yeild.

`However with the risk mentioned earlier above, two problems arise:`
- User will not want to deposit USDc/USDt into the vault because there is no certainity that they can ever recovery their enitre amount of collateral for a specfic amount of `SVT` tokens, so why deposit in the first place.(`The economice Incentive is broken here`).
-  Since with `100 SVT` tokens it isn't certain that `100 USDc` will be returned, `SVT` tokens will be sold in the open market for lesser amount of `USDc`, thereby breaking the1:1 peg for the vault tokens to `USDc`in the open market, so why deposit your `USDc` into the vault to get `SVT` tokens when you can simply by more on the open market and withdraw from the vault.
## Impact
- User cannot witdraw all the collateral for equivalent amount of `SVT` tokens due to buring all the amount but sending a lesser amount of collateral when the claim withdrawal request due to the `payBackRatio`, and because of that the can’t request withdrawal for the remaining token.
- There is no Incentive for users to deposit into the vaults as the 1:1 peg  for `SVT` to `USDc` will be broken so why deposit `USDc` into the vault to get equivalent amount of `SVT`.
## Code Snippet
- https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L210C1-L210C80
- https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L295C1-L297C79
## Tool used

Manual Review, Foundry, README.md

## Recommendation
**THE SOLUTION**
On the call to `claimForWithdrawRequest()` Users should be minted back any remaining amount of `SVT` the vault wasn't able to pay them back due to the amount of collateral deposit in the contract.

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L282C1-L299C6
```diff
   function claimForWithdrawRequest(uint256 requestId) external whenNotPaused {
        require(
            requestId < withdrawRequests.length,
            "SolverVault: Invalid request ID"
        );
        WithdrawRequest storage request = withdrawRequests[requestId];

        require(
            request.status == RequestStatus.Ready,
            "SolverVault: Request not ready for withdrawal"
        );

        request.status = RequestStatus.Done;
        uint256 amount = (request.amount * request.acceptedRatio) / 1e18;
        lockedBalance -= amount;
        IERC20(collateralTokenAddress).safeTransfer(request.receiver, amount);
+     if(request.amount > amount){
+     uint256 amountLeft = request.amount - amount;
+     SolverVaultToken(solverVaultTokenAddress).mint(request.receiver, amountLeft);
+     }
        emit WithdrawClaimedEvent(requestId, request.receiver);
    }
```