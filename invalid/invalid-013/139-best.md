Teeny Champagne Yak

high

# Users can deposit tokens not USDT, USDC into the vault

## Summary
Users can deposit tokens not USDT, USDC into the vault

## Vulnerability Detail
Based on the information provided by the protocol, the contract is expected to only interact with USDC and USDT. These are 6 decimal tokens.

> Which ERC20 tokens do you expect will interact with the smart contracts?
> 
> USDT, USDC
> 

However, there is no check in place to ensure the above in the deposit, setSymmioVaultTokenAddress, and updateCollateral functions. Even in setSymmioVaultTokenAddress and updateCollateral functions, the require statements check is against 18 decimals which is not suppose to be.

## Impact
Users can deposit tokens different from USDC and USDT which can cause wrong calculation in amount minted to users.

## Code Snippet
`function deposit(uint256 amount) external whenNotPaused {
        require(
            currentDeposit + amount <= depositLimit,
            "SolverVault: Deposit limit reached"
        );
        IERC20(collateralTokenAddress).safeTransferFrom(
            msg.sender,
            address(this),
            amount
        );
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
        emit Deposit(msg.sender, amount);
    }
`
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L158-L182

` function updateCollateral() internal {
        collateralTokenAddress = symmio.getCollateral();
        collateralTokenDecimals = IERC20Metadata(collateralTokenAddress)
            .decimals();
        require(
            collateralTokenDecimals <= 18,
            "SolverVault: Collateral decimals should be lower than 18"
        );
    }
`
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L128-L137

`function setSymmioVaultTokenAddress(
        address _symmioVaultTokenAddress
    ) internal {
        require(_symmioVaultTokenAddress != address(0), "SolverVault: Zero address");
        solverVaultTokenAddress = _symmioVaultTokenAddress;
        solverVaultTokenDecimals = SolverVaultToken(_symmioVaultTokenAddress)
            .decimals();
        require(
            solverVaultTokenDecimals <= 18,
            "SolverVault: SolverVaultToken decimals should be lower than 18"
        );
    }`
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L138-L149

## Tool used

Manual Review

## Recommendation
Deposited tokens should be restricted to USDC and USDT
