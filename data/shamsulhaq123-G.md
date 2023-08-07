## Gas optimization 

No | Issue |Instances|
|-|:-|:-:|:-:|
|G-01|Tightly pack storage variables/optimize the order of variable declaration|5
|G-02|Use calldata instead of memory for function parameters|3
|G-03| Expensive operation inside a for loop|2
|G-04|Nested if is cheaper than single statement|4
|G-05| Change public function visibility to external|1
|G-06|Use assembly for loops|2
|G-07| Use hardcode address instead address(this)|5
|G-08|Default value initialization|2
|G-09|Modifiers are redundant if used only once or not used at|1
|G-10|Use function instead of modifiers (4 instances)|2
|G-11|Not using the named return variable when a function returns, wastes deployment gas|13
|G-12|Use do while loops instead of for loops|2
|G-13|Use assembly to perform efficient back-to-back calls|1
|G-14|Avoid emitting constants.|16

 ### [G-01] Tightly pack storage variables/optimize the order of variable declaration
The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot (if the values combined are <= 32 bytes). If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot.

```solidity

file: LiquidationPair.sol

41   uint256 public immutable periodLength;

  /// @notice Sets the beginning timestamp for the first period.
  /// @dev Ensure that the periodOffset is in the past.
  uint256 public immutable periodOffset;

  /// @notice The time within an auction at which the price of available tokens matches the previous non-zero exchange rate.
  uint32 public immutable targetFirstSaleTime;

  /// @notice Require a minimum number of tokens before an auction is triggered.
  /// @dev This is important, because the gas cost ultimately determines the efficiency of the swap.
  /// If gas cost to auction is 10 cents and the auction is for 11 cents, then the auction price will be driven to zero to make up for the difference.
  /// If gas cost is 10 cents and we're seeking an efficiency of at least 90%, then the minimum auction amount should be $1 worth of tokens.
  uint256 public immutable minimumAuctionAmount;

  /// @notice The last non-zero total tokens in for an auction. This is used to configure the target price for the next auction.
  uint112 _lastNonZeroAmountIn;

  /// @notice The last non-zero total tokens out for an auction.  This is used to configure the target price for the next auction.
  uint112 _lastNonZeroAmountOut;

  /// @notice The total tokens in for the current auction.
  uint96 _amountInForPeriod;

  /// @notice The total tokens out for the current auction.
  uint96 _amountOutForPeriod;

  /// @notice The current auction period. Note that this number can wrap.
  uint16 _period;

  /// @notice The timestamp at which emissions have been consumed to for the current auction
72 uint48 _lastAuctionTime;



```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L41-L72

```solidity


file: RngRelayAuction.sol

131  function rngComplete(
    uint256 _randomNumber,
    uint256 _rngCompletedAt,
    address _rewardRecipient,
135    uint32 _sequenceId,
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L131-L135

```solidity

file: interfaces/IRngAuctionRelayListener.sol

