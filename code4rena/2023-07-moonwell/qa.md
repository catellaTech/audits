`Dear Moomwell team, as we have gone through each contract within the scope, we have noticed very good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing.We believe that at least 90% of the recommendations in the following report should be applied for better readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future standasrs.`

# QA Report for Moonwell contest

## [01] In the MToken.transfer() contract, the accrueInterest() should be firts
When the user performs `transfer()`, it will check if there is enough collateral. However, the current transfer is not executed first `accrueInterest()`, so the amount owed by the user may be less than the actual amount owed, resulting in a transfer that would otherwise be impossible.

It is recommended to execute `accrueInterest()` first when transferring:

```diff 
  // @audit in the function transfer
      function transfer(address dst, uint256 amount) override external nonReentrant returns (bool) {
      + accrueInterest();  
        return transferTokens(msg.sender, msg.sender, dst, amount) == uint(Error.NO_ERROR);
      }

  // @audit in the function transferFrom as well
      function transferFrom(address src, address dst, uint256 amount) override external nonReentrant returns (bool) {
      + accrueInterest();   
        return transferTokens(msg.sender, src, dst, amount) == uint(Error.NO_ERROR);
      }    
```
## [02] utilizationRate() could DoS in WhitePaperInterestRateModel.sol contract
Reading the documentation of this function [utilizationRate()](https://github.com/code-423n4/2023-07-moonwell/blob/main/src/core/IRModels/WhitePaperInterestRateModel.sol#L44-L58), they do not perform proper checks before carrying out the respective computation. If the denominator in any case equates 0, `i.e cash.add(borrows).sub(reserves) == 0` the execution reverts due to a division by zero.

### Recommendation
The calculation of the denominator in this case, cash + borrows - reserves should first be checked not to equate to zero before attempting the division.

The function could be changed to:

```solidity
   function utilizationRate(uint cash, uint borrows, uint reserves) public pure returns (uint) {
        // Utilization rate is 0 when there are no borrows
        if (borrows == 0) {
            return 0;
        }

        // @audit - Add some check like this
        If ((cash.add(borrows).sub(reserves)) != 0){
          return borrows.mul(1e18).div(cash.add(borrows).sub(reserves));
        }
          return 0
    }
```
## [03] Interchangeable usage of msg.sender and mToken in Comptroller.borrowAllowed()
Consider replacing `addToMarketInternal(MToken(msg.sender), borrower)` with `addToMarketInternal(MToken(mToken), borrower)`, since `msg.sender` will have to be equal vToken for the [check](https://github.com/code-423n4/2023-07-moonwell/blob/main/src/core/Comptroller.sol#L320) to pass. This can improve code clarity.

## [04] The ChainlinkCompositeOracle.constructor() function does not verify an important inputs
In the constructor function should have a checks that verifies the baseAddress, multiplierAddress, secondMultiplierAddress addressess is not equal to address(0).

## [05] In some functions trough the scope is missing the respective modifiers
In some important functions, the respective modifiers are missing, such as `view/pure`. We recommend adding them for better readability and maintenance of the contracts.

### PROOF OF CONCEPT
```solidity
Some examples in the Comptroller.sol contract.

102: function enterMarkets(address[] memory mTokens) override public returns (uint[] memory)
660: function _setPriceOracle(PriceOracle newOracle) public returns (uint)
880: function _setPauseGuardian(address newPauseGuardian) public returns (uint)
911: function _setMintPaused(MToken mToken, bool state) public returns (bool)
921: function _setBorrowPaused(MToken mToken, bool state) public returns (bool)
931: function _setTransferPaused(bool state) public returns (bool)
940: function _setSeizePaused(bool state) public returns (bool)
```
## [06] In Unitroller contract utilize Assembly - should have comments
The Moonwell team makes use of Assembly on Unitroller contract, since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does.

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Assembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### Recommendation
It is essential to clearly and comprehensively document all activities related to critical function `assembly` within the project. By doing so, a more complete and accurate understanding of what is being done is provided, which is important both for auditors and for long-term project maintenance. By following the practices of monitoring and controlling project work, it can be ensured that all assemblies are properly documented in accordance with the project's objectives and goals.

## [07] Crucial information is missing on important functions in all contracts
In the initialize functions it is not specified in the documentation if the admin will be an EOA or a contract.
Consider improving the docstrings to reflect the exact intended behaviour, and using `Address.isContract` function from OpenZeppelin’s library to detect if an address is effectively a contract.

## [08] In several contracts, NO_ERROR is used. Please consider being more descriptive with the errors.
In most cases the error is `NO_ERROR` even in cases it shouldn't be, this means that users can't really know the reasons to why their tx failed, and is a very big flaw for off-chain services.

### Recommendation
It is recommended to use a more descriptive name than NO_ERROR.

## [09] We recommend documenting and implementing how the project will behave in case timestampsPerYear is not set to 365 days
In the JumpRateModel.sol contract, the variable timestampsPerYear is a constant variable, which means it cannot be modified. This is dangerous because this variable is used in important calculations, and it is not specified how these calculations will behave in cases where a year is not 365 days.

## [10] Inconsistent solidity pragma in through the scope
The source files have different solidity compiler ranges referenced. This leads to potential security flaws between deployed contracts depending on the compiler version chosen for any particular file. It also greatly increases the cost of maintenance as different compiler versions have different semantics and behavior.

### PROOF OF CONCEPT
```solidity
// @audit In some contracts we found this issue.

mip00.sol
2: pragma solidity ^0.8.0;

MToken.sol
2: pragma solidity 0.8.17;

```
### Recommendation
We recommend to fix a definite compiler range that is consistent between contracts and upgrade any affected contracts to conform to the specified compiler.

## [11] The Comptroller.sol contract presents function overloading
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

See more about this on [Solidity Docs.](https://docs.soliditylang.org/en/v0.8.19/contracts.html#function-overloading)

### PROOF OF CONCEPT
```solidity
moonwell/blob/main/src/core/Comptroller.sol
998:  function claimReward() public {
1006: function claimReward(address holder) public {
1015: function claimReward(address holder, MToken[] memory mTokens) public {
1028: function claimReward(address[] memory holders, MToken[] memory mTokens, bool borrowers, bool suppliers) public {
```
## [12] Through the scope is use uint instead of uint256
Some developers prefer to use uint256 because it is consistent with other uint data types, which also specify their size, and also because making the size of the data explicit reminds the developer and the reader how much data they've got to play with, which may help prevent or detect bugs. insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
// @audit Use uint256 instead of uint.
Comptroller.sol
62: uint internal constant closeFactorMinMantissa = 0.05e18;
65: uint internal constant closeFactorMaxMantissa = 0.9e18;
68: uint internal constant collateralFactorMaxMantissa = 0.9e18;
```
This issue is happen through the scope.

### Recommendation
It's best to use uint256. It brings about readability and consistency in your code, and it allows you to adhere to best practices in smart contracts.

## [13] Mappings do not follow the style of Solidity
The [solidity documentation](https://docs.soliditylang.org/en/v0.8.19/style-guide.html#mappings) strongly recommends the following: 
`In variable declarations, do not separate the keyword mapping from its type by a space. Do not separate any nested mapping keyword from its type by whitespace.`

### PROOF OF CONCEPT
```solidity
ChainlinkOracle.sol
23: mapping (address => uint256) internal prices;
27: mapping (bytes32 => AggregatorV3Interface) internal feeds;
```
## [14] Empty blocks should be removed or emit something 
In Solidity, any function defined in a non-abstract contract must have a complete implementation that specifies what to do when that function is called. 

```solidity
 constructor() {}
```
### Recommended Mitigation Steps
```solidity
MErc20Delegate.sol
// @audit Consider emit something or simply remove the code block.
15: constructor() {}
```
## [15] Use a single file for all system-wide constants
In all LPS implementations, there are many "constants" used in the system. It is recommended to store all constants in a single file (for example, Constants.sol) and use inheritance to access these values.

This helps improve readability and facilitates future maintenance. It also helps with any issues, as some of these hard-coded contracts are administrative contracts.

Constants.sol is used and imported in contracts that require access to these values.
This is just a suggestion, as the project currently has more than 11 files  that store constants.

## [16] We suggest to use named parameters for mapping type
Consider using named parameters in mappings (e.g. mapping(address account => uint256 balance)) to improve readability. This feature is present since Solidity 0.8.18.

```solidity
mapping(address account => uint256 balance); 
```
## [17] In certain functions of the main contract Comptroller.sol, we noticed bad practices
To improve readability and facilitates future maintenance we recommend The proper use of '_' as a function name prefix is a common pattern, especially for internal and private functions.

### PROOF OF CONCEPT
```solidity
Some examples in the Comptroller.sol contract.
121: function addToMarketInternal(MToken mToken, address borrower) internal returns (Error) {
263: function redeemAllowedInternal(address mToken, address redeemer, uint redeemTokens) internal view returns (uint) {
528: function getAccountLiquidityInternal(address account) internal view returns (Error, uint, uint) {
```
We addressed this issue by emphasizing it in the `Comptroller.sol` contract because it is the main contract, but this problem extends throughout the scope. It is easier for us, as auditors, to navigate and understand the functions we read when they adhere to the established Solidity standards, whether they are internal, private, public, etc.

The use of underscores is incorrectly applied in certain external functions, leading to confusion when trying to read and understand the flow of the code. 

An example of this issue can be observed in the following function:

```solidity
959: function _rescueFunds(address _tokenAddress, uint _amount) external {
```
### Recommendation
It is important to adhere to good Solidity coding practices to maintain code clarity and project maintainability in both the short and long term. Consistency in following these practices ensures that the code remains readable and understandable. This promotes better collaboration among developers and reduces the chances of introducing errors or confusion in the codebase.

## [18] In the names of input parameters for functions, we recommend following the Solidity standard.
We recommend using underscores in the names of function parameters to differentiate between local and global variables throughout the functions.

