  ## QA
---

### Layout Order [1]

- The best-practices for layout within a contract is the following order: state variables, events, modifiers, constructor and functions.

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol

```solidity
// place events after all state variables
36:  event OriginChainOwnerSet(address owner);
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol

```solidity
// place events after all state variables
51:  event SetBoost(
64:  event Deposited(
74:  event Withdrawn(
86:  event Liquidated(
97:  event BoostAccrued(

// place this modifier right before the constructor.
283:  modifier onlyPrizeToken(address _tokenIn) {
292:  modifier onlyLiquidationPair(address _token) {
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol

```solidity
// place events after all state variables
53:  event AuctionRewardDistributed(
66:  event RngSequenceCompleted(
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol

```solidity
// place events after all state variables
23:    event RelayedToDispatcher(address indexed rewardRecipient, bytes32 indexed messageId);
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol

```solidity
// place events after all state variables.
22:  event LiquidationRouterCreated(LiquidationPairFactory indexed liquidationPairFactory);
30:  event SwappedExactAmountOut(

// place this modifier right before the constructor.
84:  modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol

```solidity
// place events after all state variables.
26:  event PairCreated(
```

---

### Function Visibility [2]

- Order of Functions: Ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. Functions should be grouped according to their visibility and ordered: constructor, receive function (if exists), fallback function (if exists), external, public, internal, private. Within a grouping, place the view and pure functions last.

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol

```solidity
// place this non-view before all the view external ones.
96:  function setOriginChainOwner(address _newOriginChainOwner) external {
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol

```solidity
// place this external before the public one.
47:  function remapTo(address _destination) external {
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol

```solidity
// place non-view external before all the external views
288:  function getRngResults()
347:  function setNextRngService(RNGInterface _rngService) external onlyOwner {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol

```solidity
// place view function after non-view functions from the same visibility
287:  function _getElapsedTime() internal view returns (SD59x18) {
```

---

### natSpec missing [3]

Some functions are missing @params or @returns. Specification Format.” These are written with a triple slash (///) or a double asterisk block(/** ... */) directly above function declarations or statements to generate documentation in JSON format for developers and end-users. It is recommended that Solidity contracts are fully annotated using NatSpec for all public interfaces (everything in the ABI). These comments contain different types of tags:
- @title: A title that should describe the contract/interface @author: The name of the author (for contract, interface) 
- @notice: Explain to an end user what this does (for contract, interface, function, public state variable, event) 
- @dev: Explain to a developer any extra details (for contract, interface, function, state variable, event) 
- @param: Documents a parameter (just like in doxygen) and must be followed by parameter name (for function, event)
- @return: Documents the return variables of a contract’s function (function, public state variable)
- @inheritdoc: Copies all missing tags from the base function and must be followed by the contract name (for function, public state variable)
- @custom…: Custom tag, semantics is application-defined (for everywhere)

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol

```solidity
6:  library RemoteOwnerCallEncoder {
7:    function encodeCalldata(address target, uint256 value, bytes memory data) internal pure returns (bytes memory) {
```

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol

```solidity
// @params missing
51:  constructor(

// @return missing
96:  function setOriginChainOwner(address _newOriginChainOwner) external {

63:  function execute(address target, uint256 value, bytes calldata data) external returns (bytes memory) {
85:  function originChainOwner() external view returns (address) {
103:  function _setOriginChainOwner(address _newOriginChainOwner) internal {

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/interfaces/IAuction.sol

```solidity
// @return missing
31:  function lastSequenceId() external view returns (uint32);
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol

```solidity
// @return missing
131:  function rngComplete(
202:  function auctionDuration() external view returns (uint64) {
214:  function lastSequenceId() external view returns (uint32) {
219:  function getLastAuctionResult()
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol

```solidity
// @return missing
215:  function canStartNextSequence() external view returns (bool) {
223:  function isAuctionOpen() external view returns (bool) {
234:  function currentFractionalReward() external view returns (UD2x18) {
239:  function getLastAuction() external view returns (RngAuctionResult memory) {
244:  function getLastAuctionResult()

// @param and @return missing
300:  function computeRewardFraction(uint64 __auctionElapsedTime) external view returns (UD2x18) {

398:  function _computeRewardFraction(uint64 __auctionElapsedTime) internal view returns (UD2x18) {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol

```solidity
// @return missing
77:  function computeK(
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol

```solidity
// @param missing
22:  event LiquidationRouterCreated(LiquidationPairFactory indexed liquidationPairFactory);
```

---

### State variable and function names [4]

- Variables should be named according to their specifications
- private and internal `variables` should preppend with `underline`
- private and internal `functions` should preppend with `underline`
- constant state variables should be UPPER_CASE

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol

```solidity
//  private and internal `functions` should preppend with `underline` 
7:    function encodeCalldata(address target, uint256 value, bytes memory data) internal pure returns (bytes memory) {
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol

```solidity
// private and internal `functions` should preppend with `underline`
31:    function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol

```solidity
// private and internal `functions` should preppend with `underline`
25:  function fractionalReward(
58:  function rewards(
79:  function reward(
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol

```solidity
// private and internal `functions` should preppend with `underline`
23:  function purchasePrice(
54:  function purchaseAmount(
77:  function computeK(
```

---

### Version [5]

- Pragma versions should be standardized and avoid floating pragma `( ^ )`.

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.0;
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.0;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/interfaces/IAuction.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/interfaces/IRngAuctionRelayListener.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol

```solidity
// remove the ^ to lock this to a single version, slightly safer.
2:  pragma solidity ^0.8.19;
```

---

### Observations [6]

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/libraries/RemoteOwnerCallEncoder.sol

```solidity
// remove these test comments.
14:        // assembly {
15:        //    return (add(returnData, 0x20), mload(returnData))
16:        // }
```

https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol

```solidity
// remove these test comments.
64:    // console2.log("EXECUTE");
65:    // console2.logBytes(data);
68:    // console2.log("success?", success);
69:    // console2.logBytes(returnData);
70:    // console2.log(abi.decode(returnData, (uint256)));
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol

```solidity
// explicitly declare state variables visibility, internal in this case
57:  uint112 _lastNonZeroAmountIn;
60:  uint112 _lastNonZeroAmountOut;
63:  uint96 _amountInForPeriod;
66:  uint96 _amountOutForPeriod;
69:  uint16 _period;
72:  uint48 _lastAuctionTime;
75:  SD59x18 _emissionRate;
78:  SD59x18 _initialPrice;
```
