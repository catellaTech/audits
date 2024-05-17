# C4 Quest Protocol Gas Report

## Files analyzed
- quest-protocol\contracts\Erc1155Quest.sol
- quest-protocol\contracts\Erc20Quest.sol
- quest-protocol\contracts\Quest.sol
- quest-protocol\contracts\QuestFactory.sol
- quest-protocol\contracts\RabbitHoleReceipt.sol
- quest-protocol\contracts\RabbitHoleTickets.sol
- quest-protocol\contracts\ReceiptRenderer.sol
- quest-protocol\contracts\TicketRenderer.sol


## Issues found

## Don't Initialize Variables with Default Value

#### Impact
Issue Information: [G001]

#### Findings:
```
quest-protocol\contracts\Quest.sol::103 => uint256 redeemableTokenCount = 0;
quest-protocol\contracts\RabbitHoleReceipt.sol::115 => uint foundTokens = 0;
quest-protocol\contracts\RabbitHoleReceipt.sol::126 => uint filterTokensIndexTracker = 0;
```



## Use != 0 instead of > 0 for Unsigned Integer Comparison

#### Impact
Issue Information: [G002]

#### Findings:
```
quest-protocol\contracts\RabbitHoleReceipt.sol::129 => if (tokenIdsForQuest[i] > 0) {
```


## Use immutable for OpenZeppelin AccessControl's Roles Declarations

#### Impact
Issue Information: [G003]

#### Findings:
```
quest-protocol\contracts\QuestFactory.sol::17 => bytes32 public constant CREATE_QUEST_ROLE = keccak256('CREATE_QUEST_ROLE');
quest-protocol\contracts\QuestFactory.sol::211 => bytes32 messageDigest = keccak256(abi.encodePacked('\x19Ethereum Signed Message:\n32', hash_));

```
## Use calldata Instead of Memory for Function Parameters

#### Impact
Issue Information: [G004]

#### Findings:
```
quest-protocol\contracts\Quest.sol::2 =>     function _setClaimed(uint256[] memory tokenIds_);
```

## INCREMENTS CAN BE UNCHECKED

## Description 
In Solidity 0.8+, there’s a default overflow check on unsigned integers. It’s possible to uncheck this in for-loops and save some gas at each iteration, but at the cost of some code readability, as this uncheck cannot be made inline.

#### Impact
Issue Information: [G005]

#### Findings:

```
quest-protocol\contracts\Quest.sol::70 =>   for (uint i = 0; i < tokenIds_.length; i++);
quest-protocol\contracts\Quest.sol::104 =>  for (uint i = 0; i < tokens.length; i++);
quest-protocol\contracts\RabbitHoleReceipt.sol::117 => for (uint i = 0; i < msgSenderBalance; i++);
quest-protocol\contracts\RabbitHoleReceipt.sol::128 => for (uint i = 0; i < msgSenderBalance; i++);
```

## The code would go from:

```
for (uint i = 0; i < tokenIds_.length; i++) {.....}
```

## to:

```
for (uint i = 0; i < tokenIds_.length;) {  
 // ...  
 unchecked { ++i; }  
}  
```