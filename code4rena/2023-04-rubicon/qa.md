`Dear Rubicon v2 Team, as we have gone through each contract within the scope, we have noticed good practices that have been implemented. However, we have identified some inconsistencies that we recommend addressing. We understand that every team has a different level of good practices, but we believe that at least 90% of the recommendations in the following report should be applied for better gas efficiency, readability, and most importantly, safety.`

`Note: We have provided a description of the situation and recommendations to follow, including articles and resources we have created to help identify the problem and address it quickly, and to implement them in future projects.`

### Issues
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| S | Suggestions | Suggestion Details |

| Total Found Issues | 44 |
|:--:|:--:|

### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | Open TODOs | 12 |
| [L-02] | CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE | 1 |
| [L-03] | INITIALIZE FUNCTION CAN BE CALLED BY ANYBODY | 2 |
| [L-04] | ADD A TIMELOCK TO CRITICAL FUNCTIONS | 3 |
| [L-05] | IN THE EVENTS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE  | 3 |
| [L-06] | MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS | 3 |
| [L-07] | UPGRADEABLE CONTRACT IS MISSING A __GAP[50] STORAGE VARIABLE | 1 |
| [L-08] | ADD CONSTRUCTOR INITIALIZERS | 2 |
| [L-09] | THE CONTRACT IMPORTS A LIBRARY THAT IT DOES NOT USE BUT SHOULD | 1 |
| [L-10] | USE 'increaseAllowance' INSTEAD OF THE FUNCTION 'approve' | 1 |
| [L-11] | DID NOT APPROVE TO ZERO FIRST | 1 |
| [L-12] | INCONSISTENT SOLIDITY PRAGMA |  |
| [L-13] | PREVENT DIV BY 0 |  |
| [L-14] | MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN | 2 |

| Total Low Issues | 14 | Total Instances | 32 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | USE OF FLOATING PRAGMA | 1 |
| [N-02] | USE A MORE RECENT VERSION OF SOLIDITY  | 5 |
| [N-03] | CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES | ALL CONTRACTS |
| [N-04] | MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS  | 4 |
| [N-05] | NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS | 4 |
| [N-06] | USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONTRACTS/LIBRARY | 4 |
| [N-07] | LACK OF NATSPEC DOCUMENTATION | All Contracts |
| [N-08] | USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18) | 11 |
| [N-09] | CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS | 7 |
| [N-10] | REMOVE THE COMMENTED CODE FROM THE PROJECT | 2 |
| [N-11] | NATSPEC DONT COMPLY WITH SOLDITY STYLE GUIDE | 1 |
| [N-12] | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE | all contracts |
| [N-13] | NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES | all contracts |
| [N-14] | NEED FUZZING TEST | all contracts |
| [N-15] | SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE | 12 |
| [N-16] | USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED() | 6 |
| [N-17] | LONG REVERT STRINGS | 2 |
| [N-18] | MISSING ERROR MESSAGES IN REQUIRE STATEMENTS | 19 |
| [N-19] | ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS | 2 |
| [N-20] | FUNCTION OVERLOADING | 4 |
| [N-21] | USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED | 11 |
| [N-22] | EVENTS IS MISSING INDEXED FIELDS | 2 |
| [N-23] | TOKENS ACCIDENTALLY SENT TO THE CONTRACT CANNOT BE RECOVERED | 2 |
| [N-24] | CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING | 1 |

| Total Non-Critical Issues | 24 | Total Instances | 100 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | FUNCTION NAMING SUGGESTIONS | 2 |
| [R-02] | SOME NUMBER VALUES CAN BE REFACTORED WITH _ | 2 |

| Total Refactor Issues | 2 | Total Instances | 4 |
|:--:|:--:|:--:|--:|


### Suggestion Details 
| Count | Explanation | 
|:--:|:-------|
| [S-01] | WE SUGGEST USING THE OPENZEPPELIN SAFECAST LIBRARY |  |
| [S-02] | WE SUGGEST USING THE OPENZEPPELIN ADDRESS LIBRARY |  |
| [S-03] | WE SUGGEST USING THE BoringERC20 LIBRARY |  | 
| [S-04] | WE SUGGEST USING A MORE RECENT SOLIDITY PRAGMA TO TAKE ADVANTAGE |  | 

| Total Suggestions | 4 |
|:--:|:--:|

# Detailed Findings

## [L-01] Open TODOs
It is important to address any issue or problem related to code architecture, incentives, and error handling/reporting before deploying the code to the mainnet, even if there are sections of the code marked as "TODO".

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
/// @audit Remove: 'TODOs'
15: contract DSAuthEvents {
    /// event LogSetAuthority(address indexed authority); /// TODO: this event is not used in the contract, remove?
    event LogSetOwner(address indexed owner);
18: }
99: /// TODO: double check it is sound logic to kill this event
128: /// TODO: double check it is sound logic to kill this event
184: /// TODO: double check it is sound logic to kill this event
197: /// TODO: we will need to make sure this emit is included in any taker pay maker scenario
299: /// TODO: double check it is sound logic to kill this event
428: /// TODO: double check it is sound logic to kill this event
661: /// TODO: this event is not used in the contract, remove?
663: /// TODO: this event is not used in the contract, remove?
666: /// TODO: this event is not used in the contract, remove?
675: //buy enabled TODO: review this decision!

