**./pt-v5-vault-boost/src/VaultBooster.sol**
- L222 - The operation amountAvailable - _amountOut could be unchecked, since it is validated in L219


**./pt-v5-draw-auction/src/RngAuction.sol**
- L34/37/54 - Three errors are created that are never used (AuctionDurationZero, AuctionTargetTimeZero, AuctionDurationGteSequencePeriod), therefore they should be eliminated.
