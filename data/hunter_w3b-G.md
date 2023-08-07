# Gas Optimization

# Summary

| Number | Gas Optimization Details                                                                                  | Context |
| :----: | :-------------------------------------------------------------------------------------------------------- | :-----: |
| [G-01] | Avoid contract existence checks by using low level calls                                                  |    6    |
| [G-02] | Functions Guaranteed to revert when called by normal users can be markd payable                           |    5    |
| [G-03] | Stack variable used as a cheaper cache for a state variable is only used once                             |    1    |
| [G-04] | `RngAuction::sequenceOffset` should be cached in stack variables rather than re-reading them from storage |   11    |
| [G-05] | Make the State Variabel outside the loop to save gas                                                      |    2    |
| [G-06] | Using calldata instead of memory for read-only arguments                                                  |    1    |
| [G-07] | Internal functions not called by the contract should be removed to save deployment gas                    |    1    |
| [G-08] | Structs can be packed into fewer storage slots by editing variables                                       |    2    |
| [G-09] | Use nested if and, avoid multiple check combinations `&&`                                                 |    5    |
| [G-10] | Use assembly to write address storage values                                                              |    6    |
| [G-11] | Caching global variables is more expensive than using the actual variable                                 |    1    |
| [G-12] | A `modifier` used only once and not being inherited should be inlined to save gas                         |    3    |
| [G-13] | Do not calculate `constant`                                                                               |    1    |
| [G-14] | `>=` costs less gas than `>`                                                                              |   15    |
| [G-15] | Use do `while` loops instead of `for loops`                                                               |    2    |
| [G-16] | Missing zero address check in `constructor`                                                               |    8    |
| [G-17] | `Ternary` operation is cheaper than `if-else `statement                                                   |    2    |
| [G-18] | Use `assembly` to hash instead of solidity                                                                |    2    |
| [G-19] | Use hardcode address instead `address(this)`                                                              |    6    |
| [G-20] | Use mappings instead of arrays for possible gas save                                                      |    1    |
| [G-21] | Using delete statement can save gas                                                                       |    4    |
| [G-22] | Remove import forge-std                                                                                   |    1    |
| [G-23] | Don't use `_msgSender()` if not supporting EIP-2771                                                       |    1    |
| [G-24] | Make 3 event parameters indexed when possible                                                             |   26    |

## [G‑01] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L141

```solidity
File: src/LiquidationPair.sol

141    return source.targetOf(tokenIn);

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69

```solidity
File: src/LiquidationRouter.sol

