### Issues 
| Letter | Name | Description |
|:--:|:-------:|:-------:|
| L  | Low risk | Potential risk |
| NC |  Non-critical | Non risky findings |
| R  | Refactor | Changing the code |
| O | Ordinary | Often found issues |
| S | Suggestions | Suggestion Details |

| Total Found Issues | 30 |
|:--:|:--:|


### Low Risk
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [L-01] | USE BLOCK.NUMBER INSTEAD OF BLOCK.TIMESTAMP| 5 |
| [L-02] | IMMUTABLE ADDRESSES LACK ZERO-ADDRESS CHECK | 1 |
| [L-03] | USE OF FLOATING PRAGMA | 2 |
| [L-04] | MIXED COMPILER VERSIONS | 2 |
| [L-05] | OWNER CAN RENOUNCE OWNERSHIP | 2 |

| Total Low Risk Issues | 5 | Total Instances | 12 |
|:--:|:--:|:--:|--:|

### Non-Critical
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [N-01] | SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE | 6 |
| [N-02] | MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN  | 4 |
| [N-03] | TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY |  |
| [N-04] | FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE | 2 |
| [N-05] | TOKENS ACCIDENTALLY SENT TO THE CONTRACT CANNOT BE RECOVERED |  |
| [N-06] | CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES | ALL CONTRACTS |
| [N-07] | INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS | 1 |
| [N-08] | LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR | 3 |
| [N-09] | MISSING NATSPEC | 5 |
| [N-10] | DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES | 2 |
| [N-11] | INSUFFICIENT COVERAGE | 2 |
| [N-12] | ADD NATSPEC MAPPING COMMENT | 2 |
| [N-13] | USE SMTChecker |  |
| [N-14] | NATSPEC IS INCOMPLETE | 1 |


| Total Non-Critical Issues | 14 | Total Instances | 28 |
|:--:|:--:|:--:|--:|

### Refactor Issues 
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [R-01] | "GET" IS TYPICALLY RESERVED FOR (VIEW/PURE) FUNCTIONS | 2 |
| [R-02] | SHORTHAND WAY TO WRITE IF/ELSE STATEMENT | 42 |
| [R-03] | USE REQUIRE INSTEAD OF ASSERT | 1 |
| [R-04] | USING CALLDATA INSTEAD OF MEMORY | 2 |
| [R-05] | CACHE THE MAPPING VALUES RATHER THAN FETCH IT EVERY TIME | 17 |

| Total Refactor Issues | 5 | Total Instances | 64 |
|:--:|:--:|:--:|--:|

### Ordinary Issues
| Count | Explanation | Instances |
|:--:|:-------|:--:|
| [O-01] | FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS. | 10 |
| [O-02] | PROPER USE OF "GET" AS A FUNCTION NAME PREFIX | 5 |
| [O-03] | USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS | 1 |
| [O-04] | <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO) | 5 |
| [O-05] | INCONSISTENT IN COMMENTS |  |

| Total Ordinary Issues | 5 | Total Instances | 21 |
|:--:|:--:|:--:|--:|

### Suggestion Details 
| Count | Explanation | 
|:--:|:-------|
| [S-01] | GENERATE PERFECT CODE HEADERS EVERY TIME  |  |

| Total Suggestions | 1 |
|:--:|:--:|


# Detailed Findings

## [L-01] USE BLOCK.NUMBER INSTEAD OF BLOCK.TIMESTAMP

`Block timestamps` have historically been used for a variety of applications, such as entropy for random numbers (see the Entropy Illusion for further details), locking funds for periods of time, and various state-changing conditional statements that are `time-dependent`. Miners have the ability to adjust timestamps slightly, which can prove to be dangerous if block timestamps are used incorrectly in smart contracts.

### PROOF OF CONCEPT
- [Lottery Contract in the line 135](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L135)
- [Lottery Contract in the line 164](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L164)
- [StakedTokenLock Contract in the line 26](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L26)
- [StakedTokenLock Contract in the line 39](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39)

### MITIGATION
Block timestamps should not be used for entropy or generating random numbers—i.e., they should not be the deciding factor (either directly or through some derivation) for winning a game or changing an important state.

