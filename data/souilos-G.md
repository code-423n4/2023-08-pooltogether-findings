# VULN 1 

## [GAS] Use != 0 instead of > 0 for unsigned integer comparison
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 169 at 2023-08-pooltogether/src/RngRelayAuction.sol:

      if (_reward > 0) {


Found in line 143 at 2023-08-pooltogether/src/VaultBooster.sol:

    if (_initialAvailable > 0) {


Found in line 269 at 2023-08-pooltogether/src/VaultBooster.sol:

    if (boost.tokensPerSecond > 0) {


Found in line 272 at 2023-08-pooltogether/src/VaultBooster.sol:

    if (boost.multiplierOfTotalSupplyPerSecond.unwrap() > 0) {


Found in line 179 at 2023-08-pooltogether/src/RngAuction.sol:

    if (_feeToken != address(0) && _requestFee > 0) {


Found in line 332 at 2023-08-pooltogether/src/LiquidationPair.sol:

    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

------------------------------------------------------------------------ 

### Mitigation 

Use != 0 instead of > 0 for unsigned integer comparison.










# VULN 2 

## [GAS] ++i/i++ should be unchecked{++i}/unchecked{i++} and ++i costs less gas than i++
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 167 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    for (uint8 i = 0; i < _rewards.length; i++) {


Found in line 65 at 2023-08-pooltogether/src/libraries/RewardLib.sol:

    for (uint256 i; i < _auctionResultsLength; i++) {

------------------------------------------------------------------------ 

### Mitigation 

++i/i++ should be unchecked{++i}/unchecked{i++} when it is not possible for them to overflow, as is the case when used in for and while-loops. Moreover ++i costs less gas than i++, especially when its used in for-loops (--i/i-- too).










# VULN 3 

## [GAS] Don’t initialize variables with default value
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 167 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    for (uint8 i = 0; i < _rewards.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

In such cases, initializing the variables with default values would be unnecessary and can be considered a waste of gas. Additionally, initializing variables with default values can sometimes lead to unnecessary storage operations, which can increase gas costs. For example, if you have a large array of variables, initializing them all with default values can result in a lot of unnecessary storage writes, which can increase the gas costs of your contract.










# VULN 4 

## [GAS] Cache array length outside of loop
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 167 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    for (uint8 i = 0; i < _rewards.length; i++) {

------------------------------------------------------------------------ 

### Mitigation 

If not cached, the solidity compiler will always read the length of the array during each iteration. That is, if it is a storage array, this is an extra sload operation (100 additional extra gas for each iteration except for the first) and if it is a memory array, this is an extra mload operation (3 additional gas for each iteration except for the first).










# VULN 5 

## [GAS] Use assembly to check for address(0)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 108 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();


Found in line 110 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    if (address(_rngAuctionRelayer) == address(0)) revert RngRelayerZeroAddress();


Found in line 104 at 2023-08-pooltogether/src/RemoteOwner.sol:

    if (_newOriginChainOwner == address(0)) revert OriginChainOwnerZeroAddress();


Found in line 431 at 2023-08-pooltogether/src/RngAuction.sol:

    if (address(_newRng) == address(0)) revert RngZeroAddress();


Found in line 47 at 2023-08-pooltogether/src/LiquidationRouter.sol:

    if(address(liquidationPairFactory_) == address(0)) {


Found in line 33 at 2023-08-pooltogether/src/abstract/AddressRemapper.sol:

    if (_destinationAddress[_addr] == address(0)) {


Found in line 83 at 2023-08-pooltogether/src/libraries/RewardLib.sol:

    if (_auctionResult.recipient == address(0)) return 0;

------------------------------------------------------------------------ 

### Mitigation 

Using assembly to check for the zero address can result in significant gas savings compared to using a Solidity expression, especially if the check is performed frequently or in a loop. However, it's important to note that using assembly can make the code less readable and harder to maintain, so it should be used judiciously and with caution.










# VULN 6 

## [GAS] Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 54 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 indexed sequenceId,


Found in line 56 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 index,


Found in line 67 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 indexed sequenceId,


Found in line 68 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 indexed drawId,


Found in line 84 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  uint32 internal _lastSequenceId;


Found in line 135 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 _sequenceId,


Found in line 154 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    uint32 drawId = prizePool.closeDraw(_randomNumber);


Found in line 167 at 2023-08-pooltogether/src/RngRelayAuction.sol:

    for (uint8 i = 0; i < _rewards.length; i++) {


Found in line 197 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  function isSequenceCompleted(uint32 _sequenceId) external view returns (bool) {


Found in line 214 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  function lastSequenceId() external view returns (uint32) {


Found in line 241 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  function _sequenceHasCompleted(uint32 _sequenceId) internal view returns (bool) {


Found in line 273 at 2023-08-pooltogether/src/VaultBooster.sol:

      uint256 totalSupply = twabController.getTotalSupplyTwabBetween(address(vault), uint32(boost.lastAccruedAt), uint32(block.timestamp));


Found in line 31 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 periodLength,


Found in line 32 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 periodOffset,


Found in line 33 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 targetFirstSaleTime,


Found in line 69 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 _periodLength,


Found in line 70 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 _periodOffset,


Found in line 71 at 2023-08-pooltogether/src/LiquidationPairFactory.sol:

    uint32 _targetFirstSaleTime,


Found in line 26 at 2023-08-pooltogether/src/RngAuction.sol:

  uint32 sequenceId;


Found in line 28 at 2023-08-pooltogether/src/RngAuction.sol:

  uint32 rngRequestId;


Found in line 122 at 2023-08-pooltogether/src/RngAuction.sol:

    uint32 indexed sequenceId,


Found in line 124 at 2023-08-pooltogether/src/RngAuction.sol:

    uint32 rngRequestId,


Found in line 188 at 2023-08-pooltogether/src/RngAuction.sol:

    (uint32 rngRequestId,) = rng.requestRandomNumber();


Found in line 189 at 2023-08-pooltogether/src/RngAuction.sol:

    uint32 sequenceId = _openSequenceId();


Found in line 261 at 2023-08-pooltogether/src/RngAuction.sol:

  function openSequenceId() external view returns (uint32) {


Found in line 269 at 2023-08-pooltogether/src/RngAuction.sol:

  function lastSequenceId() external view returns (uint32) {


Found in line 295 at 2023-08-pooltogether/src/RngAuction.sol:

    uint32 requestId = _lastAuction.rngRequestId;


Found in line 365 at 2023-08-pooltogether/src/RngAuction.sol:

  function _openSequenceId() internal view returns (uint32) {


Found in line 374 at 2023-08-pooltogether/src/RngAuction.sol:

    return uint32((currentTime - sequenceOffset) / sequencePeriod);


Found in line 422 at 2023-08-pooltogether/src/RngAuction.sol:

    uint32 requestId = _lastAuction.rngRequestId;


Found in line 48 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint32 public immutable targetFirstSaleTime;


Found in line 69 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint16 _period;


Found in line 97 at 2023-08-pooltogether/src/LiquidationPair.sol:

    uint32 _periodLength,


Found in line 98 at 2023-08-pooltogether/src/LiquidationPair.sol:

    uint32 _periodOffset,


Found in line 99 at 2023-08-pooltogether/src/LiquidationPair.sol:

    uint32 _targetFirstSaleTime,


Found in line 340 at 2023-08-pooltogether/src/LiquidationPair.sol:

    _period = uint16(__period);


Found in line 35 at 2023-08-pooltogether/src/abstract/RngAuctionRelayer.sol:

        uint32 sequenceId = rngAuction.openSequenceId();

------------------------------------------------------------------------ 

### Mitigation 

When using elements that are smaller than 32 bytes, your contract’s gas usage may be higher. This is because the EVM operates on 32 bytes at a time. Therefore, if the element is smaller than that, the EVM must use more operations in order to reduce the size of the element from 32 bytes to the desired size. https://docs.soliditylang.org/en/v0.8.11/internals/layout_in_storage.html. Each operation involving a uint8 costs an extra 22-28 gas (depending on whether the other operand is also a variable of type uint8) as compared to ones involving uint256, due to the compiler having to clear the higher bits of the memory word before operating on the uint8, as well as the associated stack operations of doing so. Use a larger size then downcast where needed.










# VULN 7 

## [GAS] <x> += <y> Costs More Gas Than <x> = <x> + <y> For State Variables
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 274 at 2023-08-pooltogether/src/VaultBooster.sol:

      deltaAmount += convert(boost.multiplierOfTotalSupplyPerSecond.intoUD60x18().mul(convert(deltaTime)).mul(convert(totalSupply)));


Found in line 221 at 2023-08-pooltogether/src/LiquidationPair.sol:

    _amountInForPeriod += uint96(swapAmountIn);


Found in line 222 at 2023-08-pooltogether/src/LiquidationPair.sol:

    _amountOutForPeriod += uint96(_amountOut);


Found in line 223 at 2023-08-pooltogether/src/LiquidationPair.sol:

    _lastAuctionTime += uint48(uint256(convert(convert(int256(_amountOut)).div(_emissionRate))));

------------------------------------------------------------------------ 

### Mitigation 

When you use the += operator on a state variable, the EVM has to perform three operations: load the current value of the state variable, add the new value to it, and then store the result back in the state variable. On the other hand, when you use the = operator and then add the values separately, the EVM only needs to perform two operations: load the current value of the state variable and add the new value to it. Better use <x> = <x> + <y> For State Variables.










# VULN 8 

## [GAS] Use hardcode address instead address(this)
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 144 at 2023-08-pooltogether/src/VaultBooster.sol:

      uint256 balance = _token.balanceOf(address(this));


Found in line 173 at 2023-08-pooltogether/src/VaultBooster.sol:

    _token.safeTransferFrom(msg.sender, address(this), _amount);


Found in line 190 at 2023-08-pooltogether/src/VaultBooster.sol:

    uint256 availableBalance = _token.balanceOf(address(this));


Found in line 276 at 2023-08-pooltogether/src/VaultBooster.sol:

    uint256 availableBalance = _tokenOut.balanceOf(address(this));


Found in line 180 at 2023-08-pooltogether/src/RngAuction.sol:

      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {


Found in line 182 at 2023-08-pooltogether/src/RngAuction.sol:

        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

------------------------------------------------------------------------ 

### Mitigation 

Instead of using address(this), it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry’s script.sol and solmate’s LibRlp.sol contracts can help achieve this.










# VULN 9 

## [GAS] Functions guaranteed to revert when called by normal users can be marked payable
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 142 at 2023-08-pooltogether/src/VaultBooster.sol:

  function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {


Found in line 188 at 2023-08-pooltogether/src/VaultBooster.sol:

  function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {


Found in line 347 at 2023-08-pooltogether/src/RngAuction.sol:

  function setNextRngService(RNGInterface _rngService) external onlyOwner {

------------------------------------------------------------------------ 

### Mitigation 

If a function modifier or require such as onlyOwner/onlyX is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.










# VULN 10 

## [GAS] Using private rather than public for constants, saves gas
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 33 at 2023-08-pooltogether/src/RngAuctionRelayerRemoteOwner.sol:

    uint256 public immutable toChainId;


Found in line 41 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable periodLength;


Found in line 45 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable periodOffset;


Found in line 54 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable minimumAuctionAmount;

------------------------------------------------------------------------ 

### Mitigation 

If needed, the values can be read from the verified contract source code, or if there are multiple values there can be a single getter function that returns a tuple of the values of all currently-public constants. Saves 3406-3606 gas in deployment gas due to the compiler not having to create non-payable getter functions for deployment calldata, not having to store the bytes of the value outside of where it’s used, and not adding another entry to the method ID table.










# VULN 11 

## [GAS] >= costs less gas than >
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 269 at 2023-08-pooltogether/src/LiquidationPair.sol:

    return emissions > liquidatable ? liquidatable : emissions;

------------------------------------------------------------------------ 

### Mitigation 

The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas.










# VULN 12 

## [GAS] With assembly, .call (bool success) transfer can be done gas-optimized
------------------------------------------------------------------------ 

### Proof of concept 

Found in 2023-08-pooltogether/src/RemoteOwner.sol (function execute(address target, uint256 value, bytes calldata data) external returns (bytes memory) {):



  function execute(address target, uint256 value, bytes calldata data) external returns (bytes memory) {

    // console2.log("EXECUTE");

    // console2.logBytes(data);

    _checkSender();

    (bool success, bytes memory returnData) = target.call{ value: value }(data);

    // console2.log("success?", success);

    // console2.logBytes(returnData);

    // console2.log(abi.decode(returnData, (uint256)));

    if (!success) revert CallFailed(returnData);

    assembly {



Found in 2023-08-pooltogether/src/RngAuctionRelayerDirect.sol (function relay():

    function relay(

        IRngAuctionRelayListener _rngAuctionRelayListener,

        address _relayRewardRecipient

    ) external returns (bytes memory) {

        bytes memory data = encodeCalldata(_relayRewardRecipient);

        (bool success, bytes memory returnData) = address(_rngAuctionRelayListener).call(data);

        if (!success) {

            revert DirectRelayFailed(returnData);

        }

        emit DirectRelaySuccess(_relayRewardRecipient, returnData);

        return returnData;


------------------------------------------------------------------------ 

### Mitigation 

return data (bool success,) has to be stored due to EVM architecture, but in a usage like below, ‘out’ and ‘outsize’ values are given (0,0), this storage disappears and gas optimization is provided.










# VULN 13 

## [GAS] Use assembly to write address storage values
------------------------------------------------------------------------ 

### Proof of concept 

Found in lines 102 to 107 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  constructor(
    PrizePool prizePool_,
    address _rngAuctionRelayer,
    uint64 auctionDurationSeconds_,
    uint64 auctionTargetTime_
  ) {


Found in lines 51 to 55 at 2023-08-pooltogether/src/RemoteOwner.sol:

  constructor(
    uint256 originChainId_,
    address executor_,
    address __originChainOwner
  ) ExecutorAware(executor_) {


Found in lines 118 to 122 at 2023-08-pooltogether/src/VaultBooster.sol:

  constructor(
    PrizePool _prizePool,
    address _vault,
    address _owner
  ) Ownable(_owner) {


Found in lines 140 to 147 at 2023-08-pooltogether/src/RngAuction.sol:

  constructor(
    RNGInterface rng_,
    address owner_,
    uint64 sequencePeriod_,
    uint64 sequenceOffset_,
    uint64 auctionDurationSeconds_,
    uint64 auctionTargetTime_
  ) Ownable(owner_) {


Found in lines 93 to 104 at 2023-08-pooltogether/src/LiquidationPair.sol:

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

------------------------------------------------------------------------ 

### Mitigation 

Use assembly to write address storage values. Here are a few reasons: * Reduced opcode usage: When using assembly, you can directly manipulate storage values using lower-level instructions like sstore (storage store) instead of relying on higher-level Solidity storage assignments. These direct operations typically result in fewer opcode executions, reducing gas costs. * Avoiding unnecessary checks: Solidity storage assignments often involve additional checks and operations, such as enforcing security modifiers or triggering events. By using assembly, you can bypass these additional checks and perform the necessary storage operations directly, resulting in gas savings. * Optimized packing: Assembly provides greater flexibility in packing and unpacking data structures. By carefully arranging and manipulating the storage layout in assembly, you can achieve more efficient storage utilization and minimize wasted storage space. * Fine-grained control: Assembly allows for precise control over gas-consuming operations. You can optimize gas usage by selecting specific instructions and minimizing unnecessary operations or data copying.
