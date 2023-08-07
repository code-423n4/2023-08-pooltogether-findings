# GAS OPTIMIZATIONS

All the gas optimizations are determined based opcodes and sample tests.

| Count | Issues | Instances | Gas Saved |
| --- | --- | --- | --- |
| [G-1] | Structs can be modified to fit in fewer storage slots | 1 | 40000 |
| [G-2] | Use struct dot notation to avoid memory allocation | 2 | 356 |
| [G-3] | Can Make The Variable Outside The Loop To Save Gas | 1 | Per interation (100 gas) |
| [G-4] | Use UD60x18 instead  of UD59X18s gas | 6 | — |
| [G-5] | Use emit outside the loop for better gas optimization | 1 | — |
| [G-6] | Use of memory instead of storage for struct/array state variables | 1 | 2100 |
| [G-7] |  Use of memory instead of calldata for immutable arguments | 2 | — |
| [G-8] | Multiple accesses of a mapping/array should use a local variable cacheo not calculate constants | 2 | 168 |

### Total gas saved: 42724

### [G-01] Structs can be modified to fit in fewer storage slots

Some member types can be safely modified, and as result, these `struct` will fit in fewer storage slots. Each slot saved can avoid an extra Gsset (**20000 gas**) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

`Gas save 40000` save 2 slots

 https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29C1-L35C2

```solidity
File: /src/VaultBooster.sol
29: struct Boost {
30: address liquidationPair; //address(20)
31: UD2x18 multiplierOfTotalSupplyPerSecond; //UD2x18=uint64 (8)
32: uint96 tokensPerSecond; //uint96 (12)
33: uint144 available; //uint144 (18)
34: uint48 lastAccruedAt; //uint48 (6)
35: }
```

```diff
File: /src/VaultBooster.sol
29: struct Boost {
30: address liquidationPair; 
-31: UD2x18 multiplierOfTotalSupplyPerSecond; 
+31: uint96 tokensPerSecond; 
+32: UD2x18 multiplierOfTotalSupplyPerSecond; 
33: uint144 available; 
34: uint48 lastAccruedAt; 
35: }
```

### [G-02] Use struct `dot` notation to avoid memory allocation

In the instances below, structs values are being assigned via the following method: `Type({field1: value1, field2: value2});`. This method results in a new struct being created in memory with our specified fields. That memory struct is then written to storage. We can leverage the struct `dot` notation to assign values **directly to storage**, thus avoiding allocating memory space for a temporary struct.

`Gas save : 356`

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L149C1-L155C8

```solidity
File: /src/VaultBooster.sol
149: _boosts[_token] = Boost({
150:      liquidationPair: _liquidationPair,
151:      multiplierOfTotalSupplyPerSecond: _multiplierOfTotalSupplyPerSecond,
152:      tokensPerSecond: _tokensPerSecond,
153:      available: _initialAvailable,
154:      lastAccruedAt: uint48(block.timestamp)
155:    });
```

```diff
File: /src/VaultBooster.sol
+      Boost storage boost = _boosts[_token]
-149: _boosts[_token] = Boost({
149:  _boosts[_token] = Boost({
-150:      liquidationPair: _liquidationPair,
-151:      multiplierOfTotalSupplyPerSecond: _multiplierOfTotalSupplyPerSecond,
-152:      tokensPerSecond: _tokensPerSecond,
-153:      available: _initialAvailable,
-154:      lastAccruedAt: uint48(block.timestamp)
+150:      boost.liquidationPair: _liquidationPair,
+151:      boost.multiprlierOfTotalSupplyPerSecond: _multiplierOfTotalSupplyPerSecond,
+152:      boost.tokensPerSecond: _tokensPerSecond,
+153:      boost.available: _initialAvailable,
+154:      boost.lastAccruedAt: uint48(block.timestamp)
155:    });
```

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L192C1-L199C1

```solidity
File: /src/RngAuction.sol

192: _lastAuction = RngAuctionResult({
193:      recipient: _rewardRecipient,
194:      rewardFraction: rewardFraction,
195:      sequenceId: sequenceId,
196:      rng: rng,
197:      rngRequestId: rngRequestId
198:    });
```

### [G-03] Can Make The Variable Outside The Loop To Save Gas

When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L168C7-L168C46

```solidity
File: /src/RngRelayAuction.sol
168: uint104 _reward = uint104(_rewards[i]);
```

### [G-04] Use `UD60x18` instead  of `UD59X18`

