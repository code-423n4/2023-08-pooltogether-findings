|ID|Title|Category|Severity|Instances|
|-|:-|:-:|:-:|:-:|
| [[1](#l-1--unsafe-downcast)] | Unsafe downcast | Control Flow | Low | 15 |
| [[2](#l-1--gas-griefingtheft-is-possible-on-unsafe-external-call)] | Gas griefing/theft is possible on unsafe external call | Volatile Code | Low | 1 |
| [[3](#l-3--unbounded-loop)] | Unbounded loop | Control Flow | Low | 1 |
| [[4](#l-4--use-ownable2step's-transfer-function-rather-than-ownable's-for-transfers-of-ownership)] | Use Ownable2Step's transfer function rather than Ownable's for transfers of ownership | Centralization / Privilege | Low | 4 |
| [[5](#l-5--owner-can-renounce-ownership)] | Owner can renounce Ownership | Centralization / Privilege | Low | 1 |
| [[6](#l-6--contract-will-stop-functioning-in-the-year-2106)] | Contract will stop functioning in the year 2106 | Control Flow | Low | 1 |
| [[7](#l-7--missing-contract-existence-checks-before-low-level-calls)] | Missing Contract-existence Checks Before Low-level Calls | Control Flow | Low | 1 |
| [[8](#l-8--divide-by-zero)] | Divide by Zero | Logical Issue | Low | 2 |
| [[9](#l-9--functions-calling-contracts-with-transfer-hooks-are-missing-reentrancy-guards)] | Functions calling contracts with transfer hooks are missing reentrancy guards | Control Flow | Low | 4 |
| [[10](#nc-1--variables-need-not-be-initialized-to-zero)] | Variables need not be initialized to zero | Coding Style | Informational | 1 |
| [[11](#nc-2--remove-import-forge-stdconsole2sol)] | Remove import `forge-std/console2.sol` | Coding Style | Informational | 1 |
| [[12](#nc-3--consider-disabling-renounceownership)] | Consider disabling `renounceOwnership()` | Control Flow | Informational | 1 |
| [[13](#nc-4--approve-is-fine-as-it-cannot-be-front-run)] | Approve is fine as it cannot be front-run | Volatile Code | Informational | 1 |
| [[14](#nc-5--control-structures-do-not-follow-the-solidity-style-guide)] | Control structures do not follow the Solidity Style Guide | Coding Style | Informational | 54 |


## **L-1** | Unsafe downcast

### **Description** :-

When a type is downcast to a smaller type, the higher order bits are truncated, effectively applying a modulo to the original value. Without any other checks, this wrapping will lead to unexpected behavior and bugs

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
221:uint96(swapAmountIn)
```
#### **Context**:-
[LiquidationPair.sol#L221](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L221)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
222:uint96(_amountOut)
```
#### **Context**:-
[LiquidationPair.sol#L222](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L222)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
223:uint48(uint256(convert(convert(int256(_amountOut)
```
#### **Context**:-
[LiquidationPair.sol#L223](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L223)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
340:uint16(__period)
```
#### **Context**:-
[LiquidationPair.sol#L340](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L340)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
149:uint64(auctionTargetTime_)
```
#### **Context**:-
[RngAuction.sol#L149](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L149)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
149:uint64(auctionDurationSeconds_)
```
#### **Context**:-
[RngAuction.sol#L149](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L149)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
358:uint64(block.timestamp)
```
#### **Context**:-
[RngAuction.sol#L358](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L358)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
374:uint32((currentTime - sequenceOffset)
```
#### **Context**:-
[RngAuction.sol#L374](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L374)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
119:uint64(convert(auctionTargetTime_)
```
#### **Context**:-
[RngRelayAuction.sol#L119](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L119)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
139:uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt)
```
#### **Context**:-
[RngRelayAuction.sol#L139](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L139)
```solidity
File:pt-v5-draw-auction/src/libraries/RewardLib.sol
```
```solidity
45:uint64(rewardFraction.unwrap()
```
#### **Context**:-
[RewardLib.sol#L45](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L45)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
154:uint48(block.timestamp)
```
#### **Context**:-
[VaultBooster.sol#L154](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L154)
```solidity
163:uint48(block.timestamp)
```
#### **Context**:-
[VaultBooster.sol#L163](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L163)
```solidity
224:uint48(block.timestamp)
```
#### **Context**:-
[VaultBooster.sol#L224](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L224)
```solidity
252:uint48(block.timestamp)
```
#### **Context**:-
[VaultBooster.sol#L252](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L252)

## **L-2** | Gas griefing/theft is possible on unsafe external call

### **Description** :-

`return` data `(bool success,)` has to be stored due to EVM architecture, if in a usage like below, ‘out’ and ‘outsize’ values are given (0,0) . Thus, this storage disappears and may come from external contracts a possible Gas griefing/theft problem is avoided

#### <ins>**Proof Of Concept**</ins>

```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
67:    (bool success, bytes memory returnData) = target.call{ value: value }
```
#### **Context**:-
[RemoteOwner.sol#L67](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L67)

## **L-3** | Unbounded loop

### **Description** :-

While looping large collections, it's possible to run out of gas - causing a DOS condition

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
167:for (uint8 i = 0; i < _rewards.length; i++) {
```
#### **Context**:-
[RngRelayAuction.sol#L167](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167)


## **L-4** | Use Ownable2Step's transfer function rather than Ownable's for transfers of ownership

### **Description** :-

`Ownable2Step` and `Ownable2StepUpgradeable` prevent the contract ownership from mistakenly being transferred to an address that cannot handle it (e.g. due to a typo in the address), by requiring that the recipient of the owner permissions actively accept via a contract call of its own.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
6:import { Ownable } from "owner-manager/Ownable.sol";
```
#### **Context**:-
[RngAuction.sol#L6](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L6)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
72:contract RngAuction is IAuction, Ownable {
```
#### **Context**:-
[RngAuction.sol#L72](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L72)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
10:import { Ownable } from "openzeppelin/access/Ownable.sol";
```
#### **Context**:-
[VaultBooster.sol#L10](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L10)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
40:contract VaultBooster is Ownable, ILiquidationSource {
```
#### **Context**:-
[VaultBooster.sol#L40](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L40)

## **L-5** | Owner can renounce Ownership

### **Description** :-

The contract owner is not prevented from renouncing the ownership while the contract is paused, which would cause any user assets stored in the protocol to be locked indefinitely.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
40:contract VaultBooster is Ownable, ILiquidationSource {
```
#### **Context**:-
[VaultBooster.sol#L40](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L40)

## **L-6** | Contract will stop functioning in the year 2106

### **Description** :-

Limiting the timestamp to fit in a uint32 will cause the call below to start reverting in 2106.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
273:      uint256 totalSupply = twabController.getTotalSupplyTwabBetween(address(vault), uint32(boost.lastAccruedAt), uint32(block.timestamp)
```
#### **Context**:-
[VaultBooster.sol#L273](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L273)

## **L-7** | Missing Contract-existence Checks Before Low-level Calls

### **Description** :-

Low-level calls return success if there is no code present at the specified address..

#### <ins>**Proof Of Concept**</ins>

```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
67:    (bool success, bytes memory returnData) = target.call{ value: value }(data);
```
#### **Context**:-
[RemoteOwner.sol#L67](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L67)

## **L-8** | Divide by Zero

### **Description** :-

The divisions below take an input parameter which does not have any zero-value checks, which may lead to the functions reverting when zero is passed.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
382:    return (_timestamp - periodOffset) / periodLength;
```
#### **Context**:-
[LiquidationPair.sol#L382](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L382)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
374:    return uint32((currentTime - sequenceOffset) / sequencePeriod);
```
#### **Context**:-
[RngAuction.sol#L374](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L374)

## **L-9** | Functions calling contracts with transfer hooks are missing reentrancy guards
### **Description** :-
Even if the function follows the best practice of check-effects-interaction, not using a reentrancy guard when there may be transfer hooks will open the users of this protocol up to read-only reentrancies with no way to protect against it, except by block-listing the whole protocol.

#### <ins>Proof Of Concept</ins>


```solidity
File:pt-v5-cgda-liquidator/src/LiquidationRouter.sol
```

```solidity
69:    IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
```
#### **Context**:-
[LiquidationRouter.sol#L69](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69)

```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```

```solidity
173:    _token.safeTransferFrom(msg.sender, address(this), _amount);
```
#### **Context**:-
[VaultBooster.sol#L173](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L173)

```solidity
193:    _token.transfer(msg.sender, _amount);
```
#### **Context**:-
[VaultBooster.sol#L193](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L193)

```solidity
226:    IERC20(_tokenOut).safeTransfer(_account, _amountOut);
```
#### **Context**:-
[VaultBooster.sol#L226](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L226)

## **NC-1** | Variables need not be initialized to zero

### **Description** :-

The default value for variables is zero, so initializing them to zero is superfluous.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
167:uint8 i = 0;
```
#### **Context**:-
[RngRelayAuction.sol#L167](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167)

## **NC-2** | Remove import `forge-std/console2.sol`

### **Description** :-

It’s used to print the values of variables while running tests to help debug and see what’s happening inside your contracts But since it’s a development tool, it serves no purpose on mainnet. 

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
4:import "forge-std/
```
#### **Context**:-
[VaultBooster.sol#L4](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L4)

## **NC-3** | Consider disabling `renounceOwnership()`

### **Description** :-

If the plan for your project does not include eventually giving up all ownership control, consider overwriting OpenZeppelin's Ownable's renounceOwnership() function in order to disable it.

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
72:contract RngAuction is IAuction, Ownable 
```
#### **Context**:-
[RngAuction.sol#L72](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L72)

## **NC-4** | Approve is fine as it cannot be front-run

### **Description** :-

Use Approve .

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
185:.safeIncreaseAllowance(address(rng)
```
#### **Context**:-
[RngAuction.sol#L185](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L185)

## **NC-5** | Control structures do not follow the Solidity Style Guide

### **Description** :-

See the control structures section of the Solidity [Style Guide](https://docs.soliditylang.org/en/latest/style-guide.html#control-structures)

#### <ins>**Proof Of Concept**</ins>

```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
114:    if (_decayConstant.mul(period59).unwrap() > uEXP_MAX_INPUT) {
```
#### **Context**:-
[LiquidationPair.sol#L114](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L114)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
118:    if (targetFirstSaleTime >= periodLength) {
```
#### **Context**:-
[LiquidationPair.sol#L118](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L118)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
122:    if (_initialAmountIn == 0) {
```
#### **Context**:-
[LiquidationPair.sol#L122](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L122)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
126:    if (_initialAmountOut == 0) {
```
#### **Context**:-
[LiquidationPair.sol#L126](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L126)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
218:    if (swapAmountIn > _amountInMax) {
```
#### **Context**:-
[LiquidationPair.sol#L218](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L218)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
277:    if (amount < minimumAuctionAmount) {
```
#### **Context**:-
[LiquidationPair.sol#L277](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L277)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
288:    if (block.timestamp < _lastAuctionTime) {
```
#### **Context**:-
[LiquidationPair.sol#L288](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L288)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
298:    if (_amountOut == 0) {
```
#### **Context**:-
[LiquidationPair.sol#L298](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L298)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
302:    if (_amountOut > maxOut) {
```
#### **Context**:-
[LiquidationPair.sol#L302](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L302)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
314:    if (purchasePrice == 0) {
```
#### **Context**:-
[LiquidationPair.sol#L314](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L314)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
324:    if (currentPeriod != _period) {
```
#### **Context**:-
[LiquidationPair.sol#L324](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L324)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
332:    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {
```
#### **Context**:-
[LiquidationPair.sol#L332](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
343:    if (_emissionRate.unwrap() != 0) {
```
#### **Context**:-
[LiquidationPair.sol#L343](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L343)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationPair.sol
```
```solidity
379:    if (_timestamp < periodOffset) {
```
#### **Context**:-
[LiquidationPair.sol#L379](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L379)
```solidity
File:pt-v5-cgda-liquidator/src/LiquidationRouter.sol
```
```solidity
85:    if (!_liquidationPairFactory.deployedPairs(_liquidationPair)) {
```
#### **Context**:-
[LiquidationRouter.sol#L85](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L85)
```solidity
File:pt-v5-cgda-liquidator/src/libraries/ContinuousGDA.sol
```
```solidity
30:    if (_amount.unwrap() == 0) {
```
#### **Context**:-
[ContinuousGDA.sol#L30](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L30)
```solidity
File:pt-v5-cgda-liquidator/src/libraries/ContinuousGDA.sol
```
```solidity
38:    if (_emissionRate.unwrap() > 1e18) {
```
#### **Context**:-
[ContinuousGDA.sol#L38](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38)
```solidity
File:pt-v5-cgda-liquidator/src/libraries/ContinuousGDA.sol
```
```solidity
61:    if (_price.unwrap() == 0) {
```
#### **Context**:-
[ContinuousGDA.sol#L61](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L61)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
148:    if (sequencePeriod_ == 0) revert SequencePeriodZero();
```
#### **Context**:-
[RngAuction.sol#L148](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L148)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
149:    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));
```
#### **Context**:-
[RngAuction.sol#L149](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L149)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
171:    if (!_canStartNextSequence()) revert CannotStartNextSequence();
```
#### **Context**:-
[RngAuction.sol#L171](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L171)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
176:    if (_auctionElapsedTimeSeconds > auctionDuration) revert AuctionExpired();
```
#### **Context**:-
[RngAuction.sol#L176](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L176)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
179:    if (_feeToken != address(0) && _requestFee > 0) {
```
#### **Context**:-
[RngAuction.sol#L179](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L179)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
180:      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {
```
#### **Context**:-
[RngAuction.sol#L180](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180)
```solidity
File:pt-v5-draw-auction/src/RngAuction.sol
```
```solidity
371:    if (currentTime < sequenceOffset) {
```
#### **Context**:-
[RngAuction.sol#L371](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L371)
```solidity
383:    if (currentTime < sequenceOffset) {
```
#### **Context**:-
[RngAuction.sol#L383](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L383)
```solidity
File:pt-v5-draw-auction/src/RngAuctionRelayerDirect.sol
```
```solidity
40:        if (!success) {
```
#### **Context**:-
[RngAuctionRelayerDirect.sol#L40](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol#L40)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
108:    if (address(prizePool_) == address(0)) revert PrizePoolZeroAddress();
```
#### **Context**:-
[RngRelayAuction.sol#L108](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L108)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
110:    if (address(_rngAuctionRelayer) == address(0)) revert RngRelayerZeroAddress();
```
#### **Context**:-
[RngRelayAuction.sol#L110](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L110)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
111:    if (auctionDurationSeconds_ == 0) revert AuctionDurationZero();
```
#### **Context**:-
[RngRelayAuction.sol#L111](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L111)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
112:    if (auctionTargetTime_ == 0) revert AuctionTargetTimeZero();
```
#### **Context**:-
[RngRelayAuction.sol#L112](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L112)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
113:    if (auctionTargetTime_ > auctionDurationSeconds_) {
```
#### **Context**:-
[RngRelayAuction.sol#L113](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L113)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
138:    if (_sequenceHasCompleted(_sequenceId)) revert SequenceAlreadyCompleted();
```
#### **Context**:-
[RngRelayAuction.sol#L138](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L138)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
140:    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();
```
#### **Context**:-
[RngRelayAuction.sol#L140](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L140)
```solidity
File:pt-v5-draw-auction/src/RngRelayAuction.sol
```
```solidity
169:      if (_reward > 0) {
```
#### **Context**:-
[RngRelayAuction.sol#L169](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L169)
```solidity
File:pt-v5-draw-auction/src/libraries/RewardLib.sol
```
```solidity
35:    if (x.gt(t)) {
```
#### **Context**:-
[RewardLib.sol#L35](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L35)
```solidity
File:pt-v5-draw-auction/src/libraries/RewardLib.sol
```
```solidity
83:    if (_auctionResult.recipient == address(0)) return 0;
```
#### **Context**:-
[RewardLib.sol#L83](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L83)
```solidity
File:pt-v5-draw-auction/src/libraries/RewardLib.sol
```
```solidity
84:    if (_reserve == 0) return 0;
```
#### **Context**:-
[RewardLib.sol#L84](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L84)
```solidity
File:pt-v5-draw-auction/src/abstract/RngAuctionRelayer.sol
```
```solidity
32:        if (!rngAuction.isRngComplete()) revert RngNotCompleted();
```
#### **Context**:-
[RngAuctionRelayer.sol#L32](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L32)
```solidity
File:pt-v5-draw-auction/src/abstract/AddressRemapper.sol
```
```solidity
33:    if (_destinationAddress[_addr] == address(0)) {
```
#### **Context**:-
[AddressRemapper.sol#L33](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L33)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
143:    if (_initialAvailable > 0) {
```
#### **Context**:-
[VaultBooster.sol#L143](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L143)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
145:      if (balance < _initialAvailable) {
```
#### **Context**:-
[VaultBooster.sol#L145](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L145)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
219:    if (_amountOut > amountAvailable) {
```
#### **Context**:-
[VaultBooster.sol#L219](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L219)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
266:    if (deltaTime == 0) {
```
#### **Context**:-
[VaultBooster.sol#L266](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L266)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
269:    if (boost.tokensPerSecond > 0) {
```
#### **Context**:-
[VaultBooster.sol#L269](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L269)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
272:    if (boost.multiplierOfTotalSupplyPerSecond.unwrap() > 0) {
```
#### **Context**:-
[VaultBooster.sol#L272](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L272)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
284:    if (IERC20(_tokenIn) != prizePool.prizeToken()) {
```
#### **Context**:-
[VaultBooster.sol#L284](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L284)
```solidity
File:pt-v5-vault-boost/src/VaultBooster.sol
```
```solidity
293:    if (_boosts[IERC20(_token)].liquidationPair != msg.sender) {
```
#### **Context**:-
[VaultBooster.sol#L293](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L293)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
56:    if (originChainId_ == 0) revert OriginChainIdZero();
```
#### **Context**:-
[RemoteOwner.sol#L56](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L56)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
71:    if (!success) revert CallFailed(returnData);
```
#### **Context**:-
[RemoteOwner.sol#L71](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L71)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
104:    if (_newOriginChainOwner == address(0)) revert OriginChainOwnerZeroAddress();
```
#### **Context**:-
[RemoteOwner.sol#L104](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L104)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
118:    if (!isTrustedExecutor(msg.sender)) revert LocalSenderNotExecutor(msg.sender);
```
#### **Context**:-
[RemoteOwner.sol#L118](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L118)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
119:    if (_fromChainId() != _originChainId) revert OriginChainIdUnsupported(_fromChainId());
```
#### **Context**:-
[RemoteOwner.sol#L119](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L119)
```solidity
File:remote-owner/src/RemoteOwner.sol
```
```solidity
120:    if (_msgSender() != address(_originChainOwner)) revert OriginSenderNotOwner(_msgSender());
```
#### **Context**:-
[RemoteOwner.sol#L120](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L120)
