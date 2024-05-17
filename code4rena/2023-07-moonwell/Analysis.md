# Analysis of the Moonwell Contest

## Introduction
The Moonwell Protocol is a fork of Benqi, which in turn is a fork of Compound v2, and it features borrow caps and multi-token emissions. It is an open lending and borrowing protocol built on Base, Moonbeam, and Moonriver, allowing you to leverage what's yours by simplifying the lending and borrowing process.

## Analysis of the Codebase
There are 12 contracts in the scope, of which the main ones that users will interact with are:
    - MToken.sol
    - Comptroller.sol
    - MultiRewardDistributor.sol

Let's take a look at how the flow will look when a user interacts with the Moonwell protocol:

1. When a user calls the mint function in MToken, it will call the mintAllowed function in the Comptroller.
2. If minting is allowed, then the updateMarketSupplyIndexAndDisburseSupplierRewards function in the MultiRewardDistributor contract is called.
3. This function then updates the supply reward index in the MultiRewardDistributor.
4. Subsequently, the user's individual rewards are updated by calling the balanceOf function in MToken.

In summary, the interactions that occur when a user calls mint are: 
    `MToken -> Comptroller -> MultiRewardDistributor -> MToken`
        
        +--------------+       +-------------------+       +---------------------------+
        |              |       |                   |       |                           |
        |   MToken     <------->   Comptroller    +------->  MultiRewardDistributor    |
        |              |       |                   |       |                           |
        +-------+------+       +---------+---------+       +---------------+-----------+
                ^                         
                |                                               
                |
                v
        +-------+------+        
        |              |
        |     User     |
        |              |
        +--------------+

The 3 components are highly interconnected and dependent on each other.

While everything is interconnected, the Comptroller.sol contract is the main one, as it allows:
    - Disbursing protocol rewards
    - Determining liquidity for lenders based on price oracles
    - Listing new markets
    - Storing and Updating Risk Parameters
    - Authorizing liquidations (determining if a position is liquidatable)
    - Authorizing minting/supplying to the protocol
    - Authorizing redemptions/withdraws from the protocol
    - Authorizing loan repayments
    - Authorizing mToken movements

## Architecture Feedback, Centralization and Systemic Risks ⚒⚠
- We have observed that the protocol strives to achieve decentralization. However, we have emphasized in the QA that there is a need for significantly more comprehensive documentation, particularly regarding the nature of the admin account. Specifically, it should be clarified whether the admin is controlled by a smart contract governed by a multisig or by an Externally Owned Account (EOA). This aspect raises concerns about the centralization of the project since this particular address holds the authority to make critical changes.

-  The architecture Comptroller is the central brain to the entire Moonwell protocol, it allows multiples criticals functions.

- The architecture of MTokens are an ERC-20 compliant token that is used to track positions within the Moonwell protocol across markets. mTokens are transferrable and fungible, and can be redeemed for an underlying position assuming there is sufficient liquidity and that position represented by those tokens haven't been marked for use as collateral

- The architecture of MultiRewardDistributor, integrates with the Moonwell Comptroller and manages all reward disbursal and index calculations both for the global market indices as well as individual user indices on those markets. It is largely the same logic that compound uses, just generalized (meaning that transfers will not fail if things can't be sent out, but the excess is accrued on the books to be sent later).

- We have identified two significant risks, one of which we mentioned in the centralization risk, and the other in the JumpRateModel.sol contract, the variable timestampsPerYear is a constant variable, which means it cannot be modified. This is dangerous because this variable is used in important calculations, and it is not specified how these calculations will behave in cases where a year is not 365 days. 


## Time Spent ⏱
A total of 4 days were spent to cover this audit, broken down into the following:

- 1st Day: Understand protocol docs, action creation flow and policy management
- 2nd Day: Focus on linking docs logic to Comptroller.sol, MultiRewardDistributor.sol and MTokens coupled with typing reports for vulnerabilities found
- 3rd Day: Focus on different types of strategies contract coupled with typing reports for vulnerabilities found
- 4th Day: Sum up audit by completing QA report and Analysis





### Time spent:
96 hours