If you don't need negative numbers, there's no point in using the signed flavor `SD59x18`. The unsigned flavor `UD60x18` is more gas efficient.

[see Documentation](https://github.com/PaulRBerg/prb-math#usage)

```solidity
File: /src/LiquidationPair.sol
74: /// @notice The rate of token emissions for the current auction
75:  SD59x18 _emissionRate;
76:
77:  /// @notice The initial price for the current auction
78:  SD59x18 _initialPrice;
```

```solidity
File: src/libraries/ContinuousGDA.sol
14: SD59x18 internal constant ONE = SD59x18.wrap(1e18);
```

### [G-05] Use emit outside the loop for better gas optimization**.**

Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop. Gas savings should be multiplied by the average loop length.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L171C8-L172C8

```solidity
File: /src/RngRelayAuction.sol
171: emit AuctionRewardDistributed(_sequenceId, auctionResults[i].recipient, i, _reward);

```

### [G-06] Use of `memory` instead of `storage` for struct/array state variables

When fetching data from `storage`, using the `memory` keyword to assign it to a variable reads all fields of the struct/array and incurs a Gcoldsload (**2100 gas**) for each field.

This can be avoided by declaring the variable with the `storage` keyword and caching the necessary fields in stack variables.

The `memory` keyword should only be used if the entire struct/array is being returned or passed to a function that requires `memory`, or if it is being read from another `memory` array/struct.

`Gas save 2100`

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L147C4-L147C68

```solidity
File: /src/RngRelayAuction.sol
147: AuctionResult[] memory auctionResults = new AuctionResult[](2);
```

### [G-07] Use of `memory` instead of `calldata` for immutable arguments

Consider refactoring the function arguments from `memory` to `calldata` when they are immutable, as `calldata` is cheaper.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L79C2-L82C4

```solidity
File: /libraries/RewardLib.sol
58: function rewards(
59:    AuctionResult[] memory _auctionResults,
60:    uint256 _reserve
61:  )
```

```solidity
File: /libraries/RewardLib.sol
79: function reward(
80:    AuctionResult memory _auctionResult,
81:    uint256 _reserve
52:  )
```

### [G-08] Multiple accesses of a mapping/array should use a local variable cache

Caching a mapping's value in a local storage or calldata variable when the value is accessed multiple times saves ~42 gas per access due to not having to perform the same offset calculation every time. **Help the Optimizer by saving a storage variable's reference instead of repeatedly fetching it**

To help the optimizer,declare a storage type variable and use it instead of repeatedly fetching the reference in a map or an array. As an example, instead of repeatedly calling `someMap[someIndex]`, save its reference like this: `SomeStruct storage someStruct = someMap[someIndex]` and use it.

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L249C3-L257C4

```solidity
File: /src/VaultBooster.sol
//@audit  _boosts[_tokenOut]
249: function _accrue(IERC20 _tokenOut) internal returns (uint256) {
250:    uint256 available = _computeAvailable(_tokenOut);
251:    _boosts[_tokenOut].available = available.toUint144();
252:    _boosts[_tokenOut].lastAccruedAt = uint48(block.timestamp);
253:
254:    emit BoostAccrued(_tokenOut, available);
255:
256:    return available;
257:  }
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L211C3-L237C4

```solidity
File: /src/VaultBooster.sol
//@audit _boosts[IERC20(_tokenOut)]
211: function liquidate(
212:    address _account,
213:    address _tokenIn,
214:    uint256 _amountIn,
215:    address _tokenOut,
216:    uint256 _amountOut
217:  ) external override onlyPrizeToken(_tokenIn) onlyLiquidationPair(_tokenOut) returns (bool) {
218:    uint256 amountAvailable = _computeAvailable(IERC20(_tokenOut));
219:    if (_amountOut > amountAvailable) {
220:      revert InsufficientAvailableBalance(_amountOut, amountAvailable);
221:    }
222:    amountAvailable = (amountAvailable - _amountOut);
223:    _boosts[IERC20(_tokenOut)].available = amountAvailable.toUint144();
224:    _boosts[IERC20(_tokenOut)].lastAccruedAt = uint48(block.timestamp);
225:    prizePool.contributePrizeTokens(vault, _amountIn);
226:    IERC20(_tokenOut).safeTransfer(_account, _amountOut);
227:
228:    emit Liquidated(
229:      IERC20(_tokenOut),
230:     _account,
231:      _amountIn,
232:      _amountOut,
233:      amountAvailable
234:    );
235:
236:    return true;
237:  }
```