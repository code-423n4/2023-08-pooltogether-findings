**./pt-v5-cgda-liquidator/src/LiquidationPair.sol**
- L105/106/107 - No validation is performed in the constructor and the variables (source, tokenIn and tokenOut) are immutable, therefore they should be validated before setting the variable to != 0x.

- L276/280 - There is commented code, which should be removed, since it is not used and generates a reduction in the understanding of the code.


**./pt-v5-cgda-liquidator/src/libraries/ContinuousGDA.sol**
- L33/39/41/65/67/87/88 - A division is made by an input, which could be zero, therefore it should be validated before.


**./pt-v5-draw-auction/src/RngAuctionRelayerRemoteOwner.sol**
- L46/47/48 - No validation is performed in the constructor and the variables (messageDispatcher, toChainId, account) are immutable, therefore they should be validated before setting the variable to != 0x.


**./pt-v5-draw-auction/src/abstract/RngAuctionRelayer.sol**
- L25 - No validation is performed in the constructor and the rngAuction variable is immutable, therefore it should be validated before setting the variable to != 0x.


**./pt-v5-draw-auction/src/libraries/RewardLib.sol**
- L31 - A division is performed by an input, which could be zero, therefore it should be validated before.


**./pt-v5-vault-boost/src/VaultBooster.sol**
- L123/124/125 - No validation is performed in the constructor and the variables (prizePool, twabController, vault) are immutable, therefore they should be validated before setting the variable to != 0x.


**./remote-owner/src/RemoteOwner.sol**
- L64/65/68/69/70 - There is commented code, which should be removed, since it is not used and generates a reduction in the understanding of the code.


**./remote-owner/src/libraries/RemoteOwnerCallEncoder.sol**
- L14/15/16 - There is commented code, which should be eliminated, since it is not used and generates a reduction in the understanding of the code.
