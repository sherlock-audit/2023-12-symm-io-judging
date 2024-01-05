Zealous Quartz Beaver

medium

# Collateral Token can be Frozen permanently for a withdraw request, missing address validity check

## Summary
The `amount` of user's collateral token can  be frozen permanently if the `SolverVaults::requestWithdraw` function is called with `receiver` as  invalid address and the request gets accepted by the `BALANCER_ROLE`.

## Vulnerability Detail
The `receiver` parameter in  `SolverVaults::requestWithdraw` does not have address validity check.
This can lead to the user's fund getting permanently frozen in the contract 

Steps to reproduce :-
1. User calls `SolverVaults::requestWithdraw` with `amount` = any, `receiver` = zero_address
2. Balancer approves the above request 

Proof of code is attached herewith, to test the same follow the steps
1. copy the code in `solver-vaults/test/SolverVaultM1.challenge.ts`

```typescript
import {expect} from "chai"
import {Signer, ZeroAddress} from "ethers"
import {ethers, upgrades} from "hardhat"

function decimal(n: number, decimal: bigint = 18n): bigint {
    return BigInt(n) * (10n ** decimal)
}
const zero_address = "0x0000000000000000000000000000000000000000"

describe('M1', function() {
    let solverVault: any, collateralToken: any, symmio: any, solverVaultToken: any
    let owner: Signer, user: Signer, depositor: Signer, balancer: Signer, receiver: Signer, setter: Signer,
        pauser: Signer, unpauser: Signer, solver: Signer, other: Signer
    let DEPOSITOR_ROLE, BALANCER_ROLE, MINTER_ROLE, PAUSER_ROLE, UNPAUSER_ROLE, SETTER_ROLE

    let collateralDecimals: bigint = 6n, solverVaultTokenDecimals: bigint = 8n
    const depositLimit = decimal(100000)

    async function mintFor(signer: Signer, amount: BigInt) {
        await collateralToken.connect(owner).mint(signer.getAddress(), amount)
        await collateralToken.connect(user).approve(await solverVault.getAddress(), amount)
    }

    function convertToVaultDecimals(depositAmount: bigint) {
        return solverVaultTokenDecimals >= collateralDecimals ?
            depositAmount * (10n ** (solverVaultTokenDecimals - collateralDecimals)) :
            depositAmount / 10n ** (collateralDecimals - solverVaultTokenDecimals)
    }

    // setup
    before(async function() {
        [owner, user, depositor, balancer, receiver, setter, pauser, unpauser, solver, other] = await ethers.getSigners()

        const SolverVault = await ethers.getContractFactory("SolverVault")
        const MockERC20 = await ethers.getContractFactory("MockERC20")
        const Symmio = await ethers.getContractFactory("MockSymmio")

        collateralToken = await MockERC20.connect(owner).deploy(collateralDecimals)
        await collateralToken.waitForDeployment()

        symmio = await Symmio.deploy(await collateralToken.getAddress())
        await symmio.waitForDeployment()

        solverVaultToken = await MockERC20.deploy(solverVaultTokenDecimals)
        await solverVaultToken.waitForDeployment()


        solverVault = await upgrades.deployProxy(SolverVault, [
            await symmio.getAddress(),
            await solverVaultToken.getAddress(),
            await solver.getAddress(),
            500000000000000000n, // 0.5,
            depositLimit,
        ])

        DEPOSITOR_ROLE = await solverVault.DEPOSITOR_ROLE()
        BALANCER_ROLE = await solverVault.BALANCER_ROLE()
        SETTER_ROLE = await solverVault.SETTER_ROLE()
        PAUSER_ROLE = await solverVault.PAUSER_ROLE()
        UNPAUSER_ROLE = await solverVault.UNPAUSER_ROLE()
        BALANCER_ROLE = await solverVault.BALANCER_ROLE()
        MINTER_ROLE = await solverVaultToken.MINTER_ROLE()

        await solverVault.connect(owner).grantRole(DEPOSITOR_ROLE, depositor.getAddress())
        await solverVault.connect(owner).grantRole(BALANCER_ROLE, balancer.getAddress())
        await solverVault.connect(owner).grantRole(SETTER_ROLE, setter.getAddress())
        await solverVault.connect(owner).grantRole(PAUSER_ROLE, pauser.getAddress())
        await solverVault.connect(owner).grantRole(UNPAUSER_ROLE, unpauser.getAddress())
        await solverVaultToken.connect(owner).grantRole(MINTER_ROLE, solverVault.getAddress())
    })



    describe('Execution', function () {
        const depositAmount = decimal(500, collateralDecimals)
        const withdrawAmountInCollateralDecimals = depositAmount
        const withdrawAmount = convertToVaultDecimals(depositAmount)

        const requestId = 0
        const requestIds = [requestId]
        const paybackRatio = decimal(70, 16n)

        beforeEach(async function () {
            await mintFor(user, depositAmount)
            await solverVault.connect(user).deposit(depositAmount)
            await solverVaultToken.connect(user).approve(await solverVault.getAddress(), withdrawAmount)
        })

        it("POC", async function() {
            const rec = zero_address 
            // user request withdraw with invalid address
            await expect(solverVault.connect(user).requestWithdraw(withdrawAmount, rec))
                    .to.emit(solverVault, "WithdrawRequestEvent")
                    .withArgs(0, rec, withdrawAmountInCollateralDecimals)

            // balancer accepts the request
            await expect(solverVault.connect(balancer).acceptWithdrawRequest(0, requestIds, paybackRatio))
                    .to.emit(solverVault, "WithdrawRequestAcceptedEvent")
                    .withArgs(0, requestIds, paybackRatio)

            // user funds are locked forever
            const request = await solverVault.withdrawRequests(0)
            console.log(await solverVault.lockedBalance())
            expect(await solverVault.lockedBalance()).to.equal(request.amount * paybackRatio / decimal(1))
        })
        
    });
    
})

```

2. run `npx hardhat test test/SolverVaultM1.challenge.ts`


## Impact
Medium

## Code Snippet

```solidity
function requestWithdraw(
        uint256 amount,
        address receiver
    ) external whenNotPaused {
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

        currentDeposit -= amountInCollateralDecimals;

        withdrawRequests.push(
            WithdrawRequest({
                receiver: receiver,
                amount: amountInCollateralDecimals,
                status: RequestStatus.Pending,
                acceptedRatio: 0
            })
        );
        emit WithdrawRequestEvent(
            withdrawRequests.length - 1,
            receiver,
            amountInCollateralDecimals
        );
    }

```

## Tool used

Manual Review

## Recommendation
In `SolverVaults::requestWithdraw` below [line](https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L205) add this 


```diff
require(
    SolverVaultToken(solverVaultTokenAddress).balanceOf(msg.sender) >=
        amount,
        "SolverVault: Insufficient token balance"
);
+ require(receiver != address(0),  "Receiver:  Zero address")

```