contracts/utilities/poolsUtility/Position.sol
/// @audit Remove: 'TODOs'
156: // TODO: definitely need to work on naming things
```

## [L-02] CRITICAL CHANGES SHOULD USE TWO-STEP PROCEDURE
Handle ownership transfers with two steps and two transactions. First, allow the current owner to propose a new owner address. Second, allow the proposed owner (and only the proposed owner) to accept ownership, and update the contract owner internally.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
25: function setOwner(address owner_) external auth {
        owner = owner_;
        emit LogSetOwner(owner);
    }
```
### MITIGATION
Implement zero address check and Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.

## [L-03] INITIALIZE FUNCTION CAN BE CALLED BY ANYBODY
`initialize function` can be called anybody when the contract is not initialized.

More importantly, if someone else runs this function, they will have full authority because of the owner state variable.

Here is a definition of `initialize function`.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
700: function initialize(address _feeTo) public {
        require(!initialized, "contract is already initialized");
        require(_feeTo != address(0));

        /// @notice The market fee recipient
        feeTo = _feeTo;

        owner = msg.sender;
        emit LogSetOwner(msg.sender);

        /// @notice The starting fee on taker trades in basis points
        feeBPS = 1;

        initialized = true;
        matchingEnabled = true;
        buyEnabled = true;
    }

contracts/BathHouseV2.sol
32: function initialize(address _comptroller, address _pAdmin) external {
        require(!initialized, "BathHouseV2 already initialized!");
        comptroller = Comptroller(_comptroller);
        admin = msg.sender;
        proxyAdmin = _pAdmin;

        initialized = true;
    }    
```
### MITIGATION
Add a control that makes init() only call the Deployer Contract or EOA:
```solidity
// @audit example of mitigation
if (msg.sender != DEPLOYER_ADDRESS) {
        revert NotDeployer();
}
```

## [L-04] ADD A TIMELOCK TO CRITICAL FUNCTIONS
It is a good practice to give time for users to react and adjust to critical changes. A timelock provides more guarantees and reduces the level of trust required, thus decreasing risk for users. It also indicates that the project is legitimate (less risk of a malicious owner making a sandwich attack on a user). Consider adding a timelock to the following functions:

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
25:   function setOwner
955:  function setMinSell

contracts/utilities/poolsUtility/Position.sol
226:  function increaseMargin
```

## [L-05] IN THE EVENTS, INCLUDE THE OLD AND NEW VALUES OF THE UPDATED PARAMETERS TO TRACK THE CHANGES MADE
The following functions RubiconMarket.setOwner, RubiconMarket.setMinSell, and Position.increaseMargin make critical changes and should include the `old and new values` of the updated parameters so that users can be aware of the changes made.

## [L-06] MISSING EVENTS FOR ONLY FUNCTIONS THAT CHANGE CRITICAL PARAMETERS
The afunctions that change critical parameters should emit events. Events allow capturing the changed parameters so that off-chain tools/interfaces can register such changes with timelocks that allow users to evaluate them and consider if they would like to engage/exit based on how they perceive the changes as affecting the trustworthiness of the protocol or profitability of the implemented financial services. The alternative of directly querying on-chain contract state for such changes is not considered practical for most users/usages.

missing events and timelocks do not promote transparency and if such changes immediately affect users' perception of fairness or trustworthiness, they could exit the protocol causing a reduction in liquidity which could negatively impact protocol TVL and reputation.


### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
1466: function setFeeBPS(uint256 _newFeeBPS) external auth returns (bool)
1471: function setMakerFee(uint256 _newMakerFee) external auth returns (bool) 
1476: function setFeeTo(address newFeeTo) external auth returns (bool)

// @audit the same issue in BathHouseV2 the contract in the L32
32:  function initialize(address _comptroller, address _pAdmin) external // @audit this function should have a event
```
### MITIGATION
Add Event-Emit, for example in these cases:
```diff
+ emit SomeEvent(oldFeeTo, newFeeTo);
```

## [L-07] UPGRADEABLE CONTRACT IS MISSING A __GAP[50] STORAGE VARIABLE
Reference: [Storage_gaps](https://docs.openzeppelin.com/contracts/4.x/upgradeable#storage_gaps)

You may notice that every contract includes a state variable named __gap. This is empty reserved space in storage that is put in place in Upgradeable contracts. It allows us to freely add new state variables in the future without compromising the storage compatibility with existing deployments.

A storage gap occurs when a contract is updated and state variables are added or removed. If the position of these variables changes in the new contract, the information that was already stored on the blockchain in the previous position may be exposed and manipulated by an attacker.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol

674: contract RubiconMarket is MatchingEvents, ExpiringMarket, DSNote{
    function initialize(address _feeTo) public {}
}

contracts/BathHouseV2.sol
12: contract BathHouseV2{
    function initialize(address _comptroller, address _pAdmin) external {}
}
```
### MITIGATION
Please remember that smart contracts are immutable. You are following an upgradable contract architecture, so it is essential to implement the GAP mechanism in these contracts and maintain order for future upgrades to avoid compromising users' funds and introducing vulnerabilities in the future.

Consider defining an appropriate storage gap in each upgradeable parent contract at the end of all the storage variable definitions as follows:

```solidity
uint256[50] __gap; // gap to reserve storage in the contract for future variable additions
```

