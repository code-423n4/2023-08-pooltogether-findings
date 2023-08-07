## Overview

|#|Issue|
|-|:-|
| [01] | Missing zero address check for liquidation pair | 
|[02]|Ensure that `sequenceOffset_ < block.timestamp`|
|[03]|Check that the `liquidationPair` is deployed from `liquidationFactory`|
|[04]|Ensure that `sequenceOffset_ < block.timestamp`|

### [01] Missing zero address check for liquidation pair
- https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142-L165
- Zero address check for liquidation pair is missing in `VaultBooster::setBoost()`.
- The liquidator's calls will be reverted if `address(liquidationPair)==address(0)`

### [02] Ensure that `sequenceOffset_ < block.timestamp`
- https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/main/src/RngAuction.sol#L144
- In `RngAuction.sol`, while setting parameters in the constructor, it must be ensured that `sequenceOffset_ < block.timestamp` other wise the auction won't start until `sequenceOffset`.
- This might lead to unexpected outcomes, such as not being able to start the auction if and when necessary. The contract will need to be deployed again if it's important.

### [03] Check that the `liquidationPair` is deployed from `liquidationFactory` in `VaultBooster`
- https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L150
- When the end-users calls `swapExactAmountOut` on `LiquidationRouter`, it checks that whether the `LiquidationPair` being called is deployed by `LiquidationFactory` or not
```solidity
modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) {
	if (!_liquidationPairFactory.deployedPairs(_liquidationPair)) {
      revert UnknownLiquidationPair(_liquidationPair);
    }
    _;
}
```
- In `VaultBooster.sol`, while adding a liquidation pair using `setBoost()`, it does not check whether the liquidation pair is deployed by the factory.
```solidity
  function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {
    if (_initialAvailable > 0) {
      uint256 balance = _token.balanceOf(address(this));
      if (balance < _initialAvailable) {
        revert InitialAvailableExceedsBalance(_initialAvailable, balance);
      }
    }
    _boosts[_token] = Boost({
      liquidationPair: _liquidationPair,
      multiplierOfTotalSupplyPerSecond: _multiplierOfTotalSupplyPerSecond,
      tokensPerSecond: _tokensPerSecond,
      available: _initialAvailable,
      lastAccruedAt: uint48(block.timestamp)
    });

    emit SetBoost(
      _token,
      _liquidationPair,
      _multiplierOfTotalSupplyPerSecond,
      _tokensPerSecond,
      _initialAvailable,
      uint48(block.timestamp)
    );
  }
```
- If a liquidation pair is set that is not deployed by factory, then all calls by end-users will revert. Rendering the `VaultBooster` temporarily unusable.
- To ensure the robustness of this decentralized protocol, it is recommended to add check in `setBoost()` to ensure that `_liquidationPair` belongs to `LiquidationFactory`

### [02] Ensure that `sequenceOffset_ < block.timestamp`
- https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L110
- In `LiquidationPair.sol`, while setting parameters in the constructor, it must be ensured that `_periodOffset < block.timestamp` other wise the liquidations won't be possible until `_periodOffset`.