Time-sensitive logic is sometimes required; e.g., for unlocking contracts (time-locking) , It is sometimes recommended to use `block.number` and an average block time to estimate times; with a `10 second block time`, `1 week` equates to approximately, `60480 blocks`. 

`Thus, specifying a block number at which to change a contract state can be more secure, as miners are unable to easily manipulate the block number.`


## [L-02] IMMUTABLE ADDRESSES LACK ZERO-ADDRESS CHECK
Constructors should check the address written in an immutable address variable is not the zero address.

### PROOF OF CONCEPT

```solidity
src/LotteryToken.sol

14: address public immutable override owner;
```
- [LotteryToken Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L17-L18)


## [L-03] USE OF FLOATING PRAGMA 
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

### PROOF OF CONCEPT
```solidity

src/VRFv2RNSource.sol
3: pragma solidity ^0.8.7;

src/staking/StakedTokenLock.sol
3: pragma solidity ^0.8.17;
```

### MITIGATION
It is recommended to pin to a concrete compiler version.


## [L-05] MIXED COMPILER VERSIONS
The project is compiled with different versions of solidity, which is not recommended due the undefined behaviors as a result of it.

## [L-06] | OWNER CAN RENOUNCE OWNERSHIP
Typically, the contract’s owner is the account that deploys the contract. As a result, the owner is able to perform certain privileged activities.

The Openzeppelin’s Ownable used in this project contract implements renounceOwnership. This can represent a certain risk if the ownership is renounced for any other reason than by design. Renouncing ownership will leave the contract without an owner, thereby removing any functionality that is only available to the owner.

### PROOF OF CONCEPT
```solidity
src/staking/StakedTokenLock.sol

24: function deposit(uint256 amount) external override onlyOwner 
33: function withdraw(uint256 amount) external override onlyOwner
```
### MITIGATION
We recommend to either reimplement the function to disable it or to clearly specify if it is part of the contract design.

## [N-01] SORT SOLIDITY OPERATIONS USING SHORT-CIRCUIT MODE
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
src/staking/StakedTokenLock.sol

