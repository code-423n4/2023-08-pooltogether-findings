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
## Avoid caching state variables that will only be used once
Caching state variables that will only be used once incurs more gas and not recommended.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L244-L255

```diff
  function getLastAuctionResult()
    external
    view
    returns (AuctionResult memory)
  {
-    address recipient = _lastAuction.recipient;
-    UD2x18 rewardFraction = _lastAuction.rewardFraction;
    return AuctionResult({
-      recipient: recipient,
+      recipient: _lastAuction.recipient,
-      rewardFraction: rewardFraction
+      rewardFraction: _lastAuction.rewardFraction
    });
  }
```
## Consider using descriptive constants when passing zero as a function argument
Passing zero as a function argument can sometimes result in a security issue (e.g. passing zero as the slippage parameter, fees, token amounts ...). Consider using a constant variable with a descriptive name, so it's clear that the argument is intentionally being used, and for the right reasons.

Here is one specific instance found.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol#L65

```solidity
            RemoteOwnerCallEncoder.encodeCalldata(address(_remoteRngAuctionRelayListener), 0, listenerCalldata)
```
## Enhanced reward distribution to safeguarde against reserve depletion in auction calculations
The `rewards` function in the provided `RewardLib` library calculates the rewards for a series of auctions based on a given reserve. A potential issue is that midway through the calculations, the `remainingReserve` might become insufficient to provide the required reward for an auction, potentially causing underflows or incorrect behavior. To address this concern, it's recommended to implement a check to ensure that each reward doesn't exceed the `remainingReserve`, and if it does, the function should gracefully exit the loop to avoid unexpected results.

Here's a suggested fix:

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L58-L70

```diff
  function rewards(
    AuctionResult[] memory _auctionResults,
    uint256 _reserve
  ) internal pure returns (uint256[] memory) {
    uint256 remainingReserve = _reserve;
    uint256 _auctionResultsLength = _auctionResults.length;
    uint256[] memory _rewards = new uint256[](_auctionResultsLength);
    for (uint256 i; i < _auctionResultsLength; i++) {
-      _rewards[i] = reward(_auctionResults[i], remainingReserve);
-      remainingReserve = remainingReserve - _rewards[i];

+      uint256 calculatedReward = reward(_auctionResults[i], remainingReserve);
+      if (calculatedReward > remainingReserve) {
+            break; // Stop if there's not enough remaining reserve.
+        }
+      _rewards[i] = calculatedReward;
+      remainingReserve = remainingReserve - calculatedReward;
    }
    return _rewards;
  }
```
## Activate the optimizer
Before deploying your contract, activate the optimizer when compiling using “solc --optimize --bin sourceFile.sol”. By default, the optimizer will optimize the contract assuming it is called 200 times across its lifetime. If you want the initial contract deployment to be cheaper and the later function executions to be more expensive, set it to “ --optimize-runs=1”. Conversely, if you expect many transactions and do not care for higher deployment cost and output size, set “--optimize-runs” to a high number.

```
module.exports = {
solidity: {
version: "0.8.19",
settings: {
  optimizer: {
    enabled: true,
    runs: 1000,
  },
},
},
};
```
Kindly visit the following site for further information:

https://docs.soliditylang.org/en/v0.5.4/using-the-compiler.html#using-the-commandline-compiler

Here is one particular example of instance on opcode comparison that delineates the gas saving mechanism:

```
for !=0 before optimization
PUSH1 0x00
DUP2
EQ
ISZERO
PUSH1 [cont offset]
JUMPI

after optimization
DUP1
PUSH1 [revert offset]
JUMPI
```
Disclaimer: There have been several bugs with security implications related to optimizations. For this reason, Solidity compiler optimizations are disabled by default, and it is unclear how many contracts in the wild actually use them. Therefore, it is unclear how well they are being tested and exercised. High-severity security issues due to optimization bugs have occurred in the past. A high-severity bug in the emscripten -generated solc-js compiler used by Truffle and Remix persisted until late 2018. The fix for this bug was not reported in the Solidity CHANGELOG. Another high-severity optimization bug resulting in incorrect bit shift results was patched in Solidity 0.5.6. Please measure the gas savings from optimizations, and carefully weigh them against the possibility of an optimization-related bug. Also, monitor the development and adoption of Solidity compiler optimizations to assess their maturity.