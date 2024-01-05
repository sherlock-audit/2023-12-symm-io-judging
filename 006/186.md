Powerful Smoke Narwhal

medium

# Incorrect logic in `SolverVaults.setSymmioAddress` will not let even callers with `SETTER_ROLE` set new `symmio`

## Summary
`collateralTokenAddress` changes while executing `setSymmioAddress` which will make `require` statement in the end always revert. 

## Vulnerability Detail
- `setSymmioAddress` function updates `symmio`  and stores the current `collateralTokenAddress` into `beforeCollateral`
-  then it calls `updateCollateral()` which changes `collateralTokenAddress` to collateral address of newly `symmio` 
- so `require` statement in `SolverVaults.setSymmioAddress` implements incorrect comparison of `beforeCollateral` and `collateralTokenAddress` as they will not be equal. 

## Impact
This will make `SolverVaults.setSymmioAddress` always revert even if its done by addresses with `SETTER_ROLE`

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L115C13-L115C30

```solidity
function setSymmioAddress(
        address _symmioAddress
    ) public onlyRole(SETTER_ROLE) {
        require(_symmioAddress != address(0), "SolverVault: Zero address");
        symmio = ISymmio(_symmioAddress);
        address beforeCollateral = collateralTokenAddress;
        updateCollateral();
        require(
            beforeCollateral == collateralTokenAddress || // these two will never be equal
                beforeCollateral == address(0),
            "SolverVault: Collateral can not be changed"
        );
        emit SymmioAddressUpdatedEvent(_symmioAddress);
    }
```

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L128

```solidity
function updateCollateral() internal {
        collateralTokenAddress = symmio.getCollateral(); //this change which will make require fail. 
        collateralTokenDecimals = IERC20Metadata(collateralTokenAddress)
            .decimals();
        require(
            collateralTokenDecimals <= 18,
            "SolverVault: Collateral decimals should be lower than 18"
        );
    }
```

## POC

I used the test file provided in the repo and added this simple describe block with every variable in this block already declared and defined in the `beforeEach` block of the `SolverVault.test.ts` 
```javascript
describe("testing setSymmioAddress", async function() {
        it("should fail to update symmio", async () => {
            const addressOfSymioWithDifferentCollateral = await symmioWithDifferentCollateral.getAddress()
            console.log(addressOfSymioWithDifferentCollateral)
            await expect(solverVault.connect(setter).setSymmioAddress(addressOfSymioWithDifferentCollateral)).to.be.revertedWith("SolverVault: Collateral can not be changed")
            
        })
    })
```

```shell
testing setSymmioAddress
0x1c85638e118b37167e9298c2268758e058DdfDA0
      ✔ should fail to update symmio
```

As you can see it is console logging a valid address so require statement is not failing because of address(0) check but actually because of incorrect comparison of `beforeCollateral` and `collateralTokenAddress`. 

Also I have used `revertedWith("SolverVault: Collateral can not be changed")` with `setter` address as the caller so the function is definitely not reverting because of role mismatch. 

## Tool used
Manual Review, Hardhat

## Recommendation
Improve the logic so that `beforeCollateral` is not being compared to current state value of `collateralTokenAddress`