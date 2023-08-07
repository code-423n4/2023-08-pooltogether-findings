

# Summary for GAS findings


| Number | Issue | Instances |
|--------|-------|-----------|
|[G-01]| Structs can be packed into fewer storage slots by editing variables  | 2 | 
|[G-02]| Functions guaranteed to revert when called by normal users can be marked payable  | 4 | 
|[G-03]| Using delete statement can save gas | 3 | 
|[G-04]| Missing zero address check in constructor  | 4 | 
|[G-05]| `>=` costs less gas than >  | 8 | 
|[G-06]| Use do while loops instead of for loops  | 2 | 
|[G-07]| Do not calculate constant | 1 | 
|[G-08]| use mappings instead of arrays  | 1 | 
|[G-09]| Use assembly to write address storage values  | 5 | 
|[G-10]| Avoid contract existence checks by using low level calls  | 6 | 
|[G-11]| Declare the State Variabel outside the loop to save gas  | 1 | 
|[G-12]| Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it)  | 1 | 
|[G-13]| Ternary operation is cheaper than if-else statement  | 2 | 
|[G-14]| Expensive operation inside a for loop  | 2 | 
|[G-15]| Use hardcode address instead address(this)  | 6 | 
|[G-16]| Public Functions To External  | 2 | 
|[G-17]| Using bools for storage incurs overhead  | 1 | 
|[G-18]| Use assembly to perform efficient back-to-back calls  | 1 | 
|[G-19]| Internal functions not called by the contract should be removed to save deployment gas  | 1 | 
|[G-20]| Forgo internal function to save 1 STATICCALL  | 1 | 
|[G-21]| With assembly, .call (bool success) transfer can be done gas-optimized  | 1 | 
|[G-22]| Amounts should be checked for 0 before calling a transfer   | 2 | 
|[G-23]| For loops in public or external functions should be avoided due to high gas costs and possible DOS  | 1 | 




## [G‑01] Structs can be packed into fewer storage slots by editing variables



```solidity
File: src/RngAuction.sol

29    struct RngAuctionResult {
       address recipient;
       UD2x18 rewardFraction;
       uint32 sequenceId;
       RNGInterface rng;
       uint32 rngRequestId;
     }
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L23-L29



```solidity
File: src/VaultBooster.sol

29    struct Boost {
        address liquidationPair;
        UD2x18 multiplierOfTotalSupplyPerSecond;
        uint96 tokensPerSecond;
        uint144 available;
        uint48 lastAccruedAt;
}
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29-L35


## [G-02] Functions guaranteed to revert when called by normal users can be marked payable

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

```solidity
file: src/RngAuction.sol

347   function setNextRngService(RNGInterface _rngService) external onlyOwner {

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L347

```solidity
file: src/VaultBooster.sol

