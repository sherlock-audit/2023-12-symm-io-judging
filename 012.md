Nutty Wooden Pig

medium

# Anyone can call the function `claimForWithdrawRequest` and claim withdraw for other users

## Summary
Anyone can call the function `claimForWithdrawRequest` and claim withdraw for other users. 

## Vulnerability Detail
`claimForWithdrawRequest` is only set to external and whenNotPaused. Therefore, anyone could call the function on behalf of another user and force the transfer of the user's collateral from the contract to his wallet/contract.

## Impact
Executing the `claimForWithdrawRequest` function successfully enables the manipulation of the `lockedBalance` state variable by reducing it by the withdrawn amount. Consequently, this action facilitates the acceptance of additional withdrawal requests.

## Proof of Concept

```javascript
            describe("claimForWithdrawRequest", function () {
                const requestId = 0
                let lockedBalance: bigint

                beforeEach(async function () {
                    solverVault.connect(balancer).acceptWithdrawRequest(0, requestIds, paybackRatio)
                    lockedBalance = (await solverVault.withdrawRequests(0)).amount * paybackRatio / decimal(1)
                })

                it("should claim withdraw (receiver)", async function () {
                    await expect(solverVault.connect(receiver).claimForWithdrawRequest(requestId))
                        .to.emit(solverVault, "WithdrawClaimedEvent")
                        .withArgs(requestId, await receiver.getAddress())
                    const request = await solverVault.withdrawRequests(0)
                    expect(request[2]).to.equal(RequestStatus.Done)
                })

                it("should claim withdraw (from another user)", async function () {
                    await expect(solverVault.connect(user).claimForWithdrawRequest(requestId))
                        .to.emit(solverVault, "WithdrawClaimedEvent")
                        .withArgs(requestId, await receiver.getAddress())
                    const request = await solverVault.withdrawRequests(0)
                    expect(request[2]).to.equal(RequestStatus.Done)
                })
            })
```

## Code Snippet

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L282

## Tool used

Manual Review

## Recommendation
We recommend to check for `msg.sender` to be the `withdrawRequests[requestId].receiver`
