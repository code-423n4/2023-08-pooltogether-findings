# Low

## [L-01] It is possible to create pair with equal tokenIn and tokenOut
### Impact 
It is possible to create pair with equal `tokenIn` and `tokenOut` address. 

### Recommended Mitigation Steps
Add additional check for that:

```solidity
if (tokenIn == tokenOut) revert EqualAddresses();
```


## [L-02] Zero deposit can be made
Zero deposits can be made into the `VaultBooster` contract. This will only update the `lastAccruedAt` to block.timestamp of the token.

Instance: [171](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L171)

### Recommended Mitigation Steps
Add additional check for that.


## [L-03] sequenceOffset_ can be zero
The variable `sequenceOffset_` should no be equal to zero.

Instance: [151](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L151)

### Recommended Mitigation Steps
Add additional check for that.


# Information

## [I-01] Forgotten comment
Forgotten comment in the codebase.
Instances: [276](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L276), [280](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L280)

## [I-02] Forgotten library
The code has a forgotten library import.

Instance: [4](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L4)

## [I-03] Include the old RNG address in the SetNextRngService event.
Include the old RNG address in the SetNextRngService `event`.

## [I-04] Redundant errors
In RngAuction has redundant errors: `AuctionDurationGteSequencePeriod`, `AuctionDurationZero` and `AuctionTargetTimeZero`.