69    IERC20(_liquidationPair.tokenIn()).safeTransferFrom(

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180

```solidity
File: src/RngAuction.sol

180      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {


182        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);


185      IERC20(_feeToken).safeIncreaseAllowance(address(rng), _requestFee);

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L226

```solidity
File: src/VaultBooster.sol

226    IERC20(_tokenOut).safeTransfer(_account, _amountOut);

```

## [G‑02] Functions Guaranteed to revert when called by normal users can be markd payable

The `onlyTrustedLiquidationPair` modifier makes a function revert if not called by normal users

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L68

```solidity
File: src/LiquidationRouter.sol

68  ) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {

```

The `onlyOwner` modifier makes a function revert if not called by the address registered as the owner

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142

```solidity
File: src/VaultBooster.sol

142  function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {



188  function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {



217  ) external override onlyPrizeToken(_tokenIn) onlyLiquidationPair(_tokenOut) returns (bool) {




242  function targetOf(address _tokenIn) external view override onlyPrizeToken(_tokenIn) returns (address) {

```

## [G‑03] Stack variable used as a cheaper cache for a state variable is only used once

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14

```solidity
File: src/libraries/ContinuousGDA.sol

14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);

```

## [G‑04] `RngAuction::sequenceOffset` should be cached in stack variables rather than re-reading them from storage

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L365-L375

```solidity
File: src/RngAuction.sol

  function _openSequenceId() internal view returns (uint32) {
    /**
     * Use integer division to calculate a unique ID based off the current timestamp that will remain the same
     * throughout the entire sequence.
     */
    uint64 currentTime = _currentTime();
    if (currentTime < sequenceOffset) {
      return 0;
    }
    return uint32((currentTime - sequenceOffset) / sequencePeriod);
  }
```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 62ebb34..8164f77 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -1,11 +1,14 @@
   function _openSequenceId() internal view returns (uint32) {
+
+    uint64  _SequenceOffset = sequenceOffset;
+
     /**
      * Use integer division to calculate a unique ID based off the current timestamp that will remain the same
      * throughout the entire sequence.
      */
     uint64 currentTime = _currentTime();
-    if (currentTime < sequenceOffset) {
+    if (currentTime < _SequenceOffset) {
       return 0;
     }
-    return uint32((currentTime - sequenceOffset) / sequencePeriod);
+    return uint32((currentTime - _SequenceOffset) / sequencePeriod);
   }

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L381-L387

```solidity
File: src/RngAuction.sol

  function _auctionElapsedTime() internal view returns (uint64) {
    uint64 currentTime = _currentTime();
    if (currentTime < sequenceOffset) {
      return 0;
    }
    return (_currentTime() - sequenceOffset) % sequencePeriod;
  }
```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 4fd9320..98de5e2 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -1,7 +1,9 @@
   function _auctionElapsedTime() internal view returns (uint64) {
+    uint64  _SequenceOffset = sequenceOffset;
+
     uint64 currentTime = _currentTime();
-    if (currentTime < sequenceOffset) {
+    if (currentTime < _SequenceOffset) {
       return 0;
     }
-    return (_currentTime() - sequenceOffset) % sequencePeriod;
+    return (_currentTime() - _SequenceOffset) % sequencePeriod;
   }

```

### `RngAuction::_lastAuction` should be cached in stack variables rather than re-reading them from storage

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L244-L255

```solidity
File: src/RngAuction.sol

  function getLastAuctionResult()
    external
    view
    returns (AuctionResult memory)
  {
    address recipient = _lastAuction.recipient;
    UD2x18 rewardFraction = _lastAuction.rewardFraction;
    return AuctionResult({
      recipient: recipient,
      rewardFraction: rewardFraction
    });
  }
```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index b69878b..eb6a903 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -3,6 +3,10 @@
     view
     returns (AuctionResult memory)
   {
+    RngAuctionResult lastAuction = _lastAuction;

+    address recipient = lastAuction.recipient;
+    UD2x18 rewardFraction = lastAuction.rewardFraction;
-     address recipient = _lastAuction.recipient;
-     UD2x18 rewardFraction = _lastAuction.rewardFraction;
     return AuctionResult({

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L288-L302

```solidity
File: src/RngAuction.sol

  function getRngResults()
    external
    returns (
      uint256 randomNumber, uint64 rngCompletedAt
    )
  {
    RNGInterface rng = _lastAuction.rng;
    uint32 requestId = _lastAuction.rngRequestId;
    return (rng.randomNumber(requestId), rng.completedAt(requestId));
  }

  /// @notice Computes the reward fraction for the given auction elapsed time.
  function computeRewardFraction(uint64 __auctionElapsedTime) external view returns (UD2x18) {
    return _computeRewardFraction(__auctionElapsedTime);
  }
```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 6eb05b8..f1c5428 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -4,8 +4,10 @@
       uint256 randomNumber, uint64 rngCompletedAt
     )
   {
+    RngAuctionResult lastAuction = _lastAuction;

+    RNGInterface rng = lastAuction.rng;
+    uint32 requestId = lastAuction.rngRequestId;
-    RNGInterface rng = _lastAuction.rng;
-    uint32 requestId = _lastAuction.rngRequestId;

     return (rng.randomNumber(requestId), rng.completedAt(requestId));
   }

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L420-L424

```solidity
File: src/RngAuction.sol

  function _isRngComplete() internal view returns (bool) {
    RNGInterface rng = _lastAuction.rng;
    uint32 requestId = _lastAuction.rngRequestId;
    return !_canStartNextSequence() && rng.isRequestComplete(requestId);
  }
```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 81744e6..6a20cd0 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -1,4 +1,6 @@
   function _isRngComplete() internal view returns (bool) {
+    RngAuctionResult lastAuction = _lastAuction;

-    RNGInterface rng = _lastAuction.rng;
-    uint32 requestId = _lastAuction.rngRequestId;

+    RNGInterface rng = lastAuction.rng;
+    uint32 requestId = lastAuction.rngRequestId;
     return !_canStartNextSequence() && rng.isRequestComplete(requestId);

```

### `RngAuctionRelayer::rngAuction` should be cached in stack variables rather than re-reading them from storage

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L31-L38

```solidity
File: src/abstract/RngAuctionRelayer.sol

    function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
        if (!rngAuction.isRngComplete()) revert RngNotCompleted();
        (uint256 randomNumber, uint64 rngCompletedAt) = rngAuction.getRngResults();
        AuctionResult memory results = rngAuction.getLastAuctionResult();
        uint32 sequenceId = rngAuction.openSequenceId();
        results.recipient = remappingOf(results.recipient);
        return abi.encodeWithSelector(IRngAuctionRelayListener.rngComplete.selector, randomNumber, rngCompletedAt, rewardRecipient, sequenceId, results);
    }
```

```diff
diff --git a/RngAuctionRelayer.org.sol b/RngAuctionRelayer.sol
index d196760..c220226 100644
--- a/RngAuctionRelayer.org.sol
+++ b/RngAuctionRelayer.sol
@@ -1,8 +1,9 @@
     function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
-        if (!rngAuction.isRngComplete()) revert RngNotCompleted();
-        (uint256 randomNumber, uint64 rngCompletedAt) = rngAuction.getRngResults();
-        AuctionResult memory results = rngAuction.getLastAuctionResult();
-        uint32 sequenceId = rngAuction.openSequenceId();
+        RngAuction _rngAuction = rngAuction;
+        if (!_rngAuction.isRngComplete()) revert RngNotCompleted();
+        (uint256 randomNumber, uint64 rngCompletedAt) = _rngAuction.getRngResults();
+        AuctionResult memory results = _rngAuction.getLastAuctionResult();
+        uint32 sequenceId = _rngAuction.openSequenceId();
         results.recipient = remappingOf(results.recipient);
         return abi.encodeWithSelector(IRngAuctionRelayListener.rngComplete.selector, randomNumber, rngCompletedAt, rewardRecipient, sequenceId, results);
     }

```

### `RngRelayAuction::prizePool` should be cached in stack variables rather than re-reading them from storage

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L131-L176

```solidity
File: src/RngRelayAuction.sol

  function rngComplete(
    uint256 _randomNumber,
    uint256 _rngCompletedAt,
    address _rewardRecipient,
    uint32 _sequenceId,
    AuctionResult calldata _rngAuctionResult
  ) external returns (bytes32) {
    if (_sequenceHasCompleted(_sequenceId)) revert SequenceAlreadyCompleted();
    uint64 _auctionElapsedSeconds = uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt);
    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();
    // Calculate the reward fraction and set the draw auction results
    UD2x18 rewardFraction = _fractionalReward(_auctionElapsedSeconds);
    _auctionResults.rewardFraction = rewardFraction;
    _auctionResults.recipient = _rewardRecipient;
    _lastSequenceId = _sequenceId;

    AuctionResult[] memory auctionResults = new AuctionResult[](2);
    auctionResults[0] = _rngAuctionResult;
    auctionResults[1] = AuctionResult({
      rewardFraction: rewardFraction,
      recipient: _rewardRecipient
    });

    uint32 drawId = prizePool.closeDraw(_randomNumber);

    uint256 futureReserve = prizePool.reserve() + prizePool.reserveForOpenDraw();
    uint256[] memory _rewards = RewardLib.rewards(auctionResults, futureReserve);

    emit RngSequenceCompleted(
      _sequenceId,
      drawId,
      _rewardRecipient,
      _auctionElapsedSeconds,
      rewardFraction
    );

    for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
    }

    return bytes32(uint(drawId));
  }

```

```diff
diff --git a/RngRelayAuction.org.sol b/RngRelayAuction.sol
index a24c414..b73ad58 100644
--- a/RngRelayAuction.org.sol
+++ b/RngRelayAuction.sol
@@ -5,6 +5,9 @@
     uint32 _sequenceId,
     AuctionResult calldata _rngAuctionResult
   ) external returns (bytes32) {
+
+    PrizePool  _prizePool = prizePool
+
     if (_sequenceHasCompleted(_sequenceId)) revert SequenceAlreadyCompleted();
     uint64 _auctionElapsedSeconds = uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt);
     if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();
@@ -21,9 +24,9 @@
       recipient: _rewardRecipient
     });

