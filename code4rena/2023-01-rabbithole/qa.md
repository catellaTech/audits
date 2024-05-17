# C4 Quest Protocol QA Report


## Unspecific Compiler Version Pragma

## Description
Avoid floating pragmas for non-library contracts.

While floating pragmas make sense for libraries to allow them to be included with multiple different versions of applications, it may be a security risk for application implementations.

A known vulnerable compiler version may accidentally be selected or security tools might fall-back to an older compiler version ending up checking a different EVM compilation that is ultimately deployed on the blockchain.

It is recommended to pin to a concrete compiler version.

#### Impact
Issue Information: [L001]

#### Findings:
```
quest-protocol\contracts\Erc1155Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\Erc20Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\Quest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\QuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\RabbitHoleReceipt.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\RabbitHoleTickets.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\ReceiptRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\TicketRenderer.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\interfaces\IQuest.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\interfaces\IQuestFactory.sol::2 => pragma solidity ^0.8.15;
quest-protocol\contracts\test\SampleErc1155.sol::2 => pragma solidity ^0.8.15;
```

## Non-Critial Issues
## Impact
Issue Information: [NC-001] - Functions Mutating Storage Should Emit Events
## Description
Functions mutating storage should emit an Event for easy off-chain monitoring.

#### Findings:
```
quest-protocol\contracts\Erc1155Quest.sol::41 =>  function _transferRewards(uint256 amount_);
quest-protocol\contracts\Erc1155Quest.sol::102 => function withdrawFee() public onlyAdminWithdrawAfterEnd;
quest-protocol\contracts\Erc1155Quest.sol::33 => function start() public override;
```

## QA
## Impact
Issue Information: [QA-001] - transferOwnership should be two step process
## Description
"QuestFactory.sol" inherit OpenZeppelin's OwnableUpgradeable contract which enables the onlyOwner role to transfer ownership to another address. It's possible that the onlyOwner role mistakenly transfers ownership to the wrong address, resulting in a loss of the onlyOwner role. The current ownership transfer process involves the current owner calling newQuest.transferOwnership(msg.sender) If the nominated EOA account is not a valid account, it is entirely possible the owner may accidentally transfer ownership to an uncontrolled account, breaking all functions with the onlyOwner() modifier. Lack of two-step procedure for critical operations leaves them error-prone
if the address is incorrect, the new address will take on the functionality of the new role immediately

for Example : -Alice deploys a new version of the whitehack group address. When she invokes the whitehack group address setter to replace the address, she accidentally enters the wrong address. The new address now has access to the role immediately and is too late to revert

```
The contract have many onlyOwner function and the  contract is inherited from the Ownable which includes transferOwnership. Recommended Mitigation Steps
Implement zero address check and Consider implementing a two step process where the owner nominates an account and the nominated account needs to call an acceptOwnership() function for the transfer of ownership to fully succeed. This ensures the nominated EOA account is a valid and active account.
```