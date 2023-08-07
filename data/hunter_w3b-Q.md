## [L-01] Floating-Point Arithmetic

The LiquidationPair.sol contract employs a custom fixed-point arithmetic library `SD59x18` for performing calculations involving floating-point arithmetic. However, using floating-point arithmetic in Solidity can lead to precision issues and unexpected results.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L6

```solidity
import { SD59x18, uEXP_MAX_INPUT, wrap, convert, unwrap } from "prb-math/SD59x18.sol";
```

## [L-02] Timestamp Manipulation in `LiquidationPair.sol` Contract

The LiquidationPair contract relies on the `block.timestamp` to determine auction periods and perform various calculations related to token auctions. However, this implementation introduces a Timestamp Manipulation. Malicious actors could potentially manipulate the block timestamp to exploit arbitrage opportunities and achieve unfair pricing in auctions.

Auction Timing Exploitation: By altering the block timestamp, attackers can initiate or prolong auction periods strategically. This manipulation might enable them to benefit from more favorable token exchange rates or gain an undue advantage over other participants.

Unfair Pricing: Timestamp manipulation can result in distorted pricing mechanisms during auctions. Malicious actors could artificially extend or shorten the auction periods to influence token prices, leading to unfair pricing for other participants.

Arbitrage Opportunities: Manipulating timestamps can create opportunities for arbitrage between different auction periods, leading to inconsistencies and potential losses for honest participants.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L287-L292

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L377-L383

```solidity

  function _getElapsedTime() internal view returns (SD59x18) {
    if (block.timestamp < _lastAuctionTime) {
      return wrap(0);
    }
    return convert(int256(block.timestamp)).sub(convert(int256(uint256(_lastAuctionTime))));
  }


    function _computePeriod() internal view returns (uint256) {
    uint256 _timestamp = block.timestamp;
    if (_timestamp < periodOffset) {
      return 0;
    }
    return (_timestamp - periodOffset) / periodLength;
  }
```

### Recommendation

Use `Block Numbers` Replace block.timestamp with block.number for determining auction periods and conducting time-dependent calculations

## [L-03] Possible Wrong Time Computation in `LiquidationPair::_computePeriod` Function

The `_computePeriod` function computes the current auction period by dividing the time difference between the current block timestamp and periodOffset by periodLength. However, this calculation may lead to incorrect results when periodOffset and periodLength values are set in such a way that the division result is not accurate or an integer.

For instance, if the periodOffset is set to 100 and periodLength is set to 50, the auction periods will not be incremented correctly, and the contract may not function as expected.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L377-L383

```solidity
  function _computePeriod() internal view returns (uint256) {
    uint256 _timestamp = block.timestamp;
    if (_timestamp < periodOffset) {
      return 0;
    }
    return (_timestamp - periodOffset) / periodLength;
  }
```

## [L-04] Missing Error Handling in swapExactAmountOut can lead to Unauthorized Asset Loss

The LiquidationRouter::swapExactAmountOut function doesn't include proper error handling for potential failures that may arise during the execution of the `safeTransferFrom` and `swapExactAmountOut` functions. This omission creates the possibility of loss of funds and unexpected behavior for users.If the `safeTransferFrom` function or `swapExactAmountOut` function encounters an error, such as an out-of-gas condition, an invalid address, or insufficient balance, the `swapExactAmountOut` function will continue executing without reverting the transaction. Consequently, this may result in the loss of the user's funds, and the transaction may not deliver the expected output tokens to the receiver.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L63-L80

```solidity
  function swapExactAmountOut(
    LiquidationPair _liquidationPair,
    address _receiver,
    uint256 _amountOut,
    uint256 _amountInMax
  ) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {
    IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
      msg.sender,
      _liquidationPair.target(),
      _liquidationPair.computeExactAmountIn(_amountOut)
    );

    uint256 amountIn = _liquidationPair.swapExactAmountOut(_receiver, _amountOut, _amountInMax);

    emit SwappedExactAmountOut(_liquidationPair, _receiver, _amountOut, _amountInMax, amountIn);

    return amountIn;
  }
```
