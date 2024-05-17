# QA Report for Stader Labs contest

## Overview
During the audit, 9 low and 4 non-critical issues were found.

#
## Low Risk Issues (9)
#

### [L-01] Project has NPM Dependency which uses a vulnerable version: @openzeppelin/contracts-upgradeable
```package.json

38: "@openzeppelin/contracts-upgradeable": "^4.8.0-rc.1",
```
```js
VULNERABILITY	VULNERABLE VERSION
L
Missing Authorization	
>=4.3.0 <4.9.1
L
Denial of Service (DoS)	
>=3.2.0 <4.8.3
M
Improper Input Validation	
>=4.3.0 <4.8.3
M
Incorrect Calculation	
>=4.8.0 <4.8.2
M
Incorrect Calculation	
>=4.8.0 <4.8.2
```

### Proof Of Concept
https://security.snyk.io/package/npm/@openzeppelin%2Fcontracts-upgradeable

### Recommended Mitigation Steps
Upgrade OZ to version 4.9.0 or higher


### [L-02] Missing docstrings
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the [Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html). In complex projects, the interpretation of all functions and their arguments and returns is important for code readability and auditability. 

### Recommended Mitigation Steps
NatSpec comments should be increased in contracts. Follow your our best practice like [PermissionedNodeRegistry.sol](https://github.com/code-423n4/2023-06-stader/blob/main/contracts/PermissionedNodeRegistry.sol).


### [L-03] This function OperatorRewardsCollector.depositFor dont check _receiver != address(0) && msg.value > 0
In the following fuction OperatorRewardsCollector.depositFor its no checks there, we recomended put some checks here.

```solidity
40: function depositFor(address _receiver) external payable {
        // @audit add the checks here
        require(_receiver != address(0) && msg.value > 0, "add valid params");  <----------

        balances[_receiver] += msg.value;

        emit DepositedFor(msg.sender, _receiver, msg.value);
    }
```

### [L-04] The onlyManagerRole can abuse with penalties
In the contract `Penalty.setAdditionalPenaltyAmount` the onlyManagerRole can abuse the penalties by not specifying what will be the maximum amount per extra penalty.

### Recommended Mitigation Steps
We recommend making a constant variable with the value of max-amount-per-penalty and then if the onlyManagerRole  wants to have a bit more flexibility, depending on the possible infraction he can place a require that specifies that the amount can be less than or equal to max-amount-per-penalty.

```solidity
53: function setAdditionalPenaltyAmount(bytes calldata _pubkey, uint256 _amount) external override {
        // @audit how could look the change
        require(_amount <= max-amount-per-penalty);  <----------

        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        bytes32 pubkeyRoot = UtilLib.getPubkeyRoot(_pubkey);
        additionalPenaltyAmount[pubkeyRoot] += _amount;

        emit UpdatedAdditionalPenaltyAmount(_pubkey, _amount);
    }
```

### [L-05] Risks of Centralized Protocol
The protocol specifies that it is a multi-pool architecture for node operations, designed for decentralization and scalability. However, it utilizes proxies in the contracts, and the administrators have significant control over the project, specifically being able to make changes to all contracts from `StaderConfig.sol`. This introduces risks ranging from potential failures and censorship to lack of transparency and security. When evaluating and using protocols in this environment, it is crucial to consider these risks and seek decentralized and resilient solutions that promote user security, transparency, and freedom as the project evolves.

```
Stader Labs
ETHx is a multi pool architecture for node operations, designed for `decentralization`, scalability, and resilience.
```

### [L‑06] Owner can renounce while system is paused
The contract owner or single user with a role is not prevented from renouncing the role/ownership while the contract is paused, which would cause any user assets stored in the protocol, to be locked indefinitely.

```solidity
PermissionlessNodeRegistry.sol

468: function pause() external override {
        UtilLib.onlyManagerRole(msg.sender, staderConfig);
        _pause();
    }
```

### [L‑07] Crucial information is missing on important functions in all contracts
In the initialize functions it is not specified in the documentation if the _admin will be an EOA or a contract.
Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.


### [L‑08] Be more careful with functions that require casting
We have noticed that the contracts `PermissionedPool.sol` and `PermissionlessPool` implements many type conversions from uint256 to uint64. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.

### [L‑09] Array does not have a pop function
Arrays without the pop operation in Solidity can lead to inefficient memory management and increase the likelihood of out-of-gas errors.

```solidity
PermissionedNodeRegistry.sol
174:  validatorIdsByOperatorId[operatorId].push(nextValidatorId);

PermissionlessNodeRegistry.sol
160:  validatorIdsByOperatorId[operatorId].push(nextValidatorId);

PoolUtils.sol
44: poolIdArray.push(_poolId);
```


#
## Non-critical Issues (4) 
#

### [N-01] Naming issue
In the `VaultProxy.sol` contract we recommend refactoring the `VaultProxy.initialise` function in order to greatly benefit development and code understanding by the reviewer.

### Recommended Mitigation Steps
Function `VaultProxy.initialise`  should be renamed to function `VaultProxyInitialize()` to better represent what it does.

```solidity
// @audit refactorize the function with -->  VaultProxyInitialize()
21: function initialise(
        bool _isValidatorWithdrawalVault,
        uint8 _poolId,
        uint256 _id,
        address _staderConfig
    ) external {
        if (isInitialized) {
            revert AlreadyInitialized();
        }
        UtilLib.checkNonZeroAddress(_staderConfig);
        isValidatorWithdrawalVault = _isValidatorWithdrawalVault;
        isInitialized = true;
        poolId = _poolId;
        id = _id;
        staderConfig = IStaderConfig(_staderConfig);
        owner = staderConfig.getAdmin();
    }
```

### [N-02] Use a single file for all System.Wide constants 
In the contracts `Auction.sol`, `ETHx.sol`, `PermissionedNodeRegistry.sol`, `PermissionedPool.sol`, `PermissionlessNodeRegistry.sol`, `PermissionlessPool.sol`, `PoolSelector.sol`, `PoolUtils.sol`, `StaderConfig.sol`, `StaderConfig.sol`, there are many `constanst` used in the system. It is recommended to put the most used ones in one file (for example `Constants.sol`) and use inheritance to access these values.

This helps with readability and easier maintenance for future changes. It also helps with any issues, as some of these hard-coded contracts are admin contracts.

`Constants.sol` Use and import these files in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

### Recommended Mitigation Steps
We strongly recommend implementing the constanst variables in a separate file. This will help you avoid errors and confusion in your project, and maintain a more organized and readable code structure. Following these good programming and organizational practices will help you build a solid and scalable project, which will be easier to maintain and update in the future.

### [N-03] Empty blocks should be removed or emit something 
In Solidity, any function defined in a non-abstract contract must have a complete implementation that specifies what to do when that function is called. In the contracts `VaultProxy.sol`, `NodeELRewardVault.sol`  there are constructors that do not emit nothing.

```solidity
 constructor() {}
```

### Recommended Mitigation Steps
```solidity
contracts/StaderInsuranceFund.sol
// @audit Consider emit something like the contract `StaderInsuranceFund` or simply remove the code block.
    22:  constructor() {
            _disableInitializers();
        }
```

### [N-04] VaultProxy.updateOwner should add one more parameter in the event
`VaultProxy.updateOwner` add one more parameter in the event to be able to continue tracing on chain has been its previous owner.
  