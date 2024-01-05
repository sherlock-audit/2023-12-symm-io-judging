Faithful Peanut Goose

high

# Users will Always loose their funds Deposited Into The Protocol

## Summary
The vulnerability lies in the `SolverVaults:claimForWithdrawRequest` function, where the calculation for the amount to be sent to the receiver is found to be significantly inaccurate.This vulnerability is identifed when `collateralTokenDecimals == solverVaultTokenDecimals`, It is also important to note that there is no provision to calculate `amount` to be withdrawn for variations in `collateralTokenDecimals and solverVaultTokenDecimals`


## Vulnerability Detail
In the identified function, the calculation error results in users losing their funds automatically. The withdrawal amount is miscalculated, leaving a substantial portion of the tokens in the vault. Despite the incorrect calculation, the `Request Status` of the transaction is marked as `Done` on the chain.

## Impact
Users are adversely affected as a result of the incorrect calculation during the withdrawal process. This issue has the potential to lead to substantial financial losses.

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L295

## Proof Of Concept
```solidity
      function testuserBalance(uint64 amount) public {
       uint256 newAmount = bound(amount, 10 ether, type(uint64).max);
        vm.startPrank(userA);
        USDT.approve(address(SV), USDT.balanceOf(userA));
        uint inititalBalance = USDT.balanceOf(address(userA));
        SV.deposit(newAmount); // vm.stopPrank();
        console.log("initial usdt balance of user ",inititalBalance);
        console.log("balance of vault after first deposit",USDT.balanceOf(address(SV)));

        SVT.approve(address(SV), SVT.balanceOf(userA));

        SV.requestWithdraw(newAmount, userA);
        vm.stopPrank();

        vm.startPrank(user);

        uint256[] memory acceptedRequestIds = new uint256[](1);
        acceptedRequestIds[0] = 0;
        USDT.approve(address(SV), USDT.balanceOf(user));

        SV.acceptWithdrawRequest(newAmount, acceptedRequestIds, 1);
        SV.claimForWithdrawRequest(0);
        uint finalBalance = USDT.balanceOf(address(userA)) ;
        console.log("final usdt balance of user",finalBalance);
        console.log("balance of vault after withdraw", USDT.balanceOf(address(SV)));
        assert(finalBalance == inititalBalance);

        vm.stopPrank();
    }

```
 ## LOGS ---

```solidity
    Bound Result 10000000000000000000
  initial usdt balance of user  340282366920938463463374607431768211455
  balance of vault after first deposit 10000000000000000000
  final usdt balance of user 340282366920938463453374607431768211465
  balance of vault after withdraw 19999999999999999990
```
 
## Tool used
Fuzz Testing

Manual Review

## Recommendation
It is strongly advised to conduct a thorough review and correction of the `SolverVaults:claimForWithdrawRequest` function   to rectify the inaccurate calculation of the amountLockedUp variable To acomodate situations where  `collateralTokenDecimals >= solverVaultTokenDecimals` and vice versa

