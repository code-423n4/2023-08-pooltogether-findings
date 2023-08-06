## GAS Summary<a name="GAS Summary">

### Gas Optimizations
| |Issue|Contexts|Estimated Gas Saved|
|-|:-|:-|:-:|
| [GAS&#x2011;1](#GAS&#x2011;1) | Avoid emitting event on every iteration | 1 | 375 |
| [GAS&#x2011;2](#GAS&#x2011;2) | Counting down in `for` statements is more gas efficient | 2 | 514 |
| [GAS&#x2011;3](#GAS&#x2011;3) | Use `assembly` to write address storage values | 11 | 814 |
| [GAS&#x2011;4](#GAS&#x2011;4) | Multiple accesses of a mapping/array should use a local variable cache | 8 | 640 |
| [GAS&#x2011;5](#GAS&#x2011;5) | Remove `forge-std` import | 1 | 100 |
| [GAS&#x2011;6](#GAS&#x2011;6) | The result of a function call should be cached rather than re-calling the function | 5 | 250 |
| [GAS&#x2011;7](#GAS&#x2011;7) | Use do while loops instead of for loops | 2 | 8 |
| [GAS&#x2011;8](#GAS&#x2011;8) | Use nested `if` and avoid multiple check combinations | 2 | 12 |
| [GAS&#x2011;9](#GAS&#x2011;9) | Using XOR (^) and AND (&) bitwise equivalents | 6 | 78 |

Total: 44 contexts over 9 issues

## Gas Optimizations

### <a href="#gas-summary">[GAS&#x2011;1]</a><a name="GAS&#x2011;1"> Avoid emitting event on every iteration

Expensive operations should always try to be avoided within loops. Such operations include: reading/writing to storage, heavy calculations, external calls, and emitting events. In this instance, an event is being emitted every iteration. Events have a base cost of `Glog (375 gas)` per emit and `Glogdata (8 gas) * number of bytes in event`. We can avoid incurring those costs each iteration by emitting the event outside of the loop.

#### <ins>Proof Of Concept</ins>

```solidity
File: RngRelayAuction.sol

167: for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L167



### <a href="#gas-summary">[GAS&#x2011;2]</a><a name="GAS&#x2011;2"> Counting down in `for` statements is more gas efficient

Counting down is more gas efficient than counting up because neither we are making zero variable to non-zero variable and also we will get gas refund in the last transaction when making non-zero to zero variable.

#### <ins>Proof Of Concept</ins>

```solidity
File: RngRelayAuction.sol

167: for (uint8 i = 0; i < _rewards.length; i++) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L167

```solidity
File: RewardLib.sol

65: for (uint256 i; i < _auctionResultsLength; i++) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/libraries/RewardLib.sol#L65




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.AddNum();
            c1.AddNum();
        }
    }
    
    contract Contract0 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=0;i<=9;i++){
                _num = _num +1;
            }
            num = _num;
        }
    }
    
    contract Contract1 {
        uint256 num = 3;
        function AddNum() public {
            uint256 _num = num;
            for(uint i=9;i>=0;i--){
                _num = _num +1;
            }
            num = _num;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 77011                                     | 311             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 7040            | 7040 | 7040   | 7040 | 1       |


| Contract1 contract                        |                 |      |        |      |         |
|-------------------------------------------|-----------------|------|--------|------|---------|
| Deployment Cost                           | Deployment Size |      |        |      |         |
| 73811                                     | 295             |      |        |      |         |
| Function Name                             | min             | avg  | median | max  | # calls |
| AddNum                                    | 3819            | 3819 | 3819   | 3819 | 1       |



### <a href="#gas-summary">[GAS&#x2011;3]</a><a name="GAS&#x2011;3"> Use `assembly` to write address storage values

#### <ins>Proof Of Concept</ins>

<details>

```solidity
File: LiquidationPair.sol

130: _lastNonZeroAmountIn = _initialAmountIn;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L130

```solidity
File: LiquidationPair.sol

131: _lastNonZeroAmountOut = _initialAmountOut;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L131

```solidity
File: LiquidationPair.sol

334: _lastNonZeroAmountIn = _amountInForPeriod;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L334

```solidity
File: LiquidationPair.sol

335: _lastNonZeroAmountOut = _amountOutForPeriod;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L335

```solidity
File: LiquidationPair.sol

342: _emissionRate = emissionRate_;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L342

```solidity
File: LiquidationRouter.sol

50: _liquidationPairFactory = liquidationPairFactory_;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationRouter.sol#L50

```solidity
File: RngAuction.sol

435: _nextRng = _newRng;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngAuction.sol#L435

```solidity
File: RngRelayAuction.sol

117: _auctionDurationSeconds = auctionDurationSeconds_;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L117

```solidity
File: RngRelayAuction.sol

145: _lastSequenceId = _sequenceId;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L145

```solidity
File: RemoteOwner.sol

57: _originChainId = originChainId_;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/remote-owner/src/RemoteOwner.sol#L57

```solidity
File: RemoteOwner.sol

106: _originChainOwner = _newOriginChainOwner;
```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/remote-owner/src/RemoteOwner.sol#L106



</details>


### <a href="#gas-summary">[GAS&#x2011;4]</a><a name="GAS&#x2011;4"> Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

#### <ins>Proof Of Concept</ins>


```solidity
File: RngRelayAuction.sol

170: prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
171: emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L170-L171

```solidity
File: AddressRemapper.sol

33: if (_destinationAddress[_addr] == address(0)) {
36: return _destinationAddress[_addr];

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/abstract/AddressRemapper.sol#L33

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/abstract/AddressRemapper.sol#L36



```solidity
File: VaultBooster.sol

223: _boosts[IERC20(_tokenOut)].available = amountAvailable.toUint144();
224: _boosts[IERC20(_tokenOut)].lastAccruedAt = uint48(block.timestamp);

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-vault-boost/src/VaultBooster.sol#L223-L224

```solidity
File: VaultBooster.sol

251: _boosts[_tokenOut].available = available.toUint144();
252: _boosts[_tokenOut].lastAccruedAt = uint48(block.timestamp);

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-vault-boost/src/VaultBooster.sol#L251-L252


### <a href="#gas-summary">[GAS&#x2011;5]</a><a name="GAS&#x2011;5"> Remove import `forge-std`

It's used to print the values of variables while running tests to help debug and see what's happening inside your contracts but since it's a development tool, it serves no purpose on mainnet.
Also, the remember to remove the usage of calls that use `forge-std` when removing of the import of `forge-std`.

#### <ins>Proof Of Concept</ins>

```solidity
File: VaultBooster.sol

4: import "forge-std/console2.sol";

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-vault-boost/src/VaultBooster.sol#L4






### <a href="#gas-summary">[GAS&#x2011;6]</a><a name="GAS&#x2011;6"> The result of a function call should be cached rather than re-calling the function

External calls are expensive. Results of external function calls should be cached rather than call them multiple times. Consider caching the following:

#### <ins>Proof Of Concept</ins>


```solidity
File: RngAuction.sol

370: uint64 currentTime = _currentTime();
382: uint64 currentTime = _currentTime();
386: return (_currentTime() - sequenceOffset) % sequencePeriod;

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngAuction.sol#L370

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngAuction.sol#L382

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngAuction.sol#L386



```solidity
File: RemoteOwner.sol

119: if (_fromChainId() != _originChainId) revert OriginChainIdUnsupported(_fromChainId());
120: if (_msgSender() != address(_originChainOwner)) revert OriginSenderNotOwner(_msgSender());

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/remote-owner/src/RemoteOwner.sol#L119

https://github.com/code-423n4/2023-08-pooltogether/tree/main/remote-owner/src/RemoteOwner.sol#L120



### <a href="#gas-summary">[GAS&#x2011;7]</a><a name="GAS&#x2011;7"> Use do while loops instead of for loops

A do while loop will cost less gas since the condition is not being checked for the first iteration.

#### <ins>Proof Of Concept</ins>

```solidity
File: RngRelayAuction.sol

131: function rngComplete

for (uint8 i = 0; i < _rewards.length; i++) {
      uint104 _reward = uint104(_rewards[i]);
      if (_reward > 0) {
        prizePool.withdrawReserve(auctionResults[i].recipient, _reward);
        emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);
      }

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngRelayAuction.sol#L132

```solidity
File: RewardLib.sol

58: function rewards

for (uint256 i; i < _auctionResultsLength; i++) {
      _rewards[i] = reward(_auctionResults[i], remainingReserve);
      remainingReserve = remainingReserve - _rewards[i];
    }

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/libraries/RewardLib.sol#L59


### <a href="#gas-summary">[GAS&#x2011;8]</a><a name="GAS&#x2011;8"> Use nested `if` and avoid multiple check combinations

Using nested `if`, is cheaper than using `&&` multiple check combinations. There are more advantages, such as easier to read code and better coverage reports.

#### <ins>Proof Of Concept</ins>

```solidity
File: LiquidationPair.sol

332: if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L332

```solidity
File: RngAuction.sol

179: if (_feeToken != address(0) && _requestFee > 0) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/RngAuction.sol#L179




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
    
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
    
        function testGas() public {
            c0.checkAge(19);
            c1.checkAgeOptimized(19);
        }
    }
    
    contract Contract0 {
    
        function checkAge(uint8 _age) public returns(string memory){
            if(_age>18 && _age<22){
                return "Eligible";
            }
        }
    
    }
    
    contract Contract1 {
    
        function checkAgeOptimized(uint8 _age) public returns(string memory){
            if(_age>18){
                if(_age<22){
                    return "Eligible";
                }
            }
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76923                                     | 416             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAge                                  | 651             | 651 | 651    | 651 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 76323                                     | 413             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| checkAgeOptimized                         | 645             | 645 | 645    | 645 | 1       |



### <a href="#gas-summary">[GAS&#x2011;9]</a><a name="GAS&#x2011;9"> Using XOR (^) and AND (&) bitwise equivalents

Given 4 variables a, b, c and d represented as such:

    0 0 0 0 0 1 1 0 <- a
    0 1 1 0 0 1 1 0 <- b
    0 0 0 0 0 0 0 0 <- c
    1 1 1 1 1 1 1 1 <- d

To have a == b means that every 0 and 1 match on both variables. Meaning that a XOR (operator ^) would evaluate to 0 ((a ^ b) == 0), as it excludes by definition any equalities.Now, if a != b, this means that there’s at least somewhere a 1 and a 0 not matching between a and b, making (a ^ b) != 0.Both formulas are logically equivalent and using the XOR bitwise operator costs actually the same amount of gas.However, it is much cheaper to use the bitwise OR operator (|) than comparing the truthy or falsy values.These are logically equivalent too, as the OR bitwise operator (|) would result in a 1 somewhere if any value is not 0 between the XOR (^) statements, meaning if any XOR (^) statement verifies that its arguments are different.

#### <ins>Proof Of Concept</ins>

```solidity
File: LiquidationPair.sol

298: if (_amountOut == 0) {
314: if (purchasePrice == 0) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L298

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-cgda-liquidator/src/LiquidationPair.sol#L314



```solidity
File: RewardLib.sol

83: if (_auctionResult.recipient == address(0)) return 0;
84: if (_reserve == 0) return 0;

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-draw-auction/src/libraries/RewardLib.sol#L83-L84

```solidity
File: VaultBooster.sol

266: if (deltaTime == 0) {

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/pt-v5-vault-boost/src/VaultBooster.sol#L266

```solidity
File: RemoteOwner.sol

104: if (_newOriginChainOwner == address(0)) revert OriginChainOwnerZeroAddress();

```

https://github.com/code-423n4/2023-08-pooltogether/tree/main/remote-owner/src/RemoteOwner.sol#L104




#### Test Code

    contract GasTest is DSTest {
        Contract0 c0;
        Contract1 c1;
        function setUp() public {
            c0 = new Contract0();
            c1 = new Contract1();
        }
        function testGas() public {
            c0.not_optimized(1,2);
            c1.optimized(1,2);
        }
    }
    
    contract Contract0 {
        function not_optimized(uint8 a,uint8 b) public returns(bool){
            return ((a==1) || (b==1));
        }
    }
    
    contract Contract1 {
        function optimized(uint8 a,uint8 b) public returns(bool){
            return ((a ^ 1) & (b ^ 1)) == 0;
        }
    }

#### Gas Test Report

| Contract0 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 46099                                     | 261             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| not_optimized                             | 456             | 456 | 456    | 456 | 1       |


| Contract1 contract                        |                 |     |        |     |         |
|-------------------------------------------|-----------------|-----|--------|-----|---------|
| Deployment Cost                           | Deployment Size |     |        |     |         |
| 42493                                     | 243             |     |        |     |         |
| Function Name                             | min             | avg | median | max | # calls |
| optimized                                 | 430             | 430 | 430    | 430 | 1       |