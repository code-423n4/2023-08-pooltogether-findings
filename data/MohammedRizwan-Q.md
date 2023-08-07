## Summary

### Low Risk Issues
|Number|Issue|Instances| |
|-|:-|:-:|:-:|
| [L&#x2011;01] | setOriginChainOwner() can be called more than once which is against Natspec | 1 |
| [L&#x2011;02] | Prevent createVaultBooster() from setting incorrect inputs | 1 |
| [L&#x2011;03] | update openzeppelin version to 4.9.3 | 1 |
| [L&#x2011;04] | unsafe downcast may lead to accounting errors  | 1 |

### [L&#x2011;01]  Prevent setOriginChainOwner() from calling more than once
In RemoteOwner.sol, Per the Natspec of setOriginChainOwner() it can be called only once, 

```Solidity
   * @dev Can only be called once.
```

However, with current impelementation it can be called more than once. To prevent this happening, a validation checks must be added to setOriginChainOwner() function.

There is [1 instance](https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L92-L99) of this issue:

```Solidity
File: src/RemoteOwner.sol

  function setOriginChainOwner(address _newOriginChainOwner) external {
    _checkSender();
    _setOriginChainOwner(_newOriginChainOwner);
  }
```

### Recommended Mitigation Steps

```diff
File: src/RemoteOwner.sol

+  bool initialize;

  function setOriginChainOwner(address _newOriginChainOwner) external {
+    require(!initialize, "can be called only once");
    _checkSender();
    _setOriginChainOwner(_newOriginChainOwner);
+    initialize = true;
  }
```
### [L&#x2011;02]  Prevent createVaultBooster() from setting incorrect inputs
In VaultBoosterFactory.sol, createVaultBooster() is used to create a new vault booster contract. As confirmed by sponsor, this can be called by anyone. However it has some missing sanity validations which should be added.

There is [1](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L28-L34) instance of this issue:

### Recommended Mitigation Steps

```diff

    function createVaultBooster(PrizePool _prizePool, address _vault, address _owner) external returns (VaultBooster) {
+       require(address(_prizePool) != address(0), "invalid address");
+       require(_vault != address(0), "invalid address");
+       require(_owner != address(0), "invalid address");

        VaultBooster booster = new VaultBooster(_prizePool, _vault, _owner);

        emit CreatedVaultBooster(booster, _prizePool, _vault, _owner);

        return booster;
    }
```

### [L&#x2011;03]  update openzeppelin version to 4.9.3
The contracts has used openzeppelin version 4.9.2 which is checked from package.json. External libraries which are being used in contracts must be upto date and should not have any known vulnerabilities.
Lates OZ features can be checked [here](https://github.com/OpenZeppelin/openzeppelin-contracts/releases)

### Recommended Mitigation Steps
Update openzeppelin version to 4.9.3

### [L&#x2011;04]  unsafe downcast may lead to accounting errors 
In contracts, while down casting the data type of data value, It is directly processed without any safety. As seen in contract, such unsafe downcasting can lead to accounting errors which can seriously hamper the overall system.

There is [3 instances](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L221-L223) of this issue:

### Recommended Mitigation Steps
Use openzeppelin safeCast.sol for safe casting.