17  function rngComplete(
        uint256 randomNumber,
        uint256 rngCompletedAt,
        address rewardRecipient,
21        uint32 sequenceId,
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/interfaces/IRngAuctionRelayListener.sol#L17-L21

```solidity

file: VaultBooster.sol

32  uint96 tokensPerSecond;
  uint144 available;
34  uint48 lastAccruedAt

55   uint96 tokensPerSecond,
    uint144 initialAvailable,
57    uint48 lastAccruedAt
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L32-L34
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L55-L57

### [G-02] Use calldata instead of memory for function parameters
If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

Note that I’ve also flagged instances where the function is public but can be marked as external since it’s not called by the contract.

```solidity

file: libraries/RewardLib.sol

58 function rewards(
    AuctionResult[] memory _auctionResults,
    uint256 _reserve
61  )

79  function reward(
    AuctionResult memory _auctionResult,
    uint256 _reserve
82  ) 
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L58-L61
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L79-L82

```solidity

file: libraries/RemoteOwnerCallEncoder.so

7     function encodeCalldata(address target, uint256 value, bytes memory data) internal pure returns (bytes memory) 
```
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol#L7

### [G-03] Expensive operation inside a for loop

```solidity

file: LiquidationPair.sol

275   - uint256 amount = source.liquidatableBalanceOf(tokenOut);
 
      + uint256 _amount = source.liquidatableBelanceof(tokenout);
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L275

```solidity

file: LiquidationRouter.sol

69   -   IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
          msg.sender,

     +  IERC20(_liquidationPair.tokenIn())._safeTransferFrom(
         msg.sender,
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69

### [G-04] Nested if is cheaper than single statement

```soliidity

file: LiquidationPair.sol

332  if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) 
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332

```solidity

file: RngAuction.sol

179     if (_feeToken != address(0) && _requestFee > 0) 

224   return _canStartNextSequence() && _auctionElapsedTime() <= auctionDuration;

423     return !_canStartNextSequence() && rng.isRequestComplete(requestId);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L179
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L224
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L423

###   [G-05] Change public function visibility to external
Since the following public functions are not called from within the contract, their visibility can be made external for gas optimization.

```solidity

file: abstract/AddressRemapper.sol

32   function remappingOf(address _addr) public view returns (address)

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L32

### [G-06] Use assembly for loops
In the following instances, assembly is used for more gas efficient loops. The only memory slots that are manually used in the loops are scratch space (0x00-0x20), the free memory pointer (0x40), and the zero slot (0x60). This allows us to avoid using the free memory pointer to allocate new memory, which may result in memory expansion costs.

```solidity

file: RngRelayAuction.sol

167   for (uint8 i = 0; i < _rewards.length; i++)

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167

```solidity

file: libraries/RewardLib.sol

65   for (uint256 i; i < _auctionResultsLength; i++)


```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65

### [G-07] Use hardcode address instead address(this)
Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address.

```solidity

file: RngAuction.sol

180  if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {
        // Transfer tokens from caller to this contract before continuing
182       IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180-L182

```solidity

file: VaultBooster.sol

144     uint256 balance = _token.balanceOf(address(this));

171 
  function deposit(IERC20 _token, uint256 _amount) external {
    _accrue(_token);
173    _token.safeTransferFrom(msg.sender, address(this), _amount);

190  uint256 availableBalance = _token.balanceOf(address(this));

276    uint256 availableBalance = _tokenOut.balanceOf(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L171-L173
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L190
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L276

### [G-08] Default value initialization
Problem
If a variable is not set/initialized, it is assumed to have the default value (0, false, 0x0 etc depending on the data type).
Explicitly initializing it with its default value is an anti-pattern and wastes gas.
Instances include:

```solidity

file: LiquidationPair.sol

279     amount = 0;

337   _amountInForPeriod = 0;
338    _amountOutForPeriod = 0;
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L279
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L337-338

### [G-09] Modifiers are redundant if used only once or not used at all. 
• Deployment. Gas Saved: 33 649
• Minumal Method Call. Gas Saved: -202
• Average Method Call. Gas Saved: 2 700
• Maximum Method Call. Gas Saved: 4 931

```solidity

file: VaultBooster.sol

283   modifier onlyPrizeToken(address _tokenIn) {
    if (IERC20(_tokenIn) != prizePool.prizeToken()) {
      revert UnsupportedTokenIn();
    }
    _;
  }

  /// @notice Ensures that the caller is the liquidation pair for the given token
  /// @param _token The token whose boost's liquidation pair must be the caller
  modifier onlyLiquidationPair(address _token) {
    if (_boosts[IERC20(_token)].liquidationPair != msg.sender) {
      revert OnlyLiquidationPair();
    }
296    _;
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L283-L296

### [G-10] Use function instead of modifiers 
Deployment. Gas Saved: 115 926
Minimum Method Call. Gas Saved: 162
Average Method Call. Gas Saved: -264
Maximum Method Call. Gas Saved: -481
Overall gas change: 734 (2.459%)

```solidity

file: LiquidationRouter.sol

84   modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) 


```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L84

```solidity

file: VaultBooster.sol

283   modifier onlyPrizeToken(address _tokenIn) {
    if (IERC20(_tokenIn) != prizePool.prizeToken()) {
      revert UnsupportedTokenIn();
    }
    _;
  }

  /// @notice Ensures that the caller is the liquidation pair for the given token
  /// @param _token The token whose boost's liquidation pair must be the caller
  modifier onlyLiquidationPair(address _token) {
    if (_boosts[IERC20(_token)].liquidationPair != msg.sender) {
      revert OnlyLiquidationPair();
    }
296    _;


```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L283-L296

### [G-11]  Not using the named return variable when a function returns, wastes deployment gas

```solidity

file: LiquidationPair.sol

225    return swapAmountIn;

318     return purchasePrice;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L225
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L318

```solidity

file: LiquidationPairFactory.sol

107   return _liquidationPair;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L107

```solidity

file: LiquidationRouter.sol

79   return amountIn;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L79

```solidity

file: libraries/ContinuousGDA.sol

43     return result;

68    return amount;

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L43
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L68

```solidity

file: RngAuction.sol

327  return sequenceOffset;

335  return sequencePeriod;
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L327
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L335

```solidity

file: RngAuctionRelayerDirect.sol

44     return returnData;

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol#L44

```solidity

file: RngAuctionRelayerRemoteOwner.sol

68       return messageId;

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol#L68

```solidity

file: libraries/RewardLib.sol

69   return _rewards;

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L69

```solidity

file: abstract/AddressRemapper.sol

34   return _addr;
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L34

```solidity

file: VaultBoosterFactory.sol

33   return booster;

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L33

```solidity

file: RemoteOwner.sol

86    return _originChainOwner;
```
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L86

### [G-12] Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

```solidity

file: RngRelayAuction.sol

167    for (uint8 i = 0; i < _rewards.length; i++) 


```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167

```solidity

file: libraries/RewardLib.sol

65   for (uint256 i; i < _auctionResultsLength; i++) 

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65

## [G-13] Use assembly to perform efficient back-to-back calls
If a similar external call is performed back-to-back, we can use assembly to reuse any function signatures and function parameters that stay the same. In addition, we can also reuse the same memory space for each function call (scratch space + free memory pointer + zero slot), which can potentially allow us to avoid memory expansion costs.

Note: In order to do this optimization safely we will cache the free memory pointer value and restore it once we are done with our function calls. We will also set the zero slot back to 0 if neccessary.

```solidity

file: interfaces/IAuction.sol

28   function auctionDuration() external view returns (uint64);

  /// @notice Returns the last completed auction's sequence id
  function lastSequenceId() external view returns (uint32);

  /// @notice Computes the reward fraction given the auction elapsed time
  /// @param _auctionElapsedTime The elapsed time of the auction
  /// @return The reward fraction
  function computeRewardFraction(uint64 _auctionElapsedTime) external view returns (UD2x18);

  /**
   * @notice Returns the results of the last completed auction.
   * @return auctionResults The completed auction results
   */
  function getLastAuctionResult() external view returns (AuctionResult memory);
43 }

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/interfaces/IAuction.sol#L28-L43

### [G-14] Avoid emitting constants.
A log topic (declared with indexed) has a gas cost of Glogtopic (375 gas).

```solidity

file: LiquidationPairFactory.sol

93  emit PairCreated(
      _liquidationPair,
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
105    );
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L93

```solidity

file: LiquidationRouter.sol

52    emit LiquidationRouterCreated(liquidationPairFactory_);

77    emit SwappedExactAmountOut(_liquidationPair, _receiver, _amountOut, _amountInMax, amountIn);
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L52
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L77

```solidity

file: RngAuction.sol

200 emit RngAuctionCompleted(
      _rewardRecipient,
      sequenceId,
      rng,
      rngRequestId,
      _auctionElapsedTimeSeconds,
      rewardFraction
207    );
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L200-L207

```solidity

file: RngAuctionRelayerDirect.sol

43   emit DirectRelaySuccess(_relayRewardRecipient, returnData);

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol#L43

```solidity

file: RngAuctionRelayerRemoteOwner.sol

67      emit RelayedToDispatcher(rewardRecipient, messageId);

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol#L67

```solidity

file: RngRelayAuction.sol

159   emit RngSequenceCompleted(
      _sequenceId,
      drawId,
      _rewardRecipient,
      _auctionElapsedSeconds,
      rewardFraction
165    );

171     emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L159-L165
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L171

```solidity

file: abstract/AddressRemapper.sol

60  emit AddressRemapped(_caller, _destination);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L60

```solidity

file: VaultBooster.sol

157    emit SetBoost(
      _token,
      _liquidationPair,
      _multiplierOfTotalSupplyPerSecond,
      _tokensPerSecond,
      _initialAvailable,
      uint48(block.timestamp)
164    );

175    emit Deposited(_token, msg.sender, _amount);

195  emit Withdrawn(_token, msg.sender, _amount);

228 
    emit Liquidated(
      IERC20(_tokenOut),
      _account,
      _amountIn,
      _amountOut,
      amountAvailable
    );

254   emit BoostAccrued(_tokenOut, available);    
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L157
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L175
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L195
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L228-L234
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L254

```solidity

file: VaultBoosterFactory.sol

31   emit CreatedVaultBooster(booster, _prizePool, _vault, _owner);


```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L31

```solidity

file: RemoteOwner.sol

108   emit OriginChainOwnerSet(_newOriginChainOwner);
```
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L108