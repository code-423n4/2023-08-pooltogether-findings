## Summary

### Gas Optimization

no | Issue |Instances||
|-|:-|:-:|:-:|
| [G-01] | Avoid contract existence checks by using low level calls | 6 | - |
| [G-02] | Ternary operation is cheaper than if-else statement | 2 | - |
| [G-03] | Stack variable used as a cheaper cache for a state variable is only used once | 1 | - |
| [G-04] | use mappings instead of arrays | 1 | - |
| [G-05] | Using delete statement can save gas | 3 | - |
| [G-06] | `>=` costs less gas than > | 11 | - |
| [G-07] | Use hardcode address instead address(this) | 7 | - |
| [G-08] | Using calldata instead of memory for read-only arguments | 1 | - |
| [G-09] | Missing zero address check in constructor | 4 | - |
| [G-10] | Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it) | 1 | - |
| [G-11] | Do not calculate constant | 1 | - |
| [G-12] | Use do while loops instead of for loops | 2 | - |
| [G-13] | Functions Guaranteed to revert when called by normal users can be markd payable | 5 | - |
| [G-14] | Declare the State Variabel outside the loop to save gas | 1 | - |
| [G-15] | Structs can be packed into fewer storage slots by editing variables | 2 | - |
| [G-16] | Internal functions not called by the contract should be removed to save deployment gas | 1 | - |
| [G-17] | Use nested if and, avoid multiple check combinations `&&` | 2 | - |
| [G-18] | Use assembly to write address storage values | 5 | - |


## Gas Optimizations  

## [G‑01] Avoid contract existence checks by using low level calls

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


## [G-02] Ternary operation is cheaper than if-else statement


```solidity
File: src/abstract/AddressRemapper.sol

    if (_destinationAddress[_addr] == address(0)) {
      return _addr;
    } else {
      return _destinationAddress[_addr];
    }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L33-L37


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
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L23-L45

## [G‑03] Stack variable used as a cheaper cache for a state variable is only used once

```solidity
File: src/libraries/ContinuousGDA.sol

14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14

## [G-04] use mappings instead of arrays

```solidity
File: src/LiquidationPairFactory.sol

43  LiquidationPair[] public allPairs;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L43

## [G-05] Using delete statement can save gas

```solidity
File:  src/LiquidationPair.sol

279      amount = 0;


337    _amountInForPeriod = 0;

338    _amountOutForPeriod = 0;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L279


## [G-06] `>=` costs less gas than >


```solidity
File: src/RngRelayAuction.sol

113    if (auctionTargetTime_ > auctionDurationSeconds_) {


140    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();

169      if (_reward > 0) {

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

332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol

```solidity
File: src/RngAuction.sol

149    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));

176    if (_auctionElapsedTimeSeconds > auctionDuration) revert AuctionExpired();

179    if (_feeToken != address(0) && _requestFee > 0) {

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol


## [G-07] Use hardcode address instead address(this)

nstead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.
References: https://book.getfoundry.sh/reference/forge-std/compute-create-address

```solidity
File: src/VaultBooster.sol

144      uint256 balance = _token.balanceOf(address(this));

173    _token.safeTransferFrom(msg.sender, address(this), _amount);

190    uint256 availableBalance = _token.balanceOf(address(this));

276    uint256 availableBalance = _tokenOut.balanceOf(address(this));

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144


```solidity
File: src/RngAuction.sol


180      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {

182        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180

## [G‑08] Using calldata instead of memory for read-only arguments

calldata must be used when declaring  an external function's dynamic parameters.
When a function with a memory array is called externally, the abi.decode ()  step has to use a for-loop to copy each index of the calldata to the memory index. Each iteration of this for-loop costs at least 60 gas (i.e. 60 * <mem_array>.length). Using calldata directly, obliviates the need for such a loop in the contract code and runtime execution.

```solidity
File: src/libraries/RewardLib.sol

59    AuctionResult[] memory _auctionResults,

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L59

## [G-09] Missing zero address check in constructor

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
     address owner_, // <= FOUND
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

## [G‑10] Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it)

```solidity
File:  src/LiquidationPair.sol

378    uint256 _timestamp = block.timestamp;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L378

## [G‑11] Do not calculate constant

When you define a constant in Solidity, the compiler can calculate its value at compile-time and replace all references to the constant with its computed value. This can be helpful for readability and reducing the size of the compiled code, but it can also increase gas usage at runtime.

```solidity
File:src/libraries/ContinuousGDA.sol

14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14


## [G-12] Use do while loops instead of for loops

```solidity
File: libraries/RewardLib.sol

    for (uint256 i; i < _auctionResultsLength; i++) {
      _rewards[i] = reward(_auctionResults[i], remainingReserve);
      remainingReserve = remainingReserve - _rewards[i];
    }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65-L68

```solidity
File: src/RngRelayAuction.sol

    for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L172


## [G‑13] Functions Guaranteed to revert when called by normal users can be markd payable

The onlyOwner modifier makes a function revert if not called by the address registered as the owner

```solidity
File: src/LiquidationRouter.sol

68  ) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L68


```solidity
File: src/VaultBooster.sol

142  function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {

188  function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {

217  ) external override onlyPrizeToken(_tokenIn) onlyLiquidationPair(_tokenOut) returns (bool) {

242  function targetOf(address _tokenIn) external view override onlyPrizeToken(_tokenIn) returns (address) {

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142


## [G‑14] Declare the State Variabel outside the loop to save gas

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
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167-L173


## [G‑15] Structs can be packed into fewer storage slots by editing variables

The following structures could be optimized moving the position of certain values in order to save a lot slots:
Enums are represented by integers; the possibility listed first by 0, the next by 1, and so forth.
An enum type just acts like uintN, where N is the smallest legal value large enough to accomodate all the possibilities.

Refference: https://ethdebug.github.io/solidity-data-representation

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
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L23-L29


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
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29-L35


## [G‑16] Internal functions not called by the contract should be removed to save deployment gas

If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

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

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L31-L38

## [G‑17] Use nested if and, avoid multiple check combinations `&&`

Using nested if is cheaper than using && multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

```solidity
File: src/LiquidationPair.sol

332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332


```solidity
File: src/RngAuction.sol

179    if (_feeToken != address(0) && _requestFee > 0) {

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L179

## [G‑18] Use assembly to write address storage values

By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```solidity
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```


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

