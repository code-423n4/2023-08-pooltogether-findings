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