## Custom error misleading name
The name of the following custom error should be renamed as follows to be more in line of its intended purpose:

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L12

```diff
- error TargetFirstSaleTimeLtPeriodLength(uint passedTargetSaleTime, uint periodLength);
+ error TargetFirstSaleTimeGtPeriodLength(uint passedTargetSaleTime, uint periodLength);
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L118-L120

```diff
    if (targetFirstSaleTime >= periodLength) {
-      revert TargetFirstSaleTimeLtPeriodLength(targetFirstSaleTime, periodLength);
+      revert TargetFirstSaleTimeGtPeriodLength(targetFirstSaleTime, periodLength);
    }
```
## Inconsistent typecasting
The typecast below should be corrected as follows since `Conversions.convert` takes in `int256` parameter. Additionally, down-casting `periodLength` of `int` or `int256` to `int32` could lead to overflow issue.  

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L282

```diff
-    return convert(int256(amount)).div(convert(int32(int(periodLength))));
+    return convert(int256(amount)).div(convert(int256(periodLength)));
```
## Avoid caching global variables
There is no gas benefit caching a global variable like, `msg.sender`, `block.timestamp` etc. Moreover, the cached `timestamp` is only used once in either the `if` block or the last line of `return`, which further defeat the purpose of variable caching.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L375-L383

```diff
  /// @notice Computes the current auction period
  /// @return the current period
  function _computePeriod() internal view returns (uint256) {
-    uint256 _timestamp = block.timestamp;
-    if (_timestamp < periodOffset) {
+    if (block.timestamp < periodOffset) {
      return 0;
    }
-    return (_timestamp - periodOffset) / periodLength;
+    return (block.timestamp - periodOffset) / periodLength;
  }
```
## Missing constructor check for `AuctionDurationGteSequencePeriod()`
The error `RngAuction.AuctionDurationGteSequencePeriod` has been declared but never used. It should be utilized to implement the following missing check.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L140-L156

```diff
  constructor(
    RNGInterface rng_,
    address owner_,
    uint64 sequencePeriod_,
    uint64 sequenceOffset_,
    uint64 auctionDurationSeconds_,
    uint64 auctionTargetTime_
  ) Ownable(owner_) {
    if (sequencePeriod_ == 0) revert SequencePeriodZero();
    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));
+    if (auctionDurationSeconds_ > sequencePeriod_) revert AuctionDurationGteSequencePeriod(uint64(auctionDurationSeconds_), uint64(sequencePeriod_));
    sequencePeriod = sequencePeriod_;
    sequenceOffset = sequenceOffset_;
    auctionDuration = auctionDurationSeconds_;
    auctionTargetTime = auctionTargetTime_;
    _auctionTargetTimeFraction = intoUD2x18(convert(uint(auctionTargetTime_)).div(convert(uint(auctionDurationSeconds_))));
    _setNextRngService(rng_);
  }
```
## Grammatical and spelling errors
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L60

```solidity
      @ audit `exactly` --> `exact`
  /// @param _amountOut The `exactly` amount of output tokens expected
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L74

```solidity
      @ audit `wil` --> `will`
  /// @notice The PrizePool whose draw `wil` be closed.
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L48

```solidity
      @ audit `Not` --> `Note`
  /// @dev `Not` that the reward fractions compound
```
## Reverse order of `if else` logic
The order of the `if else` logic seems to have been reversed. Depending on the values of `_emissionRate.unwrap()` and `_emissionRate` involved, this could be crucial enough to make the logic malfunction.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38-L42

```
    if (_emissionRate.unwrap() > 1e18) {
      result = _k.div(_emissionRate).mul(topE).div(bottomE);
    } else {
      result = _k.mul(topE.div(_emissionRate.mul(bottomE)));
    }
```