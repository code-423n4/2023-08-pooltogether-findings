# GAS OPTMIZATIONS

## [G-01] ADD UNCHECKED TO SAVE GAS IN LiquidationPair.sol

When the contract it´s ensuring that a variable never undeflow you can add unchecked and save some gas:

```
file: Breadcrumbspt-v5-cgda-liquidator/src/LiquidationPair.sol

  function _getElapsedTime() internal view returns (SD59x18) {
    if (block.timestamp < _lastAuctionTime) {
      return wrap(0);
    }
    return convert(int256(block.timestamp)).sub(convert(int256(uint256(_lastAuctionTime))));
  }

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L287C3-L292C4

The contract it´s already checking if the block.timestamp is less than _lastAuctionTime so you can add the next unchecked:

```
  function _getElapsedTime() internal view returns (SD59x18) {
    if (block.timestamp < _lastAuctionTime) {
      return wrap(0);
    }
unchecked {
    return convert(int256(block.timestamp)).sub(convert(int256(uint256(_lastAuctionTime))));
}
  }

```

## [G-02] DOUBLE CASTING IN VaultBooster.sol

The contrac is performing a casting that it doesn´t need:

```
file:pt-v5-vault-boost/src/VaultBooster.sol

function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {
    ...
    _boosts[IERC20(_token)].available = (availableBoost > remainingBalance ? remainingBalance : availableBoost).toUint144();
    ...
  }
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L188C3-L196C4

the input is and IRC20 for the _token value but its casting again more later in the _boosts
