Melted Brunette Cougar

high

# [H-2] `SolverVaults::setDepositLimit` function doesn't check existing deposits against the `SolverVaults::depositLimit` state variable therefore potentially halting existing depositors from depositing further

## Summary
The `setDepositLimit` function allows the modification of the deposit limit without considering the current total deposited amount `currentDeposit`. This design oversight means that if the deposit limit is reduced midway to a value below the current total deposits, it could erroneously prevent existing depositors who have not reached their individual deposit limits from depositing further funds. This issue arises because the new deposit limit check does not account for the already deposited amounts by users.

## Vulnerability Detail
Add the below test case to `test/SolverVault.test.ts`

<details>
<summary>PoC</summary>
</br>

```javascript
//existing unit tests under 'deposit' test suite

it("should not allow existing depositors to deposit up to the original limit even after lowering the deposit limit", async function () {
  // Let depositor deposit an amount within the initial limit
  const initialDeposit = depositLimit - decimal(10000);
  await mintFor(user, initialDeposit);
  await solverVault.connect(user).deposit(initialDeposit);

  // Lower the deposit limit to less than the current total deposit
  const newLimit = initialDeposit - decimal(1000);
  await solverVault.connect(setter).setDepositLimit(newLimit);

  // Attempt to deposit again - should be allowed up to the original limit
  const additionalDeposit = decimal(5000);
  await mintFor(user, additionalDeposit);
  await expect(
    solverVault.connect(user).deposit(additionalDeposit)
  ).to.be.revertedWith("SolverVault: Deposit limit reached");
});
```
</details>


## Impact
1. Restricted User Access: Existing users who have not reached their deposit limits could be unjustly prevented from depositing additional funds.
2. Reduced Liquidity and Participation: The contract's functionality and appeal could be significantly hampered, as users might find it unreliable if deposit limits can change unpredictably.
3. Potential for Unintended Contract Lockdown: In extreme cases, setting the deposit limit too low compared to the current deposits might effectively lock the contract from receiving new deposits, impacting its operational effectiveness.

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L154

<details>
<summary>Code</summary>
</br>

```javascript
function setDepositLimit(
    uint256 _depositLimit
) public onlyRole(SETTER_ROLE) {
    //@audit-info : if midway depositLimit is reduced, existing users cannot deposit more
@>  depositLimit = _depositLimit;
    emit DepositLimitUpdatedEvent(_depositLimit);
}
```

</details>

## Tool used

- Manual Review

## Recommendation
Proposed Changes in `SolverVaults` contract:

<details>
<summary>Code</summary>
</br>

```diff
contract SolverVault {
+ uint256 public globalDepositLimit;
  uint256 public depositLimit;

    // Percentage of global limit for individual limit (e.g., 10%)
    //feel free to include a setter function for this/not make it a constant
+   uint256 private constant INDIVIDUAL_LIMIT_PERCENTAGE = 10;

    ...

    function setDepositLimit(
-    uint256 _depositLimit
+    uint256 _globalDepositLimit
    ) public onlyRole(SETTER_ROLE) {
+   require(_globalDepositLimit >= individualDepositLimit, "Global limit cannot be less than individual limit");

+   globalDepositLimit = _globalDepositLimit;

-   depositLimit = _depositLimit;
+    depositLimit = (globalDepositLimit * INDIVIDUAL_LIMIT_PERCENTAGE) / 100;

-    emit DepositLimitUpdatedEvent(_depositLimit);
+    emit DepositLimitUpdatedEvent(_globalDepositLimit, depositLimit);
    }

    ...
}
```

</details>