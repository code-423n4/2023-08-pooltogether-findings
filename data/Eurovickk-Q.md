https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol

Mapping Key: The mapping mapping(LiquidationPair => bool) public deployedPairs; uses LiquidationPair instances as keys. This might not work as expected because contracts are not valid keys for mappings due to the way hashing works. You should use the address of the contract instead.


pragma solidity 0.8.19;

import "./LiquidationPair.sol";

contract LiquidationPairFactory {
   
 // Change the mapping to use address as keys
    mapping(address => bool) public deployedPairs;

    function createPair(
        ILiquidationSource _source,
        address _tokenIn,
        address _tokenOut,
        uint32 _periodLength,
        uint32 _periodOffset,
        uint32 _targetFirstSaleTime,
        SD59x18 _decayConstant,
        uint112 _initialAmountIn,
        uint112 _initialAmountOut,
        uint256 _minimumAuctionAmount
    ) external returns (LiquidationPair) {
        LiquidationPair _liquidationPair = new LiquidationPair(
            _source,
            _tokenIn,
            _tokenOut,
            _periodLength,
            _periodOffset,
            _targetFirstSaleTime,
            _decayConstant,
            _initialAmountIn,
            _initialAmountOut,
            _minimumAuctionAmount
        );

        // Use the address of the new pair as the key
        deployedPairs[address(_liquidationPair)] = true;

        // ... rest of the code ...
    }

    // ... rest of the code ...
}

In this modified version, we use the address of the _liquidationPair contract instance as the key for the deployedPairs mapping. This is possible because contract addresses are deterministic based on the creator's address and the creator's nonce. By using the contract address as the key, you ensure that each deployed contract can be tracked within the mapping.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol

Error Handling in swapExactAmountOut: In the swapExactAmountOut function, you call IERC20(_liquidationPair.tokenIn()).safeTransferFrom(...) to transfer input tokens from the sender to the target address of the LiquidationPair. You need to make sure that the sender has approved the LiquidationRouter to spend their tokens before calling this function, otherwise, the transfer might fail. You might want to include a check to ensure that the allowance is sufficient.
