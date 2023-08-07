1 - The emission rate in the `LiquidationPair` is not updated when the vault balance is updated
==

The [LiquidationPair.swapExactAmountOut()](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L211) helps to anyone to introduce `prize tokens` and get the underlying `token out`.

When the Auction is started, it [calculates the emission rate](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L341) using the [current liquidatable balance](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L275C1-L275C1). Once the auction period has started, the [swapExactAmountOut()](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L216) function will not update the emission rate because the [_checkUpdateAuction()](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L322C12-L322C31) function only allow the data calculation once per period.

The problem is that the `liquidatable balance` can be changed while the auction is in execution causing the wrong data in [emission rate](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L274) and the wrong calculation in the [swap purchase price](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L308).

The vault's liquidatable balance can be changed becuase anyone can [deposit](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L171) to the vault and affect the accrue balance.

The emission rate should be recalculated if the vault liquidatable balance is changed.

2 - There is not validation that the `LiquidationPair` period length and offest match with the `prize pool` draw length and offset.
==

The [deployment documentation](https://github.com/code-423n4/2023-08-pooltogether#deployment-configuration) says: *The pair's period length and offset will match the prize pool draw length and offset.*. So in the deployment, there will be a validation for that. The problem is that after that anyone can create a LiquidationPair using the [LiquidationPairFactory](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L65) and those checks are not in the `LiquidationPair` contract. Those validations are crucial because the liquidation auctions should be syncronized with the `prize pool`.

Add a validation that the `prize pool` draw length and offsett must match with the `LiquidationPair` period length and offset. The validation should occur in the [liquidation pair creation](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L109-L110).

