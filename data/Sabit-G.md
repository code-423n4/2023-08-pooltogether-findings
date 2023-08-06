1. Using bool for storage incurs overhead

avoid a Gwarmaccess (100 gas) for the extra SLOAD, and avoid Gsset (20000 gas) when changing from false to true, after having been true in the past.

https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L51
