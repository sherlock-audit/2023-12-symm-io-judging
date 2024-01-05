Orbiting Mocha Vulture

medium

# `depositToSymmio` requires collateral token's `approve` to return `true`, however some tokens might not return anything, reverting and blocking deposit functionality

## Summary

When trying to `depositToSymmio` with non-standard collateral token (such as mainnet USDT), the following line will revert due to absence of return value:
```solidity
    require(
        IERC20(collateralTokenAddress).approve(address(symmio), amount),
        "SolverVault: Approve failed"
    );
```
This means that it will be impossible to deposit to symmio and this core functionality will be broken.

## Vulnerability Detail

`ERC20.approve` method should return `bool`, however not all ERC20 tokens fully conform to this standard and some do not return anything. In particular, mainnet's `USDT` token doesn't return anything from `approve`:
```solidity
function approve(address _spender, uint _value) public onlyPayloadSize(2 * 32) {

    // To change the approve amount you first have to reduce the addresses`
    //  allowance to zero by calling `approve(_spender, 0)` if it is not
    //  already 0 to mitigate the race condition described here:
    //  https://github.com/ethereum/EIPs/issues/20#issuecomment-263524729
    require(!((_value != 0) && (allowed[msg.sender][_spender] != 0)));

    allowed[msg.sender][_spender] = _value;
    Approval(msg.sender, _spender, _value);
}
```

## Impact

It is impossible to `depositToSymmio` with non-standard ERC20 collateral tokens which do not return anything from `approve`. While there is a known token (mainnet USDT) with such behaviour, eth mainnet is not listed in intended deployment chains. However, this is still a concern, because there is no guarantee that all tokens in all intended networks will conform to ERC20 standard in regard to `approve` function.

## Code Snippet

`SolverVault.depositToSymmio` requires collateral's `approve` to return `true` (but it might not return anything at all):
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L193-L196

## Tool used

Manual Review

## Recommendation

Consider using `SafeApprove` from `OpenZeppelin`, which is inteded exactly for this purpose and works both when there is no return value and when there is a return value and it's true.