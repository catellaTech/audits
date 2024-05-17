## [H-01] USE THE OPENZEPPELIN SAFECAST LIBRARY FOR CRITICAL FUNCTIONS
We have noticed that the contract `PrivatePool.sol.` implements many type conversions from uint256 to uint128. We recommend using the OpenZeppelin SafeCast library to make the project more robust and take advantage of the gas optimizations and best practices provided by OpenZeppelin.

### PROOF OF CONCEPT
```solidity
main/src/PrivatePool.sol

211: function buy(uint256[] calldata tokenIds, uint256[] calldata tokenWeights, MerkleMultiProof calldata proof)
        public
        payable
        returns (uint256 netInputAmount, uint256 feeAmount, uint256 protocolFeeAmount)
    {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to buy
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the required net input amount and fee amount
        (netInputAmount, feeAmount, protocolFeeAmount) = buyQuote(weightSum);

        // check that the caller sent 0 ETH if the base token is not ETH
        if (baseToken != address(0) && msg.value > 0) revert InvalidEthAmount();

        // ~~~ Effects ~~~ //

        // update the virtual reserves
        virtualBaseTokenReserves += uint128(netInputAmount - feeAmount - protocolFeeAmount); // @audit Implement safe cast 
        virtualNftReserves -= uint128(weightSum); // @audit Implement safe cast 
    }

301: function sell(
        uint256[] calldata tokenIds,
        uint256[] calldata tokenWeights,
        MerkleMultiProof calldata proof,
        IStolenNftOracle.Message[] memory stolenNftProofs // put in memory to avoid stack too deep error
    ) public returns (uint256 netOutputAmount, uint256 feeAmount, uint256 protocolFeeAmount) {
        // ~~~ Checks ~~~ //

        // calculate the sum of weights of the NFTs to sell
        uint256 weightSum = sumWeightsAndValidateProof(tokenIds, tokenWeights, proof);

        // calculate the net output amount and fee amount
        (netOutputAmount, feeAmount, protocolFeeAmount) = sellQuote(weightSum);

        //  check the nfts are not stolen
        if (useStolenNftOracle) {
            IStolenNftOracle(stolenNftOracle).validateTokensAreNotStolen(nft, tokenIds, stolenNftProofs);
        }

        // ~~~ Effects ~~~ //

        // update the virtual reserves
        virtualBaseTokenReserves -= uint128(netOutputAmount + protocolFeeAmount + feeAmount); // @audit Implement safe cast 
        virtualNftReserves += uint128(weightSum); // @audit Implement safe cast 
```
### RECOMMENDATION
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