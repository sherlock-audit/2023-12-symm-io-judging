Early Sable Ape

high

# Mint functionality won't work

## Summary
In the constructor of SolverVaults.sol vault is not granted the minter role due to which when minting of SolverVaultToken won't be possible

## Vulnerability Detail
Following is deposit function used by the user in order to mint solverVault tokens .
```solidity
 function deposit(uint256 amount) external whenNotPaused {
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
```
When the following call is made 
```solidity
  SolverVaultToken(solverVaultTokenAddress).mint(
            msg.sender,
            amountInSolverVaultTokenDecimals
        );
```
mint function of solver vault token is as follows
```solidity
function mint(address to, uint256 amount) public onlyRole(MINTER_ROLE) {
        _mint(to, amount);
    }
```
From above it can be seen that it has modifier onlyRole(MINTER_ROLE) and when the following call is made SolverVaultToken(solverVaultTokenAddress).mint(
            msg.sender,
            amountInSolverVaultTokenDecimals
        );
msg.sender is the vault contract and is doesn't have minter role as can be seen from the constructor which is as follows 

```solidity

    function initialize(
        address _symmioAddress,
        address _symmioVaultTokenAddress,
        address _solver,
        uint256 _minimumPaybackRatio,
        uint256 _depositLimit
    ) public initializer {
        __AccessControl_init();
        __Pausable_init();
        
        require(_minimumPaybackRatio <= 1e18, "SolverVault: Invalid ratio");
        
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEPOSITOR_ROLE, msg.sender);
        _grantRole(BALANCER_ROLE, msg.sender);
        _grantRole(SETTER_ROLE, msg.sender);
        _grantRole(PAUSER_ROLE, msg.sender);
        _grantRole(UNPAUSER_ROLE, msg.sender);
        setSymmioAddress(_symmioAddress);
        setSymmioVaultTokenAddress(_symmioVaultTokenAddress);
        setDepositLimit(_depositLimit);
        setSolver(_solver);
        lockedBalance = 0;
        currentDeposit = 0;
        minimumPaybackRatio = _minimumPaybackRatio;
    }
```

## Impact
DOS of mint functionality

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L175
## Tool used

Manual Review

## Recommendation
modifiy the constructor as follows
```solidity
 function initialize(
        address _symmioAddress,
        address _symmioVaultTokenAddress,
        address _solver,
        uint256 _minimumPaybackRatio,
        uint256 _depositLimit
    ) public initializer {
        __AccessControl_init();
        __Pausable_init();
        
        require(_minimumPaybackRatio <= 1e18, "SolverVault: Invalid ratio");
        
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        _grantRole(DEPOSITOR_ROLE, msg.sender);
        _grantRole(BALANCER_ROLE, msg.sender);
        _grantRole(SETTER_ROLE, msg.sender);
        _grantRole(PAUSER_ROLE, msg.sender);
        _grantRole(UNPAUSER_ROLE, msg.sender);
=>    _grantRole(UNPAUSER_ROLE, address(this));
        setSymmioAddress(_symmioAddress);
        setSymmioVaultTokenAddress(_symmioVaultTokenAddress);
        setDepositLimit(_depositLimit);
        setSolver(_solver);
        lockedBalance = 0;
        currentDeposit = 0;
        minimumPaybackRatio = _minimumPaybackRatio;
    }
```
and initilize the following 
```solidity
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
