Thankful Topaz Reindeer

high

# SolverVault::minimumPaybackRatio can be set to 0 during initializing and leads to denial of service (DoS)

## Summary
`uint256 minimumPaybackRatio` variable can be set to 0 in the `SolverVaults::initialize` function. The parameter is set during the initializing and comes from outside. Only one check is existing during the function `require(_minimumPaybackRatio <= 1e18, "SolverVault: Invalid ratio")` - https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L90

## Vulnerability Detail
When `minimumPaybackRatio = 0`  this can cause very big issue in the future, because the  amount of the `lockedbalance` also will be always `0`.

In `SolverVaults::acceptWithdrawRequest` function we check the following `require(_paybackRatio >= minimumPaybackRatio, "SolverVault: Payback ratio is too low");` and if the parameter `_paybackRatio is equal to 0` and also `minimumPaybackRatio = 0` this equation `totalRequiredBalance += (withdrawRequests[id].amount * _paybackRatio) / 1e18;` will be invalid, because `totalRequiredBalance` will be always `0`.  

## Impact
`lockedBalance variable` always will be equal to `0` because in  `SolverVaults::acceptWithdrawRequest` ``lockedBalance = totalRequiredBalance`

Users will never receive their claims for withdraw, because in `SolverVaults::claimForWithdrawRequest` function when try to do the following `lockedBalance -= amount;` on line 203 and the variable `lockedBalance` will underflow and the function will revert. In that case all funds will stay in the protocol and the users can not withdraw their claims.

## Code Snippet
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L90
```javascript
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
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L246
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L262
https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L274C11-L274
```javascript
function acceptWithdrawRequest(
        uint256 providedAmount,
        uint256[] memory _acceptedRequestIds, // q what will happend if the provided array has invalid data
        uint256 _paybackRatio
    ) external onlyRole(BALANCER_ROLE) whenNotPaused {
        IERC20(collateralTokenAddress).safeTransferFrom(msg.sender, address(this), providedAmount);
        require(_paybackRatio >= minimumPaybackRatio, "SolverVault: Payback ratio is too low");
        uint256 totalRequiredBalance = lockedBalance;

        for (uint256 i = 0; i < _acceptedRequestIds.length; i++) {
            uint256 id = _acceptedRequestIds[i];
            require(id < withdrawRequests.length, "SolverVault: Invalid request ID");
            require(withdrawRequests[id].status == RequestStatus.Pending, "SolverVault: Invalid accepted request");
            totalRequiredBalance += (withdrawRequests[id].amount * _paybackRatio) / 1e18;
            withdrawRequests[id].status = RequestStatus.Ready;
            withdrawRequests[id].acceptedRatio = _paybackRatio;
        }

        require(
            IERC20(collateralTokenAddress).balanceOf(address(this)) >= totalRequiredBalance,
            "SolverVault: Insufficient contract balance"
        );
        lockedBalance = totalRequiredBalance;
        emit WithdrawRequestAcceptedEvent(providedAmount, _acceptedRequestIds, _paybackRatio);
    }
```

https://github.com/sherlock-audit/2023-12-symm-io/blob/main/solver-vaults/contracts/SolverVaults.sol#L296
```javascript
function claimForWithdrawRequest(uint256 requestId) external whenNotPaused {
        require(requestId < withdrawRequests.length, "SolverVault: Invalid request ID");
        WithdrawRequest storage request = withdrawRequests[requestId];

        require(request.status == RequestStatus.Ready, "SolverVault: Request not ready for withdrawal");

        request.status = RequestStatus.Done;
        uint256 amount = (request.amount * request.acceptedRatio) / 1e18;
        lockedBalance -= amount;
        IERC20(collateralTokenAddress).safeTransfer(request.receiver, amount);
        emit WithdrawClaimedEvent(requestId, request.receiver);
    }

    function pause() external onlyRole(PAUSER_ROLE) {
        _pause();
    }

    function unpause() external onlyRole(UNPAUSER_ROLE) {
        _unpause();
    }
```

## Tool used

Manual Review

## Recommendation
Add one more require statement to validate the input parameter of `_minimumPaybackRatio`. 

```diff
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
+        require(_minimumPaybackRatio > 0, "SolverVault: Invalid ratio");

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
