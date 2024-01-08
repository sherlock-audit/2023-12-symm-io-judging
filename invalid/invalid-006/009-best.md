Shaggy Taupe Wombat

high

# If USDT token started taking fee on transfer, contract will break

## Summary
Contract does not work with USDT, which is fee-on-transfers token

## Vulnerability Detail
USDT token is listed as token that will interact with this contract:

        Which ERC20 tokens do you expect will interact with the smart contracts?

        USDT, USDC

Despite it is said that fee-on-transfer token will not interact with this contract, but USDT is also fee-on-transfer token, but for now the fee is set to 0 (see the contract here: https://etherscan.io/address/0xdAC17F958D2ee523a2206206994597C13D831ec7#code). For these tokens, it should not be assumed that if you transfer `x` tokens to an address, that the address actually receives `x` tokens. Current design of protocol are not work with fee-on-transfer token. As it will mint extraactly token based on input;

    function deposit(uint256 amount) external whenNotPaused {
        .    .    .    .    .    .    .    .    .    .    .    .    .
        uint256 amountInSolverVaultTokenDecimals = solverVaultTokenDecimals >=
            collateralTokenDecimals
            ? amount *
                (10 ** (solverVaultTokenDecimals - collateralTokenDecimals))
            : amount /
                (10 ** (collateralTokenDecimals - solverVaultTokenDecimals));

        SolverVaultToken(solverVaultTokenAddress).mint(
            msg.sender,
            amountInSolverVaultTokenDecimals
        );
        currentDeposit += amount;
        .    .    .    .    .    .    .    .    .    .    .    .    .
    }
Currently, fee of USDT token is set to 0. But in the future, it can be changed, and contract will be break

## Impact
Last users are not able to withdraw collateral token because lack of token in contract

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L168-#L179

## Tool used
Manual Review

## Recommendation
There should be pre and post checks on balances to get the real amount of token.