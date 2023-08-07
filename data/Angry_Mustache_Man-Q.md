[low] 

Title: minAuctionAmount of LiquidationPair.sol does not provide enough flexibilty  

Links to Code:  

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L277 

Impact: 

It is said in comments at https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L53 

That `minAuctionAmount` is used for efficiency but as it is made an immutable variable , it is not changeable by owner after deployment. During times of high gas cost, minAuctionAmount should have the flexibility to be increased. 

Recommendation: 

Change minAuctionAmount variable to a constant and include a function for only owner to vary it. 

[NC]  

Title: Wrong name given to exchangeRateAmount 

Name should be changed from exchangeRateAmountInToAmountOut to exchangeRateAmountOutToAmountIn at : https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L347 

As calculation involved is finding `_lastNonZeroAmountIn` divided by `_lastNonZeroAmountOut` 