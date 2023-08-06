In `LiquidationPair.sol` contract in `_computePeriod()` function it's cheaper not to cache block.timestamp.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L377-L384

```
function _computePeriod() internal view returns (uint256) {
--  uint256 _timestamp = block.timestamp;
++  if (block.timestamp < periodOffset) {
      return 0;
    }
++  return (block.timestamp - periodOffset) / periodLength;
  }
```


In `LiquidationPair.sol` it is cheaper to put `_lastNonZeroAmountIn` and `_lastNonZeroAmountOut` in one slot by converting them into uint128.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L57-L60

```
/// @notice The last non-zero total tokens in for an auction. This is used to configure the target price for the next auction.
--  uint112 _lastNonZeroAmountIn;
++  uint128 _lastNonZeroAmountIn;

  /// @notice The last non-zero total tokens out for an auction.  This is used to configure the target price for the next auction.
--  uint112 _lastNonZeroAmountOut;
++  uint128 _lastNonZeroAmountOut;
```
