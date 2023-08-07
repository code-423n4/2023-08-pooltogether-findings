1) LiquidationPairFactory instance does not prevent re-deploying the same Liquidation Pair. CreatePair can be called for same pair any number of times. This will result in duplicate entries in all pairs array.


2) Down casting in LiquidationPair::swapExactAmountOut() will result in loss of data.
   example:
   ```
      _amountInForPeriod += uint96(swapAmountIn);
      _amountOutForPeriod += uint96(_amountOut);
     _lastAuctionTime += uint48(uint256(convert(convert(int256(_amountOut)).div(_emissionRate))));
   ```

   