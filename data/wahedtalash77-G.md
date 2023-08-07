|     |
| --- |
| \[G-01\] State variables can be packed to use fewer storage slots |
| \[G-02\]  Usage of uints/ints smaller than 32 bytes (256 bits) incurs overhead |
| \[G-03\] If statements that use && can be refactored into nested if statements |
| \[G-03\] The result of a function call should be cached |
| \[G-5\] Caching global variables is expensive than using the variable itself |
| \[G‑06\] Using private/internal rather than public for constants, saves gas |
| \[G-7\] `variable == false` instead of `!variable`. |
| \[G-08\] Make for loop unchecked |
| \[G-9\] you can save storage by reordering Holding struct fields in the following way. |
| \[G‑10\] Use a more recent version of Solidity |
| \|\[G-11\]\| Use assembly for math (add, sub, mul, div) |
| \[G-12\]\| internal functions not called by the contract should be removed to save deployment gas |

* * *

## \[G-01\] State variables can be packed to use fewer storage slots

The EVM works with 32 byte words. Variables less than 32 bytes can be declared next to eachother in storage and this will pack the values together into a single 32 byte storage slot. If the variables packed together are retrieved together in functions we will effectively save ~2000 gas with every subsequent SLOAD for that storage slot. This is due to us incurring a Gwarmaccess (100 gas) versus a Gcoldsload (2100 gas).

```Solidity
/// @notice The token that is used to pay for auctions
 address public immutable tokenIn;

  /// @notice The token that is being auctioned.
  address public immutable tokenOut;

  /// @notice The rate at which the price decays
  SD59x18 public immutable decayConstant;

  /// @notice The duration of each auction.
  uint256 public immutable periodLength;

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
  uint48 _lastAuctionTime;
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L32C2-L72C27

## \[G-02\] Usage of `uints/ints` smaller than 32 bytes (256 bits) incurs overhead

The EVM operates with 32 byte words. Therefore, if you declare state variables less than 32 bytes the EVM will need to perform extra operations to cast your value to the specified size.

```Solidity
69: uint16 _period;
```

## \[G-03\] If statements that use && can be refactored into nested if statements

```Solidity
332   if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332

## \[G-04\]The result of a function call should be cached

`return periodOffset + __period * periodLength;`
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L365

`return (_timestamp - periodOffset) / periodLength;`
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L382

## \[G-5\] Caching global variables is expensive than using the variable itself

Don't cache `block.timestamp` in the following

```Solidity
function _computePeriod() internal view returns (uint256) {
    uint256 _timestamp = block.timestamp;
    if (_timestamp < periodOffset) {
      return 0;
    }
    return (_timestamp - periodOffset) / periodLength;
  }
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L377C1-L383C4

## \[G‑06\] Using private/internal rather than public for constants, saves gas

```Solidity
14: SD59x18 internal constant ONE = SD59x18.wrap(1e18);
```

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14

## \[G-7\] `variable == false` instead of `!variable`.

a bit cheapier when you replace:

`if (!success) {`
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuctionRelayerDirect.sol#L40
`if (!success) revert CallFailed(returnData);`
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L71

## \[G-08\] Make for loop unchecked

The risk of for loops getting overflowed is extremely low. Because it always increments by 1 and is limited to the arrays length. Even if the arrays are extremely long, it will take a massive amount of time and gas to let the for loop overflow.

https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167

## \[G-9\] you can save storage by reordering Holding struct fields in the following way.

```Solidity
struct Boost {
  address liquidationPair;
  uint96 tokensPerSecond;
  uint144 available;
  uint48 lastAccruedAt;
  UD2x18 multiplierOfTotalSupplyPerSecond;
}
```

## \[G‑10\] Use a more recent version of Solidity

Use a Solidity version of at least 0.8.2 to get simple compiler automatic inlining.

Use a Solidity version of at least 0.8.3 to get better struct packing and cheaper multiple storage reads.

Use a Solidity version of at least 0.8.4 to get custom errors, which are cheaper at deployment than revert()/require() strings.

Use a Solidity version of at least 0.8.10 to have external calls skip contract existence checks if the external call has a return value.

`pragma solidity ^0.8.0;`

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L2