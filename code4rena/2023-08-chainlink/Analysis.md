![chainlink](https://upload.wikimedia.org/wikipedia/commons/thumb/1/15/Chainlink_Logo_Blue.svg/250px-Chainlink_Logo_Blue.svg.png)
# Chainlink Staking v0.2 Smart Contracts Analysis

## Description Overview of Chainlink Staking v0.2
The goal of ``Chainlink`` is launching a significant ``upgrade`` to its ``staking platform``, known as ``version 0.2``. Staking allows individuals to support ``Chainlink's oracle services`` with ``LINK`` tokens and earn rewards. ``Version 0.2`` is more ``secure``, ``flexible``, and ``modular`` compared to its predecessor ``(v0.1)``. This upgrade is part of the Chainlink Economics 2.0 initiative and aims to enhance the security and flexibility of the staking platform.

![Chainlink](https://github.com/catellaTech/chainlink/blob/main/chainlinkOverview.drawio.png?raw=true)

## 1- Chainlink Staking v0.2 System Overview
We ``audited`` the ``contracts`` that were shared by the ``Chainlink`` team.
We have decided to create ``diagrams`` detailing each part of the ``functions`` of the ``smart contracts`` provided by the ``Chainlink`` protocol and to create a summary of the functionality of each of the ``contracts``. This approach proves to be ``effective`` for ``us`` as it allows us to thoroughly understand all the functionalities of each ``contract`` and visually ``document`` what we have ``comprehended`` through ``diagrams``. Furthermore, we believe that this approach can be useful in enhancing the understanding of the ``contracts`` among ``developers``, ``auditors``, and ``users``.

### **Scope**
The scope includes the following ``files``:

- **Migratable.sol**: 
    This ``contract`` provides a framework for managing the ``migration`` of a contract to a ``new address``. It enforces validation of the migration target address and allows setting and getting the migration target. It's designed to be extended by other contracts that implement the specific migration logic needed for their use case.

![Migratable](https://github.com/catellaTech/chainlink/blob/main/chainlinkMigratable.drawio.png?raw=true)

-  **MigrationProxy.sol**: 
  This is ``contract`` is used as an intermediary to ``migrate`` stakers from a ``V0.1`` staking contract to ``V0.2`` staking contracts, either in the ``Operator Staking Pool`` or the ``Community Staking Pool``, depending on the type of staker. It facilitates this ``migration`` and ensures that certain conditions are met before proceeding. It also provides functions to retrieve configuration information and verify implemented interfaces.

![MigrationProxy](https://github.com/catellaTech/chainlink/blob/main/Copia%20de%20chainlinkMigrationProxy.drawio.png?raw=true)

-  **PausableWithAccessControl.sol**: This ``contract`` extends ``OpenZeppelin's`` ``Pausable`` and ``AccessControlDefaultAdminRules`` contracts to provide access control and pausing functionality. It introduces the ``pauser role``, which allows certain addresses to ``pause`` and ``unpause`` the contract in case of emergencies. This contract is designed to enhance the security and control of the smart contract system it is integrated with.

![PausableWithAccessControl](https://github.com/catellaTech/chainlink/blob/main/chainlinkPausableWithAccessControl.png?raw=true)

-  **PriceFeedAlertsController.sol**: This ``contract`` allows alerters ``(specific users)`` to raise alerts when a ``Chainlink feed`` (an oracle that provides external data) becomes inactive or fails to update its data at the expected frequency. This ``contract`` is used to control alerts in systems based on ``Chainlink``.

![PriceFeedAlertsController](https://github.com/catellaTech/chainlink/blob/main/chainlinkPriceFeedAlertsController.png?raw=true)

-  **CommunityStakingPool.sol**: This ``contract`` manages the ``staking`` of ``LINK`` tokens for the ``community``. It provides a mechanism for verifying access to the community staking pool using ``Merkle proofs``. It also ``allows`` for changing the ``Merkle root`` and configuring the ``OperatorStakingPool`` contract.

![CommunityStakingPool](https://github.com/catellaTech/chainlink/blob/main/chainlinkCommunityStakingPool.png?raw=true)

- **OperatorStakingPool.sol**: This ``contract``  manages the ``staking`` of ``LINK`` tokens for ``operator stakers``. It has features to ``add`` and ``remove`` operators, handle slashing and rewards, and manage the alerter reward funds. It interacts with other contracts in the system and enforces access control through roles like ``SLASHER`` and ``DEFAULT_ADMIN_ROLE``.
  
![OperatorStakingPool](https://github.com/catellaTech/chainlink/blob/main/chainlinkOperatorStakingPool.png?raw=true)

- **StakingPoolBase.sol**: this is an ``abstract contract`` that encapsulates shared logic utilized by both the ``CommunityStakingPool`` and ``OperatorStakingPool`` contracts.

![StakingPoolBase](https://github.com/catellaTech/chainlink/blob/main/chainlinkStakingPoolBase.png?raw=true)


- **RewardVault.sol**: The primary purpose of the ``RewardVault`` contract is to manage the rewards distributed to participants in a staking system. The contract is designed to interact with two types of staking pools: the ``Community Staking Pool`` and the ``Operator Staking Pool``. Users can deposit rewards into the contract and control the aggregated reward rate for each pool, enabling controlled distribution of rewards to staking participants.

![RewardVault](https://github.com/catellaTech/chainlink/blob/main/chainlinkRewardVault.png?raw=true)

- **StakingTimelock.sol**: The ``Staking Timelock`` contract is responsible for managing ``operational changes`` to the other staking contracts. Following a specified ``timelock`` period, all operational transactions must be executed through this contract's address.

![StakingTimelock](https://github.com/catellaTech/chainlink/blob/main/chainlinkStakingTimelock.png?raw=true)

- **Timelock.sol**: This is the foundational ``timelock contract`` from which ``StakingTimelock`` inherits, and it defines all the ``timelock`` logic required for ``proposing``, ``executing``, and ``canceling`` transactions.

![Timelock](https://github.com/catellaTech/chainlink/blob/main/chainlinkTimelock.png?raw=true)

### **Privileged Roles**

The ``Timelock`` contract defines some ``access roles`` with different levels of ``privileges``. 
Here are the ``privilege roles``:

- **ADMIN_ROLE**: The ``administrator`` role is the most powerful role. The administrator can manage membership for all other roles. Can ``add`` or ``remove`` addresses to other roles.
```solidity
bytes32 public constant ADMIN_ROLE = keccak256('ADMIN_ROLE');
```
  **Notes**: This role is expected to be inhabited by a contract or entity that requires a secure quorum of votes before taking any action. It is mainly used for emergency actions or Timelock configuration.

- **PROPOSER_ROLE**: ``Proposers`` can schedule operations with a delay.
```solidity
bytes32 public constant PROPOSER_ROLE = keccak256('PROPOSER_ROLE');
```

- **EXECUTOR_ROLE**: ``Executors`` can execute previously scheduled operations once the delay has expired.
```solidity
bytes32 public constant EXECUTOR_ROLE = keccak256('EXECUTOR_ROLE');
```

- **CANCELLER_ROLE**: ``Cancellers`` can cancel operations that have been scheduled but not yet executed.
```solidity
bytes32 public constant CANCELLER_ROLE = keccak256('CANCELLER_ROLE');
```

The contract "``PausableWithAccessControl``" defines a single ``privileged role`` ------> **PAUSER_ROLE**: Can ``pause`` and ``unpause`` the contract.
```solidity
bytes32 public constant PAUSER_ROLE = keccak256('PAUSER_ROLE');
```

Given the ``level of control`` that these ``roles`` possess ``within the system``, users must place full trust in the fact that the ``entities`` overseeing these ``roles`` will always act ``correctly`` and in the ``best interest`` of the ``system`` and its ``users``.

## 3- Architecture Feedback 
The ``architecture`` of ``Chainlink Staking Platform v0.2`` has been thoughtfully designed with a focus on flexibility for stakers, modularity for future updates and improvements, dynamic rewards based on the state of the staking pool, and enhanced cryptoeconomic security with the ability to ``slash`` by ``Node Operator Stakers``. These features that ``Chainlink`` has implemented are designed to strengthen the security and economic sustainability of the ``Chainlink`` network. There's nothing more to add, honestly, ``Chainlink Staking Platform v0.2`` has been one of the ``best projects`` we have reviewed.

### Architecture recommendations
The platform supports a very generic well implementation, and as the team understands, this leads to a broad variety of malicious deployed well risks. It should be a high priority task for the team to find good ways of representing all Well trust assumptions to its users, and expose that through a smart contract UI or an open source front-end library

## 4- Documents

Carrying out this ``code review`` has been an extremely ``user-friendly experience``, as it has allowed us to comprehensively understand each aspect of the ``system's operation``. The ``documentation`` of the ``Chainlink Staking Platform v0.2`` project is exceptionally thorough and comprehensive, providing a strong overall view of the project's structure and how its various components function. To further enhance this detailed documentation, we have chosen to create comprehensive ``diagrams`` that visually represent each contract provided by ``Chainlink``. ``With considerable enthusiasm``, we have dedicated several days to crafting these ``diagrams`` with the aim of providing an even clearer and more accessible understanding of the underlying project architecture.
We are confident that these ``diagrams`` will bring significant value to the protocol as they can be seamlessly integrated into the existing `documentation`, enriching it and providing a more comprehensive and detailed understanding for `users`, `developers` and `auditors`.

## 5- Systemic Risks
The code review of the ``Chainlink Staking Platform v0.2 protocol`` aims to identify key systematic risks related to the ``integrity`` and ``proper`` functioning of the ``system``. These risks encompass various areas, including the integrity of ``staking`` and ``unstaking`` mechanisms, ``reward calculation`` and ``claiming logic``, contract upgrade patterns, access controls, staking pool management, reward vaults, staking and unstaking processes, rewards, alerts, and migration procedures. These ``concerns`` are focused on ensuring the security and integrity of the protocol, preventing potential situations of exploitation or asset loss by participants. Once the findings provided by ``C4 experts`` are shared, the ``development team`` must effectively address these risks to ensure the reliability and robustness of the system.

As the entire platform is ``unupgradeable``, migration in the event of a ``security hole`` or a ``black swan ``event will be challenging. The team should prepare ``multiple recovery scenarios`` and set up adequate action channels to prepare for such eventualities.


## 6- Monitoring Recommendations

While ``audits`` help in ``identifying`` code-level ``issues`` in the current implementation and potentially the code ``deployed`` in production, the ``Delegate`` team is encouraged to consider incorporating monitoring activities in the production environment. Ongoing monitoring of deployed contracts helps identify potential threats and issues affecting production environments. With the goal of providing a complete ``security assessment``, the monitoring ``recommendations`` section raises several actions addressing trust assumptions and out-of-scope components that can benefit from ``on-chain monitoring``.

## Conclusion 

To conclude this ``analysis``, we wish to congratulate the entire ``Chainlink development team`` for providing such a ``well-documented project`` with excellent coding practices. ``Chainlink`` serves as an exemplary model for other projects, thanks to its comprehensive documentation, which has allowed us to deliver a higher-quality service. In the event that critical issues are identified by the ``C4 wardens``, we have confidence that the protocol will address them in the best possible manner.

## Time Spent ⏱

A total of `3 days` were dedicated to completing this audit, distributed as follows:

1. 1st Day: Devoted to comprehending the protocol's functionalities and implementation.
2. 2nd Day: Initiated the analysis process, leveraging the documentation offered by the `Chainlink Staking v0.2`.
3. 3rd Day: Focused on conducting a thorough analysis, incorporating meticulously crafted diagrams derived from the contracts and information provided by the protocol.


### Time spent:
20 hours