-    uint32 drawId = prizePool.closeDraw(_randomNumber);
+    uint32 drawId = _prizePool.closeDraw(_randomNumber);

-    uint256 futureReserve = prizePool.reserve() + prizePool.reserveForOpenDraw();
+    uint256 futureReserve = _prizePool.reserve() + _prizePool.reserveForOpenDraw();
     uint256[] memory _rewards = RewardLib.rewards(auctionResults, futureReserve);

     emit RngSequenceCompleted(
@@ -37,7 +40,7 @@
     for (uint8 i = 0; i < _rewards.length; i++) {
       uint104 _reward = uint104(_rewards[i]);
       if (_reward > 0) {
-        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
+        _prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
         emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
       }

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L181-L184

```solidity
File:  src/RngRelayAuction.sol

  function computeRewards(AuctionResult[] calldata __auctionResults) external returns (uint256[] memory) {
    uint256 totalReserve = prizePool.reserve() + prizePool.reserveForOpenDraw();
    return _computeRewards(__auctionResults, totalReserve);
  }
```

```diff
diff --git a/RngRelayAuction.org.sol b/RngRelayAuction.sol
index 9d63e05..83a0af4 100644
--- a/RngRelayAuction.org.sol
+++ b/RngRelayAuction.sol
@@ -1,4 +1,6 @@
  function computeRewards(AuctionResult[] calldata __auctionResults) external returns (uint256[] memory) {

+    PrizePool  _prizePool = prizePool
+
+    uint256 totalReserve = _prizePool.reserve() + _prizePool.reserveForOpenDraw();
-    uint256 totalReserve = prizePool.reserve() + prizePool.reserveForOpenDraw();
     return _computeRewards(__auctionResults, totalReserve);
   }
```

## [G‑05] Make the State Variabel outside the loop to save gas

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L173

```solidity
File: src/RngRelayAuction.sol

    for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
    }
```

```diff
diff --git a/org.sol b/not.sol
index a76849b..5a505c3 100644
--- a/org.sol
+++ b/not.sol
@@ -1,6 +1,8 @@

+  PrizePool _prizePool = prizePool;
  for (uint8 i = 0; i < _rewards.length; i++) {
       uint104 _reward = uint104(_rewards[i]);
       if (_reward > 0) {
-         prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
+        _prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
         emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
       }

```

## [G‑06] Using calldata instead of memory for read-only arguments

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L59

```solidity
File: src/libraries/RewardLib.sol

59    AuctionResult[] memory _auctionResults,

```

## [G‑07] Internal functions not called by the contract should be removed to save deployment gas

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L31-L38

```solidity
File: src/abstract/RngAuctionRelayer.sol

    function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
        if (!rngAuction.isRngComplete()) revert RngNotCompleted();
        (uint256 randomNumber, uint64 rngCompletedAt) = rngAuction.getRngResults();
        AuctionResult memory results = rngAuction.getLastAuctionResult();
        uint32 sequenceId = rngAuction.openSequenceId();
        results.recipient = remappingOf(results.recipient);
        return abi.encodeWithSelector(IRngAuctionRelayListener.rngComplete.selector, randomNumber, rngCompletedAt, rewardRecipient, sequenceId, results);
    }
```

## [G‑08] Structs can be packed into fewer storage slots by editing variables

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L23-L29

```solidity
File: src/RngAuction.sol

struct RngAuctionResult {
  address recipient;
  UD2x18 rewardFraction;
  uint32 sequenceId;
  RNGInterface rng;
  uint32 rngRequestId;
}
```

Save 1 slot

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 6d7a1ba..a2adaa2 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -1,7 +1,7 @@
 struct RngAuctionResult {
   address recipient;
   UD2x18 rewardFraction;
+  RNGInterface rng;
   uint32 sequenceId;
-  RNGInterface rng;
   uint32 rngRequestId;
 }

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29-L35

```solidity
File: src/VaultBooster.sol

struct Boost {
  address liquidationPair;
  UD2x18 multiplierOfTotalSupplyPerSecond;
  uint96 tokensPerSecond;
  uint144 available;
  uint48 lastAccruedAt;
}
```

The change swaps the positions of the lastAccruedAt and multiplierOfTotalSupplyPerSecond variables. Since uint48 occupies 48 bits and UD2x18 (assuming it's a 2-decimal fixed-point number) occupies 36 bits (18 bits for each decimal) and address occupies 160 bits, these three variables together will take up 244 bits, which fits within a single storage slot (256 bits).

```diff
diff --git a/VaultBooster.org.sol b/VaultBooster.sol
index dfa2f96..a430c42 100644
--- a/VaultBooster.org.sol
+++ b/VaultBooster.sol
@@ -1,7 +1,7 @@
 struct Boost {
   address liquidationPair;
+  uint48 lastAccruedAt;
   UD2x18 multiplierOfTotalSupplyPerSecond;
   uint96 tokensPerSecond;
   uint144 available;
-  uint48 lastAccruedAt;
 }

```

## [G‑09] Use nested if and, avoid multiple check combinations `&&`

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332

```solidity
File: src/LiquidationPair.sol

332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

```

```diff
diff --git a/LiquidationPair.org.sol b/LiquidationPair.sol
index f7586f6..85eb4e3 100644
--- a/LiquidationPair.org.sol
+++ b/LiquidationPair.sol
@@ -1,6 +1,8 @@
   function _updateAuction(uint256 __period) internal {
-    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {
+    if (_amountInForPeriod > 0 ){
+         if ( _amountOutForPeriod > 0) {
       // if we sold something, then update the previous non-zero amount
       _lastNonZeroAmountIn = _amountInForPeriod;
       _lastNonZeroAmountOut = _amountOutForPeriod;
+      }
     }

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L179

```solidity
File: src/RngAuction.sol

179    if (_feeToken != address(0) && _requestFee > 0) {

```

```diff
diff --git a/RngAuction.org.sol b/RngAuction.sol
index 2d7c938..2c65f0f 100644
--- a/RngAuction.org.sol
+++ b/RngAuction.sol
@@ -1,3 +1,4 @@
     (address _feeToken, uint256 _requestFee) = rng.getRequestFee();
-    if (_feeToken != address(0) && _requestFee > 0) {
+    if (_feeToken != address(0)) {
+        if( _requestFee > 0) {
       if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {

```

## [G‑10] Use assembly to write address storage values

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L106

```solidity
File: src/LiquidationPair.sol

106    tokenIn = _tokenIn;
107    tokenOut = _tokenOut;
```

```diff
diff --git a/LiquidationPair.org.sol b/LiquidationPair.sol
index 2a1892d..216c674 100644
--- a/LiquidationPair.org.sol
+++ b/LiquidationPair.sol
@@ -11,9 +11,11 @@
     uint256 _minimumAuctionAmount
   ) {
     source = _source;
-    tokenIn = _tokenIn;
-    tokenOut = _tokenOut;
+    assembly {
+        sstore(tokenIn.slot, _tokenIn)
+        sstore(tokenOut.slot, _tokenOut)
+    }
     decayConstant = _decayConstant;
     periodLength = _periodLength;
     periodOffset = _periodOffset;

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L116

```solidity
File: src/RngRelayAuction.sol

116    rngAuctionRelayer = _rngAuctionRelayer;

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L125

```solidity
File: src/VaultBooster.sol

125    vault = _vault;

```

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L106

```solidity
File: src/RemoteOwner.sol

106    _originChainOwner = _newOriginChainOwner;

```

## [G‑11] Caching global variables is more expensive than using the actual variable

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L378

```solidity
File:  src/LiquidationPair.sol

378    uint256 _timestamp = block.timestamp;

```

## [G‑12] A modifier used only once and not being inherited should be inlined to save gas

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L84-L89

```solidity
File: src/LiquidationRouter.sol

  modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) {
    if (!_liquidationPairFactory.deployedPairs(_liquidationPair)) {
      revert UnknownLiquidationPair(_liquidationPair);
    }
    _;
  }

```

The above Modifier `onlyTrustedLiquidationPair` only use in this function It should be inlined to save gas

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L63-L80

```solidity
File: src/LiquidationRouter.sol

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

## [G‑13] Do not calculate `constant`

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14

```solidity
File:src/libraries/ContinuousGDA.sol

14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);

```

## [G-14] `>=` costs less gas than >

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38

```solidity
File: src/libraries/ContinuousGDA.sol

38    if (_emissionRate.unwrap() > 1e18) {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol

```solidity
File: src/LiquidationPair.sol

114    if (_decayConstant.mul(period59).unwrap() > uEXP_MAX_INPUT) {

218    if (swapAmountIn > _amountInMax) {

302    if (_amountOut > maxOut) {

332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol

```solidity
File: src/RngAuction.sol

149    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));

176    if (_auctionElapsedTimeSeconds > auctionDuration) revert AuctionExpired();

179    if (_feeToken != address(0) && _requestFee > 0) {

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol

```solidity
File: src/RngRelayAuction.sol

113    if (auctionTargetTime_ > auctionDurationSeconds_) {


140    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();


169      if (_reward > 0) {

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol

```solidity
File: src/VaultBooster.sol

143    if (_initialAvailable > 0) {

219    if (_amountOut > amountAvailable) {

269    if (boost.tokensPerSecond > 0) {

272    if (boost.multiplierOfTotalSupplyPerSecond.unwrap() > 0) {

```

## [G-15] Use do `while` loops instead of `for loops`

https://code4rena.com/reports/2023-05-ajna#g-09-use-do-while-loops-instead-of-for-loops

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65-L68

```solidity
File: libraries/RewardLib.sol

    for (uint256 i; i < _auctionResultsLength; i++) {
      _rewards[i] = reward(_auctionResults[i], remainingReserve);
      remainingReserve = remainingReserve - _rewards[i];
    }

```

```diff
diff --git a/RewardLib.org.sol b/RewardLib.sol
index c7237a8..f006fb3 100644
--- a/RewardLib.org.sol
+++ b/RewardLib.sol
@@ -1,10 +1,11 @@
      rewardFraction
     );

-    for (uint8 i = 0; i < _rewards.length; i++) {
+    uint8 i = 0;
+    do {
       uint104 _reward = uint104(_rewards[i]);
       if (_reward > 0) {
         prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
         emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
       }
-    }
+    } while (i < _rewards.length);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L172

```solidity
File: src/RngRelayAuction.sol

    for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
```

## [G-16] Missing zero address check in `constructor`

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L95-L96

```solidity
File: src/LiquidationPair.sol

   constructor(
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
   ) {
     source = _source;
     tokenIn = _tokenIn;
     tokenOut = _tokenOut;
     decayConstant = _decayConstant;
     periodLength = _periodLength;
     periodOffset = _periodOffset;
     targetFirstSaleTime = _targetFirstSaleTime;

     SD59x18 period59 = convert(int256(uint256(_periodLength)));
     if (_decayConstant.mul(period59).unwrap() > uEXP_MAX_INPUT) {
       revert DecayConstantTooLarge(wrap(uEXP_MAX_INPUT).div(period59), _decayConstant);
     }

     if (targetFirstSaleTime >= periodLength) {
       revert TargetFirstSaleTimeLtPeriodLength(targetFirstSaleTime, periodLength);
     }
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L142-L142

```solidity
File: src/RngAuction.sol

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
     sequencePeriod = sequencePeriod_;
     sequenceOffset = sequenceOffset_;
     auctionDuration = auctionDurationSeconds_;
     auctionTargetTime = auctionTargetTime_;
     _auctionTargetTimeFraction = intoUD2x18(convert(uint(auctionTargetTime_)).div(convert(uint(auctionDurationSeconds_))));
     _setNextRngService(rng_);
   }
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L119-L123

```solidity
File: src/VaultBooster.sol

   constructor(
     PrizePool _prizePool,
     address _vault,
     address _owner
   ) Ownable(_owner) {
     prizePool = _prizePool;
     twabController = prizePool.twabController();
     vault = _vault;
   }
```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L53-L54

```solidity
File: src/RemoteOwner.sol

   constructor(
     uint256 originChainId_,
     address executor_,
     address __originChainOwner
   ) ExecutorAware(executor_) {
     if (originChainId_ == 0) revert OriginChainIdZero();
     _originChainId = originChainId_;
     _setOriginChainOwner(__originChainOwner);
   }
```

## [G-17] `Ternary` operation is cheaper than `if-else `statement

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L33-L37

```solidity
File: src/abstract/AddressRemapper.sol

    if (_destinationAddress[_addr] == address(0)) {
      return _addr;
    } else {
      return _destinationAddress[_addr];
    }

```

```diff
diff --git a/AddressRemapper.org.sol b/AddressRemapper.sol
index e47b41d..c21492c 100644
--- a/AddressRemapper.org.sol
+++ b/AddressRemapper.sol
@@ -1,7 +1,3 @@
  function remappingOf(address _addr) public view returns (address) {
-    if (_destinationAddress[_addr] == address(0)) {
-      return _addr;
-    } else {
-      return _destinationAddress[_addr];
-    }
+    return (_destinationAddress[_addr] == address(0)) ? _addr : _destinationAddress[_addr];

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L23-L45

```solidity
File: src/libraries/ContinuousGDA.sol

  function purchasePrice(
    SD59x18 _amount,
    SD59x18 _emissionRate,
    SD59x18 _k,
    SD59x18 _decayConstant,
    SD59x18 _timeSinceLastAuctionStart
  ) internal pure returns (SD59x18) {
    if (_amount.unwrap() == 0) {
      return SD59x18.wrap(0);
    }
    SD59x18 topE = _decayConstant.mul(_amount).div(_emissionRate);
    topE = topE.exp().sub(ONE);
    SD59x18 bottomE = _decayConstant.mul(_timeSinceLastAuctionStart);
    bottomE = bottomE.exp();
    SD59x18 result;
    if (_emissionRate.unwrap() > 1e18) {
      result = _k.div(_emissionRate).mul(topE).div(bottomE);
    } else {
      result = _k.mul(topE.div(_emissionRate.mul(bottomE)));
    }
    return result;
  }

```

```diff
diff --git a/ContinuousGDA.org.sol b/ContinuousGDA.sol
index 3f1a6b1..fa259e6 100644
--- a/ContinuousGDA.org.sol
+++ b/ContinuousGDA.sol
@@ -13,10 +13,6 @@
     SD59x18 bottomE = _decayConstant.mul(_timeSinceLastAuctionStart);
     bottomE = bottomE.exp();
     SD59x18 result;
-    if (_emissionRate.unwrap() > 1e18) {
-      result = _k.div(_emissionRate).mul(topE).div(bottomE);
-    } else {
-      result = _k.mul(topE.div(_emissionRate.mul(bottomE)));
-    }
-    return result;
+
+    return result = (_emissionRate.unwrap() > 1e18) ? _k.div(_emissionRate).mul(topE).div(bottomE) : _k.mul(topE.div(_emissionRate.mul(bottomE)));
   }

```

## [G-18] Use `assembly` to hash instead of solidity

A good examples to how to use assembly for optimization
[Resource](https://medium.com/@bloqarl/solidity-gas-optimization-tips-with-assembly-you-havent-heard-yet-1381c77ff078)

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol#L8-L13

```solidity
File: src/libraries/RemoteOwnerCallEncoder.sol

        return abi.encodeWithSelector(
            RemoteOwner.execute.selector,
            target,
            value,
            data
        );
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L37

```solidity
File: src/abstract/RngAuctionRelayer.sol

37        return abi.encodeWithSelector(IRngAuctionRelayListener.rngComplete.selector, randomNumber, rngCompletedAt, rewardRecipient, sequenceId, results);

```

## [G-19] Use hardcode address instead `address(this)`

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144

```solidity
File: src/VaultBooster.sol

144      uint256 balance = _token.balanceOf(address(this));

173    _token.safeTransferFrom(msg.sender, address(this), _amount);

190    uint256 availableBalance = _token.balanceOf(address(this));

276    uint256 availableBalance = _tokenOut.balanceOf(address(this));

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180

```solidity
File: src/RngAuction.sol


180      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {

182        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

```

## [G-20] Use mappings instead of arrays for possible gas save

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L43

```solidity
File: src/LiquidationPairFactory.sol

43  LiquidationPair[] public allPairs;

```

## [G-21] Using delete statement can save gas

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L279

```solidity
File:  src/LiquidationPair.sol

279      amount = 0;


337    _amountInForPeriod = 0;

338    _amountOutForPeriod = 0;

```

## [G-22] Remove import forge-std

It's used to print the values of variables while running tests to help debug and see what's happening inside your contracts but since it's a development tool, it serves no purpose on mainnet. Also, the remember to remove the usage of calls that use forge-std when removing of the import of forge-std.

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L4

```solidity
File: src/VaultBooster.sol

4   import "forge-std/console2.sol";

```

## [G-23] Don't use ``_msgSender()` if not supporting EIP-2771

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L120

```solidity
File: src/RemoteOwner.sol

120    if (_msgSender() != address(_originChainOwner)) revert OriginSenderNotOwner(_msgSender());

```

## [G-24] Make 3 event parameters indexed when possible

Events are used to emit information about state changes in a contract. When defining an event, it's important to consider the gas cost of emitting the event, as well as the efficiency of searching for events in the Ethereum blockchain.

Making event parameters indexed can help reduce the gas cost of emitting events and improve the efficiency of searching for events. When an event parameter is marked as indexed, its value is stored in a separate data structure called the event topic, which allows

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L26-L38

```solidity
File: src/LiquidationPairFactory.sol

  event PairCreated(
    LiquidationPair indexed pair,
    ILiquidationSource source,
    address tokenIn,
    address tokenOut,
    uint32 periodLength,
    uint32 periodOffset,
    uint32 targetFirstSaleTime,
    SD59x18 decayConstant,
    uint112 initialAmountIn,
    uint112 initialAmountOut,
    uint256 minimumAuctionAmount
  );
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L30-L36

```solidity
File: src/LiquidationRouter.sol

 event SwappedExactAmountOut(
    LiquidationPair indexed liquidationPair,
    address indexed receiver,
    uint256 amountOut,
    uint256 amountInMax,
    uint256 amountIn
  );
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol

```solidity
File: src/RngRelayAuction.sol

  event AuctionRewardDistributed(
    uint32 indexed sequenceId,
    address indexed recipient,
    uint32 index,
    uint256 reward
  );



  event RngSequenceCompleted(
    uint32 indexed sequenceId,
    uint32 indexed drawId,
    address indexed rewardRecipient,
    uint64 auctionElapsedSeconds,
    UD2x18 rewardFraction
  );
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol

```solidity
File: src/VaultBooster.sol


 event SetBoost(
    IERC20 indexed token,
    address liquidationPair,
    UD2x18 multiplierOfTotalSupplyPerSecond,
    uint96 tokensPerSecond,
    uint144 initialAvailable,
    uint48 lastAccruedAt
  );


    event Liquidated(
    IERC20 indexed token,
    address indexed from,
    uint256 amountIn,
    uint256 amountOut,
    uint256 availableBoostBalance
  );

```

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L36

```solidity
File: src/RemoteOwner.sol

  event OriginChainOwnerSet(address owner);

```
