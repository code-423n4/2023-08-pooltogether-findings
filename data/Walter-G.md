### LiquidationPair
1) we could re-write it using yul
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L221-L223
```
// Define storage slots
let _amountInForPeriod := sload(0)
let _amountOutForPeriod := sload(1)
let _lastAuctionTime := sload(2)
let _emissionRate := sload(3)

// Load data from storage
let swapAmountIn := calldataload(0)
let _amountOut := sload(4)

// Calculate values
let convertedAmountOut := convert(_amountOut, int256)
let emissionRate := _emissionRate

let convertedAmountOutDivided := div(convertedAmountOut, emissionRate)
let auctionTimeToAdd := convert(convertedAmountOutDivided, uint256)

// Update storage
sstore(0, add(_amountInForPeriod, convert(swapAmountIn, uint96)))
sstore(1, add(_amountOutForPeriod, convert(_amountOut, uint96)))
sstore(2, add(_lastAuctionTime, convert(auctionTimeToAdd, uint48)))
```
2) yul
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L277-L281
```
// Define storage slot
let amount := sload(0)
let minimumAuctionAmount := sload(1)

// Compare amount with minimumAuctionAmount
if lt(amount, minimumAuctionAmount) {
    amount := 0
}
```
3) those three if statements could be replaced using yul
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L298-L300C6
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L302C5-L304
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L314-L316
in this way(for the first):
```
// Define storage slot
let _amountOut := sload(0)

// Compare _amountOut with zero
if eq(_amountOut, 0) {
    return(0)
}
```
for the second:
```
// Define storage slot
let _amountOut := sload(0)
let maxOut := sload(1)

// Compare _amountOut with maxOut
if gt(_amountOut, maxOut) {
    // Revert with error message
    revert(0, 0, 0)
}
```
and for the last one:
```
// Define storage slot
let purchasePrice := sload(0)
let _amountOut := sload(1)

// Compare purchasePrice with zero
if eq(purchasePrice, 0) {
    // Revert with error message
    revert(0, 0, _amountOut)
}
```
4) and the last report,all the _updateAuction function using yul:
```
// Define storage slots
let _amountInForPeriod := sload(0)
let _amountOutForPeriod := sload(1)
let _lastNonZeroAmountIn := sload(2)
let _lastNonZeroAmountOut := sload(3)
let _lastAuctionTime := sload(4)
let periodOffset := sload(5)
let periodLength := sload(6)
let __period := sload(7)
let _period := sload(8)
let targetFirstSaleTime := sload(9)
let _emissionRate := sload(10)
let decayConstant := sload(11)
let _initialPrice := sload(12)

// Check conditions
if and(gt(_amountInForPeriod, 0), gt(_amountOutForPeriod, 0)) {
    // Update non-zero amounts
    _lastNonZeroAmountIn := _amountInForPeriod
    _lastNonZeroAmountOut := _amountOutForPeriod
}

// Reset values
_amountInForPeriod := 0
_amountOutForPeriod := 0

// Update variables
_lastAuctionTime := add(periodOffset, mul(periodLength, __period))
_period := uint16(__period)

// Calculate emissionRate_
let emissionRate_ := _computeEmissionRate()
_emissionRate := emissionRate_

// Check _emissionRate
if gt(_emissionRate.unwrap(), 0) {
    // Calculate timeSinceLastAuctionStart
    let timeSinceLastAuctionStart := convert(int(uint(targetFirstSaleTime)))
    
    // Calculate purchaseAmount
    let purchaseAmount := mul(timeSinceLastAuctionStart, _emissionRate)
    
    // Calculate exchangeRateAmountInToAmountOut
    let exchangeRateAmountInToAmountOut := div(convert(int(uint(_lastNonZeroAmountIn))), convert(int(uint(_lastNonZeroAmountOut))))
    
    // Calculate price
    let price := mul(exchangeRateAmountInToAmountOut, purchaseAmount)
    
    // Calculate _initialPrice
    _initialPrice := ContinuousGDA.computeK(
        emissionRate_,
        decayConstant,
        timeSinceLastAuctionStart,
        purchaseAmount,
        price
    )
} else {
    _initialPrice := wrap(0)
}
```