39: if (block.timestamp > depositDeadline && block.timestamp < depositDeadline + lockDuration) {//} 
```
- [LotterySetup Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L54-L70)
- [LotterySetup Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotterySetup.sol#L165)


## [N-02] MISSING EMERGENCY STOP (CIRCUIT BREAKER) PATTERN
At the start of the project, the system may need to be stopped or upgraded, I suggest you have a script beforehand and add it to the documentation. This can also be called an `EMERGENCY STOP (CIRCUIT BREAKER) PATTERN`.

[Emergency Stop Pattern Example](https://github.com/maxwoe/solidity_patterns/blob/master/security/EmergencyStop.sol)


## [N-03] TAKE ADVANTAGE OF CUSTOM ERROR'S RETURN VALUE PROPERTY
An important feature of Custom Error is that values such as address, tokenID, msg.value can be written inside the `()` sign, this kind of approach provides a serious advantage in debugging and examining the revert details of dapps such as tenderly.

### PROOF OF CONCEPT
```solidity
src/Lottery.sol

45: if (!ticket.isValidTicket(selectionSize, selectionMax)) {revert InvalidTicket();}
53: if (drawExecutionInProgress) { revert DrawAlreadyInProgress(); }
61: if (!drawExecutionInProgress) {revert DrawNotInProgress(); }  
135: if (block.timestamp < drawScheduledAt(currentDraw)) {revert ExecutingDrawTooEarly();}
```

- [Staking Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L31-L39)
- [Staking Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L69)
- [Staking Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/Staking.sol#L82)
- [StakedTokenLock Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L26)
- [StakedTokenLock Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/staking/StakedTokenLock.sol#L39)
- [LotteryToken Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/LotteryToken.sol#L24)


## [N-04] FUNCTION WRITING THAT DOES NOT COMPLY WITH THE SOLIDITY STYLE GUIDE
The best-practice layout for a contract should follow the following order: state variables, events, modifiers, constructor and functions. Function ordering helps readers identify which functions they can call and find constructor and fallback functions easier. Functions should be grouped according to their visibility and ordered as: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Functions should then further be ordered with view functions coming after the non-view labeled ones

[Solidity Order Functions](https://docs.soliditylang.org/en/v0.8.17/style-guide.html#order-of-functions)

`Functions should be grouped according to their visibility and ordered:`

### PROOF OF CONCEPT
```solidity
constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private

```
`within a grouping, place the view and pure functions last`


## [N-05] TOKENS ACCIDENTALLY SENT TO THE CONTRACT CANNOT BE RECOVERED
It can't be recovered if the tokens accidentally arrive at the contract address, which has happened to many popular projects, so I recommend adding a recovery code to your critical contracts.

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

## [N-06] CREATE YOUR OWN IMPORT NAMES INSTEAD OF USING THE REGULAR ONES
For better readability, you should name the imports instead of using the regular ones.

Example:
```solidity
5: import { ExampleView } from "../libraries/ExampleView.sol";
```
`Instances - All of the contracts.`


## [N-07] INTERNAL FUNCTIONS ONLY CALLED ONCE CAN BE INLINED TO SAVE GAS
Not inlining costs 20 to 40 gas because of two extra JUMP instructions and additional stack operations needed for function calls.

Inlining optimization is a technique that can improve code efficiency and save gas. When a function is called in Solidity, a JUMP to the function's location occurs, followed by the execution of the function's instructions, and then another JUMP back to the point of origin. This process requires the use of additional memory and therefore costs gas.

However, if a function is called only once and has no side effects on the contract state, inlining optimization can be used to save gas. Instead of jumping to the function's location, the compiler can simply copy and paste the function's content to the place where the function is called. This eliminates the costs of additional jump instructions and additional stack operations needed for the function call.

You can see a [simple example](https://gist.github.com/catellaTech/b33b3567b71149869aa3bf280de4795f) of this.


## [N-08] LACK OF ADDRESS(0) CHECKS IN THE CONSTRUCTOR
Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly.

### PROOF OF CONCEPT
```solidity
src/staking/StakedTokenLock.sol

16: constructor(address _stakedToken, uint256 _depositDeadline, uint256              _lockDuration) {
        _transferOwnership(msg.sender);
        stakedToken = IStaking(_stakedToken);
        rewardsToken = stakedToken.rewardsToken();
        depositDeadline = _depositDeadline;
        lockDuration = _lockDuration;
    }
```
- [Lottery Contract](https://github.com/code-423n4/2023-03-wenwin/blob/main/src/Lottery.sol#L84-L105)

### MITIGATION

Consider adding zero-address checks in the discussed constructors:  

```solidity
require(newAddr != address(0));
```

## [N-09] MISSING NATSPEC
Some functions don't have description of what they are supposed to do.
There are too many instances with this inconsistency.

## [N-10] DO NOT PRE-DECLARE VARIABLE WITH DEFAULT VALUES
If a variable is not set/initialized, it is assumed to have the default value (`0` for `uint`, `false` for `bool`, `address(0)` for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas.

example: 

Not recommended ❌:

```solidity
for (uint256 i = 0 ; i ...; i++) {// ...}
```
Recommended ✔: 

```solidity
for (uint256 i; i ...; i++) {// ...}
```

### PROOF OF CONCEPT
```solidity
src/Lottery.sol
125: for (uint256 i = 0; i < drawIds.length; ++i) {
         ticketIds[i] = registerTicket(drawIds[i],
         tickets[i], frontend, referrer);
        }

172: for (uint256 i = 0; i < totalTickets; ++i) {
            claimedAmount += claimWinningTicket(ticketIds[i]);
        }        
```

## [N-11] INSUFFICIENT COVERAGE
The test coverage rate of the project is 85%. Testing all functions is best practice in terms of security criteria.

Due to its capacity, test coverage is expected to be 100%.

## [N-12] ADD NATSPEC MAPPING COMMENT
Add NatSpec comments describing mapping keys and values.

### PROOF OF CONCEPT
```solidity
src/Lottery.sol

27: mapping(uint128 => mapping(uint120 => uint256)) private unclaimedCount;
37: mapping(uint128 => mapping(uint8 => uint256)) public override winAmount;
```

## [N-13] USE SMTChecker 
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

[ Check the next example. ](https://twitter.com/0xOwenThurm/status/1614359896350425088?t=dbG9gHFigBX85Rv29lOjIQ&s=19)


## [N-14] NATSPEC IS INCOMPLETE
The use of `NatSpec` can improve the understanding and security of the contract for all parties involved. This documentation can be used by developers, users, and auditors of the contract to better understand what the contract does and how it works, without the need to examine the source code directly. In addition, the detailed and clear documentation can help prevent errors and misunderstandings in the development and auditing process of the contract.

`All of the contracts`


## [R-01] "GET" IS TYPICALLY RESERVED FOR (VIEW/PURE) FUNCTIONS
Clear function names can increase readability. Follow a standard convertion function names such as using get for getter (view/pure) functions.

### PROOF OF CONCEPT
```solidity
src/staking/StakedTokenLock.sol
50: function getReward() external override

src/staking/Staking.sol
91: function getReward() public override 
```

## [R-02] SHORTHAND WAY TO WRITE IF/ELSE STATEMENT
The normal if / else statement can be refactored in a shorthand way to write it:
- Increases readability
- Shortens the overall SLOC.

### PROOF OF CONCEPT
```solidity
src/LotterySetup.sol
120: function fixedReward(uint8 winTier) public view override returns (uint256 amount) {
        if (winTier == selectionSize) {
            return _baseJackpot(initialPot);
        } else if (winTier == 0 || winTier > selectionSize) {
            return 0;
        } else {
            uint256 mask = uint256(type(uint16).max) << (winTier * 16);
            uint256 extracted = (nonJackpotFixedRewards & mask) >> (winTier * 16);
            return extracted * (10 ** (IERC20Metadata(address(rewardToken)).decimals() - 1));
        }
    }

src/Lottery.sol
91:  function dueTicketsSoldAndReset(address beneficiary) private returns (uint256 dueTickets) {
        if (beneficiary == stakingRewardRecipient) {
            dueTickets = nextTicketId - claimedStakingRewardAtTicketId;
            claimedStakingRewardAtTicketId = nextTicketId;
        } else {
            dueTickets = frontendDueTicketSales[beneficiary];
            frontendDueTicketSales[beneficiary] = 0;
        }
    }
```
`Multiple instances accross the contracts.`

## [R-03] USE REQUIRE INSTEAD OF ASSERT
Properly functioning code should never reach a failing assert statement. If it happened, it would indicate the presence of a bug in the contract. A failing assert uses all the remaining gas, which can be financially painful for a user.

### PROOF OF CONCEPT
```solidity
src/LotterySetup.sol
147:  assert(initialPot > 0);
```

## [R-04] USING CALLDATA INSTEAD OF MEMORY 
When a function with a `memory` array is called externally, the `abi.decode()` step has to use a for-loop to copy each index of the `calldata` to the `memory` index. **Each iteration of this for-loop costs at least 60 gas** (i.e. `60 * <mem_array>.length`). Using `calldata` directly, obliviates the need for such a loop in the contract code and runtime execution.

If the array is passed to an `internal` function which passes the array to another internal function where the array is modified and therefore `memory` is used in the `external` call, it’s still more gass-efficient to use `calldata` when the `external` function uses modifiers, since the modifiers may prevent the internal functions from being called. Structs have the same overhead as an array of length one

### PROOF OF CONCEPT
```solidity
src/LotterySetup.sol
164: function packFixedRewards(uint256[] memory rewards)....;
```
## [R-05] CACHE THE MAPPING VALUES RATHER THAN FETCH IT EVERY TIME
The instances below point to the second+ access of a value inside a mapping/array, within a function. Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times, saves ~42 gas per access due to not having to recalculate the key's keccak256 hash (Gkeccak256 - 30 gas) and that calculation's associated stack operations. 

### PROOF OF CONCEPT
```solidity
src/Lottery.sol
165:  claimableAmount = winAmount[ticketInfo.drawId][winTier];
212:  winAmount[drawFinalized][winTier] = drawRewardSize(drawFinalized, winTier);
```
`Multiple instances accross the contracts.`

## [O-01] FOR FUNCTIONS, FOLLOW SOLIDITY STANDARD NAMING CONVENTIONS
The above codes don’t follow Solidity’s standard naming convention, internal and private functions: the mixedCase format starting with an underscore (_mixedCase starting with an underscore)

### PROOF OF CONCEPT
```solidity
src/VRFv2RNSource.sol
28:  function requestRandomnessFromUnderlyingSource() internal override returns;
32:  function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords) internal override ;

src/LotterySetup.sol
165:  function packFixedRewards(uint256[] memory rewards) private view returns

src/Lottery.sol
181:  function registerTicket(
        uint128 drawId,
        uint120 ticket,
        address frontend,
        address referrer
    )
        private ...
238: function drawRewardSize(uint128 drawId, uint8 winTier) private
249: function dueTicketsSoldAndReset(address beneficiary) private
259: function claimWinningTicket(uint256 ticketId) private 
271: function returnUnclaimedJackpotToThePot() private 
279: function requireFinishedDraw(uint128 drawId) internal
285: function requireFinishedDraw(uint128 drawId) internal
```
We recommend following your own patterns, as in this case: 

```solidity
 function _baseJackpot(uint256 _initialPot) internal view returns 
``` 

## [O-02] PROPER USE OF "GET" AS A FUNCTION NAME PREFIX
Clear function names can increase readability. Follow a standard convertion function names such as using get for getter (view/pure) functions.

### PROOF OF CONCEPT
```solidity
src/LotterySetup.sol
152: function drawScheduledAt(uint128 drawId) public view override returns
156: function ticketRegistrationDeadline(uint128 drawId) public view 
120: function fixedReward(uint8 winTier) public view

src/staking/Staking.sol
48: function rewardPerToken() public view 
61: function earned(address account) public view
```

## [O-03] USING STORAGE INSTEAD OF MEMORY FOR STRUCTS/ARRAYS SAVES GAS
When fetching data from a storage location, assigning the data to a memory variable causes all fields of the struct/array to be read from storage, which incurs a Gcoldsload (2100 gas) for each field of the struct/array. If the fields are read from the new memory variable, they incur an additional MLOAD rather than a cheap stack read. Instead of declaring the variable with the memory keyword, declaring the variable with the storage keyword and caching any fields that need to be re-read in stack variables, will be much cheaper, only incuring the Gcoldsload for the fields actually read. The only time it makes sense to read the whole struct/array into a memory variable, is if the full struct/array is being returned by the function, is being passed to a function that requires memory, or if the array/struct is being read from another memory array/struct.

### PROOF OF CONCEPT
```solidity
src/Lottery.sol
160:  TicketInfo memory ticketInfo = ticketsInfo[ticketId];
```

## [O-04] <X> += <Y> COSTS MORE GAS THAN <X> = <X> + <Y> FOR STATE VARIABLES (-= TOO)
Using the addition operator instead of plus-equals saves [113 gas](https://gist.github.com/catellaTech/8539f7345b17f0929a41598e7e00e3c2). Subtractions act the same way.

### PROOF OF CONCEPT
```solidity
src/staking/StakedTokenLock.sol
30: depositedBalance += amount;
43: depositedBalance -= amount;

src/Lottery.sol
129: frontendDueTicketSales[frontend] += tickets.length;
173: claimedAmount += claimWinningTicket(ticketIds[i]);
275: currentNetProfit += int256(unclaimedJackpotTickets * winAmount[drawId][selectionSize]);
```

## [O-05] INCONSISTENT IN COMMENTS
Most comments in the codebase do not end with a `.`, it is best to keep a consistent style.

`Multiple instances accross the contracts.`

## [S-01]  GENERATE PERFECT CODE HEADERS EVERY TIME 
I recommend using header for Solidity code layout and readability

[This is gonna help you.](https://github.com/transmissions11/headers)


/*//////////////////////////////////////////////////////////////
                           TESTING 1
//////////////////////////////////////////////////////////////*/