## GAS-1: Caching global variables is more expensive than using the actual variable

### Description

 It’s cheaper to use global variable as compared to caching.

### Affected file

* LiquidationPair.sol (Line: 378)

## GAS-2: Change ```public``` state variable visibility to ```private```

### Description

If it is preferred to change the visibility of the ```owner``` state variable to private, this will save significant gas.

### Affected file

* RngAuctionRelayerRemoteOwner.sol (Line: 30)

## GAS-3: Divisions which do not divide by -X can be unchecked

### Description

Divisions with a positive value cannot underflow or overflow

### Affected file

* LiquidationPair.sol (Line: 382)
* RngAuction.sol (Line: 374)

## GAS-4: Duplicated require()/revert() checks should be refactored to a modifier or function

### Description

This saves deployment gas.

### Affected file

* RemoteOwner.sol (Line: 71)
* RngAuction.sol (Line: 149, 371, 383)
* RngAuctionRelayerDirect.sol (Line: 40)
* RngRelayAuction.sol (Line: 113)

## GAS-5: Functions guaranteed to revert when called by normal users can be marked payable

### Description

If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

### Affected file

* LiquidationRouter.sol (Line: 63)
* RngAuction.sol (Line: 347)
* VaultBooster.sol (Line: 142, 188, 211, 242)

## GAS-6: Public functions not called by the contract should be declared external instead

### Description

Contracts are allowed to override their parents’ functions and change the visibility from external to public and can save gas by doing so. 

### Affected file

* AddressRemapper.sol (Line: 32)

## GAS-7: Replace modifier with function

### Description

Modifiers make code more elegant, but cost more than normal functions.

### Affected file

* LiquidationRouter.sol (Line: 84)
* VaultBooster.sol (Line: 283, 292)

## GAS-8: Should use arguments instead of state variable

### Description

Using function's parameter cost less gas then a state variable.

### Affected file

* LiquidationPair.sol (Line: 118, 118, 119, 119)
* RngRelayAuction.sol (Line: 119)
* VaultBooster.sol (Line: 124)

## GAS-9: Use ```assembly``` to write address storage values

### Affected file

* LiquidationPair.sol (Line: 105, 106, 107, 108, 109, 110, 111, 130, 131, 132, 334, 335, 337, 338, 339, 340, 342, 349, 357)
* LiquidationRouter.sol (Line: 50)
* RemoteOwner.sol (Line: 57, 106)
* RngAuction.sol (Line: 150, 151, 152, 153, 154, 192, 435)
* RngAuctionRelayer.sol (Line: 25)
* RngAuctionRelayerRemoteOwner.sol (Line: 46, 47, 48)
* RngRelayAuction.sol (Line: 109, 116, 117, 118, 145)
* VaultBooster.sol (Line: 123, 124, 125)

## GAS-10: Use calldata instead of memory for function parameters

### Description

If a reference type function parameter is read-only, it is cheaper in gas to use calldata instead of memory. Calldata is a non-modifiable, non-persistent area where function arguments are stored, and behaves mostly like memory.

### Affected file

* RemoteOwnerCallEncoder.sol (Line: 7)
* RewardLib.sol (Line: 58, 79)

## GAS-11: Use hardcoded address instead address(this)

### Description

Instead of using ```address(this)```, it is more gas-efficient to pre-calculate and use the hardcoded ```address```

### Affected file

* RngAuction.sol (Line: 180, 182)
* VaultBooster.sol (Line: 144, 173, 190, 276)

## GAS-12: Using bools for storage incurs overhead

### Description

Booleans are more expensive than uint256 or any type that takes up a full word because each write operation emits an extra SLOAD to first read the slot’s contents, replace the bits taken up by the boolean, and then write back. This is the compiler’s defense against contract upgrades and pointer aliasing, and it cannot be disabled. Use uint256(1) and uint256(2) for true/false instead.

### Affected file

* LiquidationPairFactory.sol (Line: 51)

## GAS-13: With assembly, ```.call (bool success)``` transfer can be done gas-optimized

### Description

```return``` data ```(bool success,)``` has to be stored due to EVM architecture, but in a usage in assembly, 'out' and 'outsize' values are given (0,0), this storage disappears and gas optimization is provided.

### Affected file

* RemoteOwner.sol (Line: 67)
* RngAuctionRelayerDirect.sol (Line: 39)