## [L-08] ADD CONSTRUCTOR INITIALIZERS
According to [OpenZeppelin's (OZ) recommendation](https://forum.openzeppelin.com/t/uupsupgradeable-vulnerability-post-mortem/15680/6), it is recommended to make it impossible for anyone to execute the "initialize" function on an implementation contract by adding an empty constructor with the "initializer" modifier. This way, the implementation contract is automatically initialized upon deployment.

It's worth nothing that this approach has also been incorporated into the [OZ Wizard](https://wizard.openzeppelin.com/) since the discovery of the UUPS vulnerability. In fact, the code generated by [Wizard 19](https://wizard.openzeppelin.com/) has been modified to include a constructor that automatically initializes the implementation upon deployment.

This practice also helps prevent possible attacks in which someone tries to perform an initialization transaction before the creator of the contract, which could compromise the security of the contract.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
674: contract RubiconMarket is MatchingEvents, ExpiringMarket, DSNote{
    ..........................................................
    700: function initialize(address _feeTo) public{}
}

contracts/BathHouseV2.sol
12: contract BathHouseV2
    .............................
    32: function initialize(address _comptroller, address _pAdmin) external {
}
```

## [L-09] THE CONTRACT IMPORT A LIBRARY THAT IT DOES NOT USE BUT SHOULD
The V2Migrator contract, on line 4, imports the SafeERC20 library from OpenZeppelin but does not use it. According to a comment on line 29, every pool in V1 that wants to migrate to V2 should be able to do so with the `migrate` function. In the same comment, they refer to USDC, which, for example, does not follow the ERC20 standard, and many other tokens don't either. You guys should implement the SafeERC20 library from OZ in this contract

### PROOF OF CONCEPT
```solidity
// @audit implement this library in the migrate function
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
```
### MITIGATION
implement the `SafeERC20` library from OpenZepplin in the `V2Migrator.sol` contract.

## [L-10] USE 'increaseAllowance' INSTEAD OF THE FUNCTION 'approve'
Use `increaseAllowance` instead of the `approve` function since this is an alternative to approve that can be used as a mitigation for problems described in `IERC20-approve`.

### PROOF OF CONCEPT
```solidity
/// OpenZeppelin docs
/** 
* This is an alternative to {approve} that can be used as a mitigation for
* problems described in {IERC20-approve}.
*
*/
contracts/V2Migrator.sol
// @audit use `increaseAllowance`
53: underlying.approve(bathTokenV2, amountWithdrawn);
```
### MITIGATION
Use `increaseAllowance` instead of the `approve` function from OpenZepplin in the `V2Migrator.sol` contract.


## [L-11] DID NOT APPROVE TO ZERO FIRST
Some tokens like `USDT` do not work when changing the allowance from an existing non-zero allowance value.
They must first be approved by zero and then the actual allowance must be approved.

### PROOF OF CONCEPT
```solidity
contracts/V2Migrator.sol
53: underlying.approve(bathTokenV2, amountWithdrawn);
```

### IMPACT
A number of features within the vaults will not work if the approve function reverts.

### MITIGATION
It is recommended to set the `allowance to zero before increasing` the allowance and use `safeIncreaseAllowance`.

## [L-12] INCONSISTENT SOLIDITY PRAGMA
The source files have different solidity compiler ranges referenced. This leads to potential security flaws between deployed contracts depending on the compiler version chosen for any particular file. It also greatly increases the cost of maintenance as different compiler versions have different semantics and behavior.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
2: pragma solidity ^0.8.9;

contracts/BathHouseV2.sol
2: pragma solidity 0.8.17; // @audit this an another three contract have this pragma version 

contracts/periphery/BathBuddy.sol
2: pragma solidity ^0.8.0;
```
### MITIGATION
We recommend to fix a definite compiler range that is consistent between contracts and upgrade any affected contracts to conform to the specified compiler.

## [L-13] PREVENT DIV BY 0
On several locations in the code precautions are not being taken for not dividing by 0, this will revert the code. These functions can be calledd with 0 value in the input, this value is not checked for being bigger than 0, that means in some scenarios this can potentially trigger a division by zero.

### PROOF OF CONCEPT
```solidity
contracts/utilities/FeeWrapper.sol
41: fees[i] = (tokenAmounts[i] * feeValue) / feeType;
```
### MITIGATION
Recommend making sure division by 0 won’t occur by checking the variables beforehand and handling this edge case.

## [L-14] MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an `EMERGENCY STOP (CIRCUIT BREAKER) PATTERN`. Use the follow example to implement it into in the `BathHouseV2.sol`, `claimRewards` function and `RubiconMarket.sol` in the following functions `sellAllAmount`, `buyAllAmount`.

[Emergency Stop Pattern Example](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)


## [N-01] USE OF FLOATING PRAGMA 
It is advisable to avoid using floating pragmas in contracts that are not libraries.

While floating pragmas make sense in the context of libraries, allowing them to be included in multiple different versions of applications, they can pose a security risk in application implementations.

There is a possibility that a vulnerable compiler version may be accidentally selected or security tools may fallback to an older compiler version, resulting in verification of a different EVM compilation that is ultimately deployed on the blockchain.

Therefore, it is recommended to pin to a specific compiler version to ensure the security of the contract.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
2: pragma solidity ^0.8.9;
```
### MITIGATION
We recommend following your own best practice patterns in `RubiconMarket.sol`, as they do in `BathHouseV2.sol` and the rest of contracts, for interfaces and contracts.

```diff
contracts/RubiconMarket.sol
- 2: pragma solidity ^0.8.9;
+ 2: pragma solidity 0.8.9;
```

## [N-02] USE A MORE RECENT VERSION OF SOLIDITY
It is recommended to update the version of Solidity being used, as an old version is currently in use. It is advisable to consider using the latest version, `0.8.19`. You can check the improvements and bug fixes offered by the new versions [here](https://github.com/ethereum/solidity/blob/develop/Changelog.md)

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
2: pragma solidity ^0.8.9;

contracts/BathHouseV2.sol
2: pragma solidity 0.8.17;

contracts/V2Migrator.sol
2: pragma solidity 0.8.17;

contracts/utilities/poolsUtility/Position.sol
2: pragma solidity 0.8.17;

contracts/utilities/FeeWrapper.sol
2: pragma solidity 0.8.17;
```

### RECOMMENDATION
```diff
contracts/RubiconMarket.sol
- 2: pragma solidity ^0.8.9;
+ 2: pragma solidity 0.8.19;

contracts/BathHouseV2.sol
- 2: pragma solidity 0.8.17;
+ 2: pragma solidity 0.8.19;
```

## [N-03] CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES
To improve code readability, it is recommended to use specific names when importing instead of regular ones. 

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
12: import "@openzeppelin/contracts/utils/StorageSlot.sol";

contracts/BathHouseV2.sol
4: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
5: import "./compound-v2-fork/InterestRateModel.sol";
6: import "./compound-v2-fork/CErc20Delegator.sol";
7: import "./compound-v2-fork/Comptroller.sol";
8: import "./compound-v2-fork/Unitroller.sol";
9: import "./periphery/BathBuddy.sol";

contracts/V2Migrator.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
6: import "./compound-v2-fork/CTokenInterfaces.sol";

contracts/utilities/poolsUtility/Position.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
6: import "@openzeppelin/contracts/utils/math/SafeMath.sol";
7: import "@openzeppelin/contracts/access/Ownable.sol";
8: import "../../compound-v2-fork/Comptroller.sol";
9: import "../../compound-v2-fork/PriceOracle.sol";
10: import "../../BathHouseV2.sol";
11: import "../../RubiconMarket.sol";

contracts/utilities/FeeWrapper.sol
4: import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";
5: import "./RubiconRouter.sol";
```
### RECOMMENDATION
```diff
-11: import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
+11: import {ERC20} from "@openzeppelin/contracts/token/ERC20/ERC20.sol";
```

## [N-04] MANDATORY CHECKS FOR EXTRA SAFETY IN THE SETTERS
In the folowing functions below, there are some checks that can be made in order to achieve more safe and efficient code.

Address zero check can be added in the function offer, RubiconMarket contract to ensure the address owner and recipient aren't address(0).

### PROOF OF CONCEPT
```solidity
25:     function setOwner(address owner_) external auth {
            owner = owner_; // @audit Check owner_ it's not address 0
            emit LogSetOwner(owner);
        }
        
511:    function offer(
            uint256 pay_amt,
            ERC20 pay_gem,
            uint256 buy_amt,
            ERC20 buy_gem,
            address owner,  // @audit Check that it's not address 0
            address recipient // @audit Check that it's not address 0
        )
         
802:    function offer(
            uint256 pay_amt, //maker (ask) sell how much
            ERC20 pay_gem, //maker (ask) sell which token
            uint256 buy_amt, //maker (ask) buy how much
            ERC20 buy_gem, //maker (ask) buy which token
            uint256 pos, //position to insert offer, 0 should be used if unknown
            address owner,  // @audit Check that it's not address 0
            address recipient // @audit Check that it's not address 0
        )

// @audit also the same issue in the BathHouseV2 contract in the line 32
32: function initialize(address _comptroller, address _pAdmin) external                
```

## [N-05] NATSPEC COMMENTS SHOULD BE INCREASED IN CONTRACTS
It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). It is clearly stated in the [Solidity official documentation](https://docs.soliditylang.org/en/v0.8.15/natspec-format.html). In complex projects such as Defi, the interpretation of all functions and their arguments and returns is important for code readability and auditability. 

### MITIGATION
NatSpec comments should be increased in contracts

## [N-06] USE A SINGLE FILE FOR ALL SYSTEM-WIDE CONTRACTS/LIBRARY
In the main contract `RubiconMarket.sol`, there are many `contracts/libraries` used in the system. It is recommended to put the most used ones in one file (for example `SafeMath.sol`, `DSAuth.sol`, `Events.sol`) and use inheritance to access these values.

This helps with readability and easier maintenance for future changes. It also helps with any issues, as some of these hard-coded contracts are admin contracts.

`SafeMath.sol`, `DSAuth.sol`, `Events.sol` Use and import these files in contracts that require access to these values. This is just a suggestion, in some use cases this may result in higher gas usage in the distribution.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Create a specific file for these cases and use inheritance.
    
 /// @notice DSAuth events for authentication schema
 15: contract DSAuthEvents {}

 /// @notice DSAuth library for setting owner of the contract
 /// @dev Provides the auth modifier for authenticated function calls
 22:contract DSAuth is DSAuthEvents {}
 
 /// @notice DSMath library for safe math without integer overflow/underflow
 44: contract DSMath {}

 /// @notice Events contract for logging trade activity on Rubicon Market
 /// @dev Provides the key event logs that are used in all core functionality of exchanging on the Rubicon Market
 94: contract EventfulMarket {}
```
### RECOMMENDATION
We strongly recommend implementing each contract in a separate file. This will help you avoid errors and confusion in your project, and maintain a more organized and readable code structure. Following these good programming and organizational practices will help you build a solid and scalable project, which will be easier to maintain and update in the future.

## [N-07] LACK OF NATSPEC DOCUMENTATION
During the smart contract audit, the absence of Natspec documentation has been detected in several critical functions. It is important to note that this lack of description makes it difficult to understand the functionality of the code and therefore may increase the risk of errors in the future. It is strongly recommended to include detailed documentation in all contract functions to ensure long-term readability and maintainability.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Make sure to implement Natspec documentation in critical functions.

    function kill(bytes32 id) external virtual {
        require(cancel(uint256(id)));
    }

    function make(
        ERC20 pay_gem,
        ERC20 buy_gem,
        uint128 pay_amt,
        uint128 buy_amt
    ) external virtual returns (bytes32 id) {
        return
            bytes32(
                offer(
                    pay_amt,
                    pay_gem,
                    buy_amt,
                    buy_gem,
                    msg.sender,
                    msg.sender
                )
            );
    }

    function take(bytes32 id, uint128 maxTakeAmount) external virtual {
        require(buy(uint256(id), maxTakeAmount));
    }

    function _next_id() internal returns (uint256) {
        last_offer_id++;
        return last_offer_id;
    }

    function calcAmountAfterFee(
        uint256 amount
    ) public view returns (uint256 _amount) {
        require(amount > 0);
        _amount = amount;
        _amount -= mul(amount, feeBPS) / 100_000;

        if (makerFee() > 0) {
            _amount -= mul(amount, makerFee()) / 100_000;
        }
    }

    function make(
        ERC20 pay_gem,
        ERC20 buy_gem,
        uint128 pay_amt,
        uint128 buy_amt
    ) public override returns (bytes32) {
        return
            bytes32(
                offer(
                    pay_amt,
                    pay_gem,
                    buy_amt,
                    buy_gem,
                    msg.sender,
                    msg.sender
                )
            );
    }

    function take(bytes32 id, uint128 maxTakeAmount) public override {
        require(buy(uint256(id), maxTakeAmount));
    }

    function kill(bytes32 id) external override {
        require(cancel(uint256(id)));
    }
```
### RECOMMENDATION
Implement the [NatSpec Format](https://docs.soliditylang.org/en/v0.8.19/natspec-format.html) of Solidity.
```solidity
/// @notice Calculate tree age in years, rounded up, for live trees
/// @dev The Alexandr N. Tetearing algorithm could increase precision
/// @param rings The number of rings from dendrochronological sample
/// @return Age in years, rounded up for partial years
```

## [N-08] USE SCIENTIFIC NOTATION (E.G. 1E18) RATHER THAN EXPONENTIATION (E.G. 10**18)
Scientific notation `(e.g. 1E18)` is clearer and easier to read than exponentiation `(e.g. 10**18)`. Additionally, scientific notation is widely used in the Ethereum development community and is considered a best practice for making code more readable and understandable.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Refactorize scientific notation
    74: uint256 constant WAD = 10 ** 18;
    75: uint256 constant RAY = 10 ** 27; 
    1059: / 10 ** 9;
    1099: mul(buy_amt, 10 ** 9)
    1101: / 10 ** 9
    1142: mul(pay_amt, 10 ** 9)
    1144: / 10 ** 9
    1175: mul(buy_amt, 10 ** 9)
    1177: / 10 ** 9

contracts/utilities/poolsUtility/Position.sol
317: _max = (_liq.mul(10 ** 18)).div(_price);
331: .div(10 ** 18);
```
### RECOMMENDATION
```diff
-74: uint256 constant WAD = 10 ** 18;
+74: uint256 constant WAD = 1e18; 
```

## [N-09] CONSTANTS SHOULD BE DEFINED RATHER THAN USING MAGIC NUMBERS
The `RubiconMarket.sol` contract implements `10 ** 9` multiple times. We recommend storing this value in a constant variable, as it can benefit from using readable constants instead of hex/numeric literals.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit 10 ** 9
    1059: / 10 ** 9;
    1099: mul(buy_amt, 10 ** 9)
    1101: / 10 ** 9
    1142: mul(pay_amt, 10 ** 9)
    1144: / 10 ** 9
    1175: mul(buy_amt, 10 ** 9)
    1177: / 10 ** 9
```
### RECOMMENDATION
```diff
-1099: mul(buy_amt, 10 ** 9)
+1099: uint public constant VALUE = 1e9;
+1099: mul(buy_amt, VALUE);
```

## [N-10] REMOVE THE COMMENTED CODE FROM THE PROJECT
Smart contracts often contain numerous implemented code fragments. It is advisable to consider removing unnecessary code or implementing it correctly before deploying to the main network. The presence of inconsistent code makes the project difficult to read and can lead to serious errors. Therefore, it is recommended to carefully review the code and make necessary adjustments to ensure quality and security before deploying to the main network.

### PROOF OF CONCEPT
// @audit Remove commented code 
[File RubiconMarket.sol](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol)

## [N-11] NATSPEC DONT COMPLY WITH SOLDITY STYLE GUIDE
While auditing, it was noticed that the recommended Solidity standards for Natspec were not being followed correctly.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit so many instances with these issues
250: // /// @notice Modifier that checks the user to make sure they own the offer and its valid before they attempt to cancel it
666: /// event LogInsert(address keeper, uint256 id);
718: // // After close, anyone can cancel an offer
```
### MITIGATION
Implement the [NatSpec Format](https://docs.soliditylang.org/en/v0.8.19/natspec-format.html) of Solidity.

## [N-12] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE
According to the Solidity style guide, functions should be laid out in the following order: 

- constructor() 
- receive() 
- fallback() 
- external
- public 
- internal 
- private

[Solidity Order Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

Functions should be grouped according to their visibility and ordered: `within a grouping, place the view and pure functions last`

## [N-13] NON-LIBRARY/INTERFACE FILES SHOULD USE FIXED COMPILER VERSIONS, NOT FLOATING ONES
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

### MITIGATION
```diff
staking/NeoTokyoStaker.sol#L1517-L1521
- pragma solidity ^0.8.9;
+ pragma solidity 0.8.9;
```

## [N-14] NEED FUZZING TEST
We recommend the use of fuzzing tests, especially in finance oriented protocols, due to the complexity and risk involved in handling large amounts of money in these smart contracts. Finance oriented contracts are critical in terms of security and accuracy, as any errors or vulnerabilities could be exploited by malicious attackers to steal funds or cause significant damage. 

Fuzzing tests are an important tool for identifying possible vulnerabilities in the code through the automatic and random generation of input data in the contract, which can help avoid costly errors in production. 

### RECOMMENDATION 
Use should fuzzing test like Echidna or [Foundry](https://book.getfoundry.sh/forge/fuzz-testing).

## [N-15] SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE
Short-circuiting is a solidity contract development model that uses `OR/AND` logic to sequence different cost operations. It puts low gas cost operations in the front and high gas cost operations in the back, so that if the front is low If the cost operation is feasible, you can skip (short-circuit) the subsequent high-cost Ethereum virtual machine operation.

```solidity
//f(x) is a low gas cost operation 
//g(y) is a high gas cost operation 

//Sort operations with different gas costs as follows 
f(x) || g(y) 
f(x) && g(y)
```

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
325:    if (
            quantity == 0 ||
            spend == 0 ||
            quantity > _offer.pay_amt ||
            spend > _offer.buy_amt
        )
349:    if (_offer.owner == address(0) && getRecipient(id) != address(0))
1197:   if (
            isActive(id) &&
            offers[id].pay_amt < _dust[address(offers[id].pay_gem)]
        )
1217:   while (top != 0 && _isPricedLtOrEq(id, top))
1229:   while (pos != 0 && !isActive(pos))
1244:   while (pos != 0 && _isPricedLtOrEq(id, pos))
1252:   while (pos != 0 && !_isPricedLtOrEq(id, pos))                
1319:   if (t_pay_amt == 0 || t_buy_amt == 0)
1324:   if (
            t_buy_amt > 0 &&
            t_pay_amt > 0 &&
            t_pay_amt >= _dust[address(t_pay_gem)]
        )
1452:   while (uid > 0 && uid != id)  

contracts/utilities/poolsUtility/Position.sol

176:    if (i.add(1) == vars.limit && vars.lastBorrow != 0)
391:    if (
            IERC20(_bathTokenAsset).balanceOf(address(this)) == 0 &&
            borrowBalance(_bathTokenAsset) == 0
        )
```

## [N-16] USE OF BYTES.CONCAT() INSTEAD OF ABI.ENCODEPACKED()
Since version 0.8.4 for appending bytes, [bytes.concat()](https://docs.soliditylang.org/en/v0.8.19/types.html#bytes-concat) can be used instead of abi.encodePacked().

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
369:  keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem))
400:  keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem))
423:  keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem))
442:  keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem))
476:  keccak256(abi.encodePacked(_offer.pay_gem, _offer.buy_gem))
555:  keccak256(abi.encodePacked(pay_gem, buy_gem))
```

## [N-17] LONG REVERT STRINGS
Shortening revert strings to fit in 32 bytes will decrease gas costs for deployment and gas costs when the revert condition has been met.

### PROOF OF CONCEPT

```solidity
contracts/RubiconMarket.sol

374:    require(
            _offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend),
            "_offer.buy_gem.transferFrom(msg.sender, _offer.recipient, spend) failed - check that you can pay the fee"
        );

721:    require(
            isClosed() ||
                msg.sender == getOwner(id) ||
                id == dustId ||
                (msg.sender == getRecipient(id) && getOwner(id) == address(0)),
            "Offer can not be cancelled because user is not owner, and market is open, and offer sells required amount of tokens."
        );        
```

### MITIGATION
Consider using `Custom Errors` as they are more gas efficient while allowing developers to describe the error in detail using NatSpec. 

## [N-18] MISSING ERROR MESSAGES IN REQUIRE STATEMENTS
Require statements should have descriptive strings to describe why the revert occurs.

```solidity
contracts/RubiconMarket.sol
581:  require(amount > 0);
702:  require(_feeTo != address(0));
938:  require(!locked)
1034: require(!locked);
1075: require(!locked)
1136: require(offerId != 0)
1184: require(buyEnabled)
1209: require(id > 0)
1226: require(id > 0)
1410: require(_span[pay_gem][buy_gem] > 0)
1477: require(newFeeTo != address(0))
```
- (Another lines with this in RubiconMarket.sol)[https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/RubiconMarket.sol#L519-L525]

## [N-19] ASSEMBLY CODES SPECIFIC - SHOULD HAVE COMMENTS
Since this is a low level language that is more difficult to parse by readers, include extensive documentation, comments on the rationale behind its use, clearly explaining what each assembly instruction does

This will make it easier for users to trust the code, for reviewers to validate the code, and for developers to build on or update the code.

Note that using Aseembly removes several important security features of Solidity, which can make the code more insecure and more error-prone.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
648:    assembly {
            foo := calldataload(4)
            bar := calldataload(36)
            wad := callvalue()
        }

contracts/utilities/poolsUtility/Position.sol
367:    assembly {
            switch _initBathTokenAmount
            case 0 {
                _bathTokenAmount := _currentBathTokenAmount
            }
            default {
                _bathTokenAmount := sub(
                    _currentBathTokenAmount,
                    _initBathTokenAmount
                )
            }
        }        
```

## [N-20] FUNCTION OVERLOADING
Having multiple functions with the same name in a smart contract can be dangerous or not a good practice for several reasons:

- Confusion: If there are several functions with the same name, it can be confusing for developers and users who are interacting with the smart contract. This can lead to errors and misunderstandings in the use of the contract.

- Security vulnerabilities: If multiple functions are defined with the same name, attackers can attempt to exploit this vulnerability to access or modify data or functionalities of the smart contract.

- Network overload: If there are multiple functions with the same name, there may be an impact on the efficiency and speed of the contract, as the network may be confused in trying to determine which function should be executed.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Refactorize
 674: contract RubiconMarket is MatchingEvents, ExpiringMarket, DSNote {
    ...........................................
    function offer(
        uint256 pay_amt, //maker (ask) sell how much
        ERC20 pay_gem, //maker (ask) sell which token
        uint256 buy_amt, //maker (ask) buy how much
        ERC20 buy_gem //maker (ask) buy which token
    ) external can_offer returns (uint256) {
        return
            offer(
                pay_amt,
                pay_gem,
                buy_amt,
                buy_gem,
                0,
                true,
                msg.sender,
                msg.sender
            );
    }

    function offer(
        uint256 pay_amt, //maker (ask) sell how much
        ERC20 pay_gem, //maker (ask) sell which token
        uint256 buy_amt, //maker (ask) buy how much
        ERC20 buy_gem, //maker (ask) buy which token
        uint pos, //position to insert offer, 0 should be used if unknown
        bool rounding
    ) external can_offer returns (uint256) {
        return
            offer(
                pay_amt,
                pay_gem,
                buy_amt,
                buy_gem,
                pos,
                rounding,
                msg.sender,
                msg.sender
            );
    }

    // Make a new offer. Takes funds from the caller into market escrow.
    function offer(
        uint256 pay_amt, //maker (ask) sell how much
        ERC20 pay_gem, //maker (ask) sell which token
        uint256 buy_amt, //maker (ask) buy how much
        ERC20 buy_gem, //maker (ask) buy which token
        uint256 pos, //position to insert offer, 0 should be used if unknown
        address owner,
        address recipient
    ) external can_offer returns (uint256) {
        return
            offer(
                pay_amt,
                pay_gem,
                buy_amt,
                buy_gem,
                pos,
                true,
                owner,
                recipient
            );
    }

    function offer(
        uint256 pay_amt, //maker (ask) sell how much
        ERC20 pay_gem, //maker (ask) sell which token
        uint256 buy_amt, //maker (ask) buy how much
        ERC20 buy_gem, //maker (ask) buy which token
        uint256 pos, //position to insert offer, 0 should be used if unknown
        bool matching, //match "close enough" orders?
        address owner, // owner of the offer
        address recipient // recipient of the offer's fill
    ) public can_offer returns (uint256) {
        require(!locked, "Reentrancy attempt");
        require(_dust[address(pay_gem)] <= pay_amt);

        /// @dev currently matching is perma-enabled
        // if (matchingEnabled) {
        return
            _matcho(
                pay_amt,
                pay_gem,
                buy_amt,
                buy_gem,
                pos,
                matching,
                owner,
                recipient
            );
        // }
        // return super.offer(pay_amt, pay_gem, buy_amt, buy_gem);
    }
}
```
### RECOMMENDATION
To avoid problems related to function overloading in smart contracts, it is recommended to follow best programming practices. 

Some recommendations are:
- Name functions in a descriptive and unique way to avoid function overloading.
- Use input arguments with different data types to distinguish functions.
- Avoid unnecessary function overloading, i.e., define additional functions only if necessary and not just for convenience or ease of programming.
- Clearly document the functions and their uses to avoid confusion.

By following these recommendations, developers can avoid function overloading and create smarter and more secure contracts that are easier to use.

## [N-21] USING WHILE FOR UNBOUNDED LOOPS ISN'T RECOMMENDED
Improve the efficiency and stability of your code by avoiding unbounded `while loops`. Instead of using a `while loop` for unbounded iterations, it is recommended to use other loop structures like `for` that have a clearer structure and can provide better control flow for the loop. Don't write loops that are unbounded as this can hit the gas limit, causing your transaction to fail. For the reason above, while and do-while loops are rarely used.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Avoid unbounded loops
    1037: while (pay_amt > 0) 
    1077: while (buy_amt > 0)
    1130: while (pay_amt > offers[offerId].buy_amt)
    1163: while (buy_amt > offers[offerId].pay_amt)
    1217: while (top != 0 && _isPricedLtOrEq(id, top))
    1229: while (pos != 0 && !isActive(pos))
    1244: while (pos != 0 && _isPricedLtOrEq(id, pos))
    1252: while (pos != 0 && !_isPricedLtOrEq(id, pos))
    1289: while (_best[address(t_buy_gem)][address(t_pay_gem)] > 0)
    1452: while (uid > 0 && uid != id)

    contracts/utilities/poolsUtility/Position.sol
    541: while (_assetAmount <= _desiredAmount) 
```
### RECOMMENDATION
- Avoid unbounded loops: As mentioned before, it is important to avoid unbounded while loops in your smart contracts. If you need to make a loop, make sure that the number of iterations is limited and known in advance.

## [N-22]  EVENTS IS MISSING INDEXED FIELDS
Index event fields make the field more quickly accessible to off-chain. Each event should use three [indexed](https://docs.soliditylang.org/en/v0.8.0/contracts.html#events) fields if there are three or more fields.

### PROOF OF CONCEPT

```solidity
contracts/BathHouseV2.sol
// @audit missing indexed fields to track on chain the changes
23:  event BathTokenCreated(address bathToken, address underlying);
24:  event BuddySpawned(address bathToken, address bathBuddy);
```

## [N-23] TOKENS ACCIDENTALLY SENT TO THE CONTRACT CANNOT BE RECOVERED
It can't be recovered if the tokens accidentally arrive at the V2Migrator contract address, in the line 65,it is noted that `BATH TOKENS V2` may get stuck, but no token recovery implementation is implemented if this happens so we recommend adding a recovery code to this contract.

### MITIGATION
Add this code:
```solidity
 /**
  * @notice Sends ERC20 tokens trapped in contract to external address
  * @dev Onlyowner is allowed to make this function call
  * @param account is the receiving address
  * @param externalToken is the token being sent
  * @param amount is the quantity being sent
  * @return boolean value indicating whether the operation succeeded.
  *
 */
function rescueERC20(address account, address externalToken, uint256 amount) public onlyOwner returns (bool) {
    IERC20(externalToken).transfer(account, amount);
    return true;
}
```

## [N-24] CONTRACT DOES NOT FOLLOW THE SOLIDITY STYLE GUIDE'S SUGGESTED LAYOUT ORDERING
The [style guide](https://docs.soliditylang.org/en/v0.8.16/style-guide.html#order-of-layout) says that, within a contract, the ordering should be: 
- 1. Type declarations 
- 2. State variables,
- 3. Events 
- 4. Modifiers 
- 5. Functions 
but the contract(s) below do not follow this ordering.

- [BathBuddy.sol](https://github.com/code-423n4/2023-04-rubicon/blob/main/contracts/periphery/BathBuddy.sol)

## [R-01] FUNCTION NAMING SUGGESTIONS
Proper use of _ as a function name prefix and a common pattern is to prefix internal and private function names with _. This pattern is correctly applied in all contracts, however there are some inconsistencies in just these contracts.

### PROOF OF CONCEPT
```solidity
contracts/RubiconMarket.sol
// @audit Refactorize
35: function isAuthorized(address src) internal view returns (bool) {

contracts/utilities/poolsUtility/Position.sol
125: function openPosition(
        address asset,
        address quote,
        uint256 initMargin,
        uint256 leverage
    ) internal returns (bool OK) {
```
### RECOMMENDATION
```diff
-35: function isAuthorized(address src) internal view returns (bool)
+35: function _isAuthorized(address src) internal view returns (bool) 
```

## [R-02] SOME NUMBER VALUES CAN BE REFACTORED WITH _
Consider using underscores for number values to improve readability.

```solidity
contracts/utilities/poolsUtility/Position.sol
481: uint256 _fee = _minFill.mul(_feeBPS).div(10000);
490:  _fee = _payAmount.mul(_feeBPS).div(10000);
```

## [S-01] WE SUGGEST USING THE OPENZEPPELIN SAFECAST LIBRARY
We have noticed that the contract `RubiconMarket.sol` implements many type conversions from uint256 to uint128. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.

```solidity
openzeppelin-contracts/blob/master/contracts/utils/math/SafeCast.sol#L290
    /**
     * @dev Returns the downcasted uint128 from uint256, reverting on
     * overflow (when the input is greater than largest uint128).
     *
     * Counterpart to Solidity's `uint128` operator.
     *
     * Requirements:
     *
     * - input must fit into 128 bits
     *
     * _Available since v2.5._
     */
    function toUint128(uint256 value) internal pure returns (uint128) {
        require(value <= type(uint128).max, "SafeCast: value doesn't fit in 128 bits");
        return uint128(value);
    }
```

## [S-02] WE SUGGEST USING THE OPENZEPPELIN ADDRESS LIBRARY
In the `FeeWrapper.sol` contract, the low-level call() function is used for certain operations. However, it is recommended to use the `Address.sol` library from `OpenZeppelin` to implement these operations in a secure manner and take advantage of the gas optimizations and good practices offered by that library.

```solidity
openzeppelin-contracts/blob/master/contracts/utils/Address.sol
    /**
     * @dev Replacement for Solidity's `transfer`: sends `amount` wei to
     * `recipient`, forwarding all available gas and reverting on errors.
     *
     * https://eips.ethereum.org/EIPS/eip-1884[EIP1884] increases the gas cost
     * of certain opcodes, possibly making contracts go over the 2300 gas limit
     * imposed by `transfer`, making them unable to receive funds via
     * `transfer`. {sendValue} removes this limitation.
     *
     * https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/[Learn more].
     *
     * IMPORTANT: because control is transferred to `recipient`, care must be
     * taken to not create reentrancy vulnerabilities. Consider using
     * {ReentrancyGuard} or the
     * https://solidity.readthedocs.io/en/v0.5.11/security-considerations.html#use-the-checks-effects-interactions-pattern[checks-effects-interactions pattern].
     */
    function sendValue(address payable recipient, uint256 amount) internal {
        require(address(this).balance >= amount, "Address: insufficient balance");

        (bool success, ) = recipient.call{value: amount}("");
        require(success, "Address: unable to send value, recipient may have reverted");
    }
```

## [S-03] WE SUGGEST USING THE BoringERC20 LIBRARY
We suggest implementing best practices for SYMBOL, DECIMALS & NAME on lines `L146-147` and `L148` in the `BathHouseV2` contract . Therefore, we recommend using the following functions from this library: 
[Library BoringERC20](https://github.com/boringcrypto/BoringSolidity/blob/ccb743d4c3363ca37491b87c6c9b24b1f5fa25dc/contracts/libraries/BoringERC20.sol#L33-L55).

## [S-04] WE SUGGEST USING A MORE RECENT SOLIDITY PRAGMA TO TAKE ADVANTAGE
We recommend using a more recent version of Solidity, such as 0.8.18, to take advantage of the latest improvements and features, including better code readability in the case of mappings.

You guys have 19 mappings in the scope.

For example: 
```solidity
mapping(address account => uint256 balance)
```