142    function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {

188    function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {

242    function targetOf(address _tokenIn) external view override onlyPrizeToken(_tokenIn) returns (address) {

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142


## [G-03] Using delete statement can save gas


```solidity
File:  src/LiquidationPair.sol

279      amount = 0;

337    _amountInForPeriod = 0;

338    _amountOutForPeriod = 0;

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L279

## [G-04] Missing zero address check in constructor

Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.


```solidity
File: src/LiquidationPair.sol

   constructor(
     ILiquidationSource _source,
     address _tokenIn, // <= FOUND
     address _tokenOut, // <= FOUND
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
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L95-L96


```solidity
File: src/RngAuction.sol

   constructor(
     RNGInterface rng_,
     address owner_,     // <= FOUND
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

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L142-L142




```solidity
File: src/VaultBooster.sol

   constructor(
     PrizePool _prizePool, // <= FOUND
     address _vault, // <= FOUND
     address _owner // <= FOUND
   ) Ownable(_owner) {
     prizePool = _prizePool; // <= FOUND
     twabController = prizePool.twabController();
     vault = _vault;
   }
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L119-L123



```solidity
File: src/RemoteOwner.sol

   constructor(
     uint256 originChainId_,
     address executor_, // <= FOUND
     address __originChainOwner // <= FOUND
   ) ExecutorAware(executor_) {
     if (originChainId_ == 0) revert OriginChainIdZero();
     _originChainId = originChainId_;
     _setOriginChainOwner(__originChainOwner);
   }
```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L53-L54

## [G-05] `>=` costs less gas than >

```solidity
File: src/RngRelayAuction.sol

113    if (auctionTargetTime_ > auctionDurationSeconds_) {


140    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();


```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol



```solidity
File: src/libraries/ContinuousGDA.sol

38    if (_emissionRate.unwrap() > 1e18) {
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38


```solidity
File: src/LiquidationPair.sol

114    if (_decayConstant.mul(period59).unwrap() > uEXP_MAX_INPUT) {

218    if (swapAmountIn > _amountInMax) {

302    if (_amountOut > maxOut) {


```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol


```solidity
File: src/RngAuction.sol

149    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));

176    if (_auctionElapsedTimeSeconds > auctionDuration) revert AuctionExpired();

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol

## [G-06] Use do while loops instead of for loops



```solidity
File: libraries/RewardLib.sol

65   for (uint256 i; i < _auctionResultsLength; i++) {
      _rewards[i] = reward(_auctionResults[i], remainingReserve);
      remainingReserve = remainingReserve - _rewards[i];
    }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65-L68



```solidity
File: src/RngRelayAuction.sol

167   for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L172

## [G‑07] Do not calculate constant

```solidity
File:src/libraries/ContinuousGDA.sol

14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14


## [G-08] use mappings instead of arrays


```solidity
File: src/LiquidationPairFactory.sol

43  LiquidationPair[] public allPairs;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L43

## [G‑09] Use assembly to write address storage values

```solidity
File: src/LiquidationPair.sol

106    tokenIn = _tokenIn;

107    tokenOut = _tokenOut;
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L106



```solidity
File: src/RngRelayAuction.sol

116    rngAuctionRelayer = _rngAuctionRelayer;

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L116



```solidity
File: src/VaultBooster.sol

125    vault = _vault;

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L125



```solidity
File: src/RemoteOwner.sol

106    _originChainOwner = _newOriginChainOwner;

```
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L106


## [G‑10] Avoid contract existence checks by using low level calls

Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.



```solidity
File: src/LiquidationPair.sol

141    return source.targetOf(tokenIn);

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L141



```solidity
File: src/LiquidationRouter.sol

69    IERC20(_liquidationPair.tokenIn()).safeTransferFrom(

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69



```solidity
File: src/RngAuction.sol

180      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {


182        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);


185      IERC20(_feeToken).safeIncreaseAllowance(address(rng), _requestFee);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180



```solidity
File: src/VaultBooster.sol

226    IERC20(_tokenOut).safeTransfer(_account, _amountOut);

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L226


## [G‑11] Declare the State Variabel outside the loop to save gas


```solidity
File: src/RngRelayAuction.sol

167    for (uint8 i = 0; i < _rewards.length; i++) {
          uint104 _reward = uint104(_rewards[i]);
          if (_reward > 0) {
            prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
            emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
          }
        }
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L173


## [G‑12] Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it)


```solidity
File:  src/LiquidationPair.sol

378    uint256 _timestamp = block.timestamp;

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L378

## [G-13] Ternary operation is cheaper than if-else statement

```solidity
File: src/abstract/AddressRemapper.sol

33    if (_destinationAddress[_addr] == address(0)) {
          return _addr;
        } else {
          return _destinationAddress[_addr];
        }
    
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L33-L37


```solidity
File: src/libraries/ContinuousGDA.sol

23     function purchasePrice(
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

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L23-L45

## [G-14] Expensive operation inside a for loop


```solidity
file:  RngRelayAuction.sol

167      for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
    }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L173

```solidity
file:  src/libraries/RewardLib.sol

65       for (uint256 i; i < _auctionResultsLength; i++) {
      _rewards[i] = reward(_auctionResults[i], remainingReserve);
      remainingReserve = remainingReserve - _rewards[i];
    }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65-L68

## [G-15] Use hardcode address instead address(this)

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.

References:

https://book.getfoundry.sh/reference/forge-std/compute-create-address
https://twitter.com/transmissions11/status/1518507047943245824


```solidity
file:  src/RngAuction.sol

180    if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {

182    IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180

```solidity
file:  src/VaultBooster.sol

144     uint256 balance = _token.balanceOf(address(this));

173    _token.safeTransferFrom(msg.sender, address(this), _amount);

190     uint256 availableBalance = _token.balanceOf(address(this));

276     uint256 availableBalance = _tokenOut.balanceOf(address(this));

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144

## [G-16]	Public Functions To External

```solidity
file:   src/abstract/AddressRemapper.sol

32      function remappingOf(address _addr) public view returns (address) {

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L32


```solidity
file: src/LiquidationRouter.sol

84    modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L84


## [G-17] Using bools for storage incurs overhead

Both state variables can potentially be set back and forth from true and false. This would result in a Gsset (20000 gas) everytime the values are set to true from false. We can instead use uint(1) in place of true and uint(2) in place of false and pay the Gsset (20000 gas) once during deployment (to set the slot to uint(1). This would save two Gsset (20000 gas). However, a more efficient mitigation would be to pack the variables into a slot with other variables that would inevitably be written to. Please see this finding for a more efficient solution.


```solidity
file:  src/LiquidationPairFactory.sol

51    mapping(LiquidationPair => bool) public deployedPairs;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L51

## [G-18] Use assembly to perform efficient back-to-back calls

If similar external calls are performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer), which can potentially allow us to avoid memory expansion costs. In this case we are also able to efficiently store both function signatures together in memory as one word, saving one MLOAD in the process.

Note: In order to do this optimization safely we will cache and restore the free memory pointer after we are done with our function calls

```solidity
file:  src/libraries/RewardLib.sol

32     UD60x18 t = UD60x18.wrap(_targetTimeFraction.unwrap());
33     UD60x18 r = UD60x18.wrap(_targetRewardFraction.unwrap());

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L32-L33

## [G‑19] Internal functions not called by the contract should be removed to save deployment gas

```solidity
File: src/abstract/RngAuctionRelayer.sol

31   function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
            if (!rngAuction.isRngComplete()) revert RngNotCompleted();
            (uint256 randomNumber, uint64 rngCompletedAt) = rngAuction.getRngResults();
            AuctionResult memory results = rngAuction.getLastAuctionResult();
            uint32 sequenceId = rngAuction.openSequenceId();
            results.recipient = remappingOf(results.recipient);
            return abi.encodeWithSelector(IRngAuctionRelayListener.rngComplete.selector, randomNumber, rngCompletedAt, rewardRecipient, sequenceId, results);
        }
```    
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L31-L38

## [G-20] Forgo internal function to save 1 STATICCALL

The _checkUpdateAuction() internal function performs 14 external calls and returns 14 of the return values from those calls. Certain functions invoke _checkUpdateAuction() but only use the return value from the first external call, thus performing an unnecessary extra external call. We can forgo using the internal function and instead only perform our desired external call. to save (100 gas ) for one external call 

```solidity
file:   src/LiquidationPair.sol

145     function maxAmountOut() external returns (uint256) {
     _checkUpdateAuction();
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L145


## [G-21] With assembly, .call (bool success) transfer can be done gas-optimized

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.

```solidity 
file:  src/RngAuctionRelayerDirect.sol

39   (bool success, bytes memory returnData) = address(_rngAuctionRelayListener).call(data);


```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol#L39

## [G-22] Amounts should be checked for 0 before calling a transfer 

```solidity
file:  src/VaultBooster.sol

173    _token.safeTransferFrom(msg.sender, address(this), _amount);

193    _token.transfer(msg.sender, _amount);

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L173


## [G-23] For loops in public or external functions should be avoided due to high gas costs and possible DOS

```solidity
file:  src/RngRelayAuction.sol

167    for (uint8 i = 0; i < _rewards.length; i++) {
            uint104 _reward = uint104(_rewards[i]);
            if (_reward > 0) {
              prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
              emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
            }
          }
      
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L173