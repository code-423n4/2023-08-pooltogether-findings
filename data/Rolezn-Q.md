## QA Summary<a name="QA Summary">

### Low Risk Issues
| |Issue|Contexts|
|-|:-|:-:|
| [LOW&#x2011;1](#LOW-QA&#x2011;5) | Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops | 1 |
| [LOW&#x2011;2](#LOW-QA&#x2011;6) | Functions should be marked as `payable` when interacting with `ETH` like for transfering `ETH` to other contracts | 1 |
| [LOW&#x2011;3](#LOW-QA&#x2011;8) | Large transfers may not work with some `ERC20` tokens | 5 |
| [LOW&#x2011;4](#LOW-QA&#x2011;16) | Unsafe downcast | 10 |
| [LOW&#x2011;5](#LOW-QA&#x2011;20) | Using zero as a parameter | 8 |

Total: 25 contexts over 5 issues

### Non-critical Issues
| |Issue|Contexts|
|-|:-|:-:|
| [NC&#x2011;1](#NC-QA&#x2011;3) | Condition will not revert when `block.timestamp` is `==` to the compared variable | 2 |
| [NC&#x2011;2](#NC-QA&#x2011;16) | Functions that alter state should emit events | 2 |
| [NC&#x2011;3](#NC-QA&#x2011;19) | Generate perfect code headers for better solidity code layout and readability | 2 |
| [NC&#x2011;4](#NC-QA&#x2011;23) | Initial value check is missing in Set Functions | 2 |
| [NC&#x2011;5](#NC-QA&#x2011;35) | Redundant Cast | 1 |
| [NC&#x2011;6](#NC-QA&#x2011;36) | Remove `forge-std` import | 1 |
| [NC&#x2011;7](#NC-QA&#x2011;37) | Remove unused `error` definition | 1 |
| [NC&#x2011;8](#NC-QA&#x2011;41) | Use named function calls | 1 |
| [NC&#x2011;9](#NC-QA&#x2011;44) | Zero as a function argument should have a descriptive meaning | 1 |

Total: 13 contexts over 9 issues

## Low Risk Issues

### <a href="#qa-summary">[LOW&#x2011;1]</a><a name="LOW&#x2011;1"> Function can risk gas exhaustion on large receipt calls due to multiple mandatory loops

The function contains loops over arrays that the user cannot influence. 
In any call for the function, users spend gas to iterate over the array. There are loops in the following functions that iterate all array values. Which could risk gas exhaustion on large array calls.

This risk is small, but could be lessened somewhat by allowing separate calls for small loops.
Consider allowing partial calls via array arguments, or detecting expensive call bundles and only partially iterating over them. Since the logic already filters out calls, this would make the later loops smaller.

#### <ins>Proof Of Concept</ins>


```solidity
File: RngRelayAuction.sol

131: function rngComplete(
    ...

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

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L131







### <a href="#qa-summary">[LOW&#x2011;2]</a><a name="LOW&#x2011;2"> Functions should be marked as `payable` when interacting with `ETH` like for transfering `ETH` to other contracts

#### <ins>Proof Of Concept</ins>

```solidity
File: RemoteOwner.sol

63: function execute(address target, uint256 value, bytes calldata data) external returns (bytes memory) {
    ...
    (bool success, bytes memory returnData) = target.call{ value: value }(data);

```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L63



### <a href="#qa-summary">[LOW&#x2011;3]</a><a name="LOW&#x2011;3"> Large transfers may not work with some `ERC20` tokens

Some `IERC20` implementations (e.g `UNI`, `COMP`) may fail if the valued transferred is larger than `uint96`. [Source](https://github.com/d-xo/weird-erc20#revert-on-large-approvals--transfers)

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LiquidationRouter.sol

69: IERC20(_liquidationPair.tokenIn()).safeTransferFrom(

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69

```solidity
File: RngAuction.sol

182: IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L182

```solidity
File: VaultBooster.sol

173: _token.safeTransferFrom(msg.sender, address(this), _amount);

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L173

```solidity
File: VaultBooster.sol

193: _token.transfer(msg.sender, _amount);

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L193

```solidity
File: VaultBooster.sol

226: IERC20(_tokenOut).safeTransfer(_account, _amountOut);

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L226



</details>



### <a href="#qa-summary">[LOW&#x2011;4]</a><a name="LOW&#x2011;4"> Unsafe Downcast
When a type is downcast to a smaller type, the higher order bits are truncated, effectively applying a modulo to the original value. Without any other checks, this wrapping will lead to unexpected behavior and bugs

#### <ins>Proof Of Concept</ins>


```solidity
File: RngAuction.sol

358: return uint64(block.timestamp);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L358

```solidity
File: RngRelayAuction.sol

139: uint64 _auctionElapsedSeconds = uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt);

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L139

```solidity
File: VaultBooster.sol

154: lastAccruedAt: uint48(block.timestamp)
154: uint48(block.timestamp)
163: uint48(block.timestamp)
224: uint48(block.timestamp)
252: uint48(block.timestamp)
224: _boosts[IERC20(_tokenOut)].lastAccruedAt = uint48(block.timestamp);
252: _boosts[_tokenOut].lastAccruedAt = uint48(block.timestamp);
273: uint256 totalSupply = twabController.getTotalSupplyTwabBetween(address(vault), uint32(boost.lastAccruedAt), uint32(block.timestamp));

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L154

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L154

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L163

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L224

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L252

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L224

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L252

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L273



### <a href="#qa-summary">[LOW&#x2011;5]</a><a name="LOW&#x2011;5"> Using zero as a parameter

Taking `0` as a valid argument in Solidity without checks can lead to severe security issues. A historical example is the infamous `0x0` address bug where numerous tokens were lost. This happens because `0` can be interpreted as an uninitialized `address`, leading to transfers to the `0x0` `address`, effectively burning tokens. Moreover, `0` as a denominator in division operations would cause a runtime exception. It's also often indicative of a logical error in the caller's code. It's important to always validate input and handle edge cases like `0` appropriately. Use `require()` statements to enforce conditions and provide clear error messages to facilitate debugging and safer code.

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LiquidationPair.sol

289: return wrap(0);

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L289

```solidity
File: LiquidationPair.sol

357: _initialPrice = wrap(0);

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L357

```solidity
File: ContinuousGDA.sol

31: return SD59x18.wrap(0);
62: return SD59x18.wrap(0);

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L31

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L62



```solidity
File: RngAuctionRelayerRemoteOwner.sol

65: RemoteOwnerCallEncoder.encodeCalldata(address(_remoteRngAuctionRelayListener), 0, listenerCalldata)

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol#L65


</details>




## Non Critical Issues

### <a href="#qa-summary">[NC&#x2011;1]</a><a name="NC&#x2011;1"> Condition will not revert when `block.timestamp` is `==` to the compared variable

The condition does not revert when block.timestamp is equal to the compared `>` or `<` variable. For example, if there is a check for `block.timestamp > _lastAuctionTime` then there should be a check for cases where `block.timestamp` is `==` to `_lastAuctionTime`
 
#### <ins>Proof Of Concept</ins>

```solidity
File: LiquidationPair.sol

288: if (block.timestamp < _lastAuctionTime) {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L288

```solidity
File: RngRelayAuction.sol

139: uint64 _auctionElapsedSeconds = uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt);
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L139




### <a href="#qa-summary">[NC&#x2011;2]</a><a name="NC&#x2011;2"> Functions that alter state should emit events

#### <ins>Proof Of Concept</ins>


```solidity
File: RngAuction.sol

347: function setNextRngService(RNGInterface _rngService) external onlyOwner {
    _setNextRngService(_rngService);
  }
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L347

```solidity
File: RemoteOwner.sol

96: function setOriginChainOwner(address _newOriginChainOwner) external {
    _checkSender();
    _setOriginChainOwner(_newOriginChainOwner);
  }
```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L96




### <a href="#qa-summary">[NC&#x2011;3]</a><a name="NC&#x2011;3"> Generate perfect code headers for better solidity code layout and readability

It is recommended to use pre-made headers for Solidity code layout and readability:
https://github.com/transmissions11/headers

```solidity
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```


#### <ins>Proof Of Concept</ins>


```solidity
File: LiquidationPair.sol

5: LiquidationPair.sol

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L5

```solidity
File: ContinuousGDA.sol

10: ContinuousGDA.sol

```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L10


### <a href="#qa-summary">[NC&#x2011;4]</a><a name="NC&#x2011;4"> Initial value check is missing in Set Functions

A check regarding whether the current value and the new value are the same should be added

#### <ins>Proof Of Concept</ins>

```solidity
File: RngAuction.sol

347: function setNextRngService(RNGInterface _rngService) external onlyOwner {
    _setNextRngService(_rngService);
  }
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L347

```solidity
File: RemoteOwner.sol

96: function setOriginChainOwner(address _newOriginChainOwner) external {
    _checkSender();
    _setOriginChainOwner(_newOriginChainOwner);
  }
```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L96





### <a href="#qa-summary">[NC&#x2011;5]</a><a name="NC&#x2011;5"> Redundant Cast

The type of the variable is the same as the type to which the variable is being cast

#### <ins>Proof Of Concept</ins>

```solidity
File: VaultBooster.sol

192: IERC20(_token)
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L192






### <a href="#qa-summary">[NC&#x2011;6]</a><a name="NC&#x2011;6"> Remove `forge-std` import

`forge-std` is used for logging and debugging purposes and should be removed when not used for development.

#### <ins>Proof Of Concept</ins>


```solidity
File: VaultBooster.sol

4: import "forge-std/console2.sol";

```

https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L4







### <a href="#qa-summary">[NC&#x2011;7]</a><a name="NC&#x2011;7"> Remove unused `error` definition

Note that there may be cases where an error superficially appears to be used, but this is only because there are multiple definitions of the error in different files. In such cases, the error definition should be moved into a separate file. The instances below are the unused definitions.

#### <ins>Proof Of Concept</ins>


```solidity
File: RngAuction.sol

error AuctionDurationGteSequencePeriod

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol



### <a href="#qa-summary">[NC&#x2011;8]</a><a name="NC&#x2011;8"> Use named function calls

Code base has an extensive use of named function calls, but it somehow missed one instance where this would be appropriate.

It should use named function calls on function call, as such:
```solidity
    library.exampleFunction{value: _data.amount.value}({
        _id: _data.id,
        _amount: _data.amount.value,
        _token: _data.token,
        _example: "",
        _metadata: _data.metadata
    });
```

#### <ins>Proof Of Concept</ins>


```solidity
File: RemoteOwner.sol

67: (bool success, bytes memory returnData) = target.call{ value: value }(data);

```

https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L67


### <a href="#qa-summary">[NC&#x2011;9]</a><a name="NC&#x2011;9"> Zero as a function argument should have a descriptive meaning

Consider using descriptive constants or an enum instead of passing zero directly on function calls, as that might be error-prone, to fully describe the caller's intention.

#### <ins>Proof Of Concept</ins>


```solidity
File: RngAuctionRelayerRemoteOwner.sol

65: RemoteOwnerCallEncoder.encodeCalldata(address(_remoteRngAuctionRelayListener), 0, listenerCalldata)

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerRemoteOwner.sol#L65