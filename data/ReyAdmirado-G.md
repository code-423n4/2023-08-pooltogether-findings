| | issue |
| ----------- | ----------- |
| 1 | [State variables only set in the constructor should be declared immutable.](#1-state-variables-only-set-in-the-constructor-should-be-declared-immutableones-not-caught-by-bot) |
| 2 | [Structs can be packed into fewer storage slots](#2-structs-can-be-packed-into-fewer-storage-slots-ones-not-caught-by-bot) |
| 3 | [it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied](#3-it-costs-more-gas-to-initialize-non-constantnon-immutable-variables-to-zero-than-to-let-the-default-of-zero-be-applied) |
| 4 | [require() or revert() statements that check input arguments should be at the top of the function](#4-require-or-revert-statements-that-check-input-arguments-should-be-at-the-top-of-the-function) |
| 5 | [should use arguments instead of state variable](#5-should-use-arguments-instead-of-state-variable) |
| 6 | [Functions guaranteed to revert when called by normal users can be marked `payable`](#6-functions-guaranteed-to-revert-when-called-by-normal-users-can-be-marked-payable) |
| 7 | [Optimize names to save gas](#7-optimize-names-to-save-gas) |
| 8 | [Use hardcoded address instead address(this)](#8-use-hardcoded-address-instead-addressthis) |
| 9 | [Use of emit inside a loop](#9-use-of-emit-inside-a-loop) |

## 1. State variables only set in the constructor should be declared immutable.(ones not caught by bot)

Avoids a Gsset (20000 gas) in the constructor, and replaces each Gwarmaccess (100 gas) with a PUSH32 (3 gas). While strings are not value types, and therefore cannot be immutable/constant if not hard-coded outside of the constructor, the same behavior can be achieved by making the current contract abstract with virtual functions for the string accessors, and having a child contract override the functions with the hard-coded implementation-specific values.

- [RngRelayAuction.sol#L90](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L90)


## 2. Structs can be packed into fewer storage slots (ones not caught by bot)

Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings

can save one slot if we change sorting of struct. bring `lastAccruedAt` near `liquidationPair` to put them into one slot, because they are accessed in a function together so read is gonna be cheaper.
- [VaultBooster.sol#L29](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29)

bring `sequenceId` and `rngRequestId` near `recipient` to reduce slots used by 2 also accesses to them gonna be cheaper
- [RngAuction.sol#L](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L)


## 3. it costs more gas to initialize non-constant/non-immutable variables to zero than to let the default of zero be applied

- [RngRelayAuction.sol#L167](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167)


## 4. require() or revert() statements that check input arguments should be at the top of the function

Checks that involve constants should come before checks that involve state variables, function calls, and calculations. By doing these checks first, the function is able to revert before wasting a Gcoldsload (2100 gas*) in a function that may ultimately revert in the unhappy case.

the first revert is the most expensive one so we can put it last for some possible gas cost reduction
- [LiquidationPair.sol#L114](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L114)


## 5. should use arguments instead of state variable

This will save 100 gas because it gets rid of a storage read and instead uses a argument that we already have which gives the same result

we have the value of `_auctionDurationSeconds` inside `auctionDurationSeconds_` so use the argument one to save 100 gas by reducing one SLOAD
- [RngRelayAuction.sol#L119](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L119)


## 6. Functions guaranteed to revert when called by normal users can be marked `payable`

If a function modifier or require such as onlyOwner-admin is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided. The extra opcodes avoided are CALLVALUE(2), DUP1(3), ISZERO(3), PUSH2(3), JUMPI(10), PUSH1(3), DUP1(3), REVERT(0), JUMPDEST(1), POP(2) which costs an average of about 21 gas per call to the function, in addition to the extra deployment cost.

- [LiquidationRouter.sol#L63](https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L63)

- [VaultBooster.sol#L142](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142)
- [VaultBooster.sol#L148](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L148)
- [VaultBooster.sol#L217](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L217)
- [VaultBooster.sol#L242](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L242)


## 7. Optimize names to save gas

Contracts most called functions could simply save gas by function ordering via Method ID. Calling a function at runtime will be cheaper if the function is positioned earlier in the order (has a relatively lower Method ID) because 22 gas are added to the cost of a function for every position that came before it. The caller can save on gas if you prioritize most called functions.

See more [here](https://medium.com/joyso/solidity-how-does-function-name-affect-gas-consumption-in-smart-contract-47d270d8ac92).
you can use this [tool](https://emn178.github.io/solidity-optimize-name/) to get the optimized version for function and properties signatures 


## 8. Use hardcoded address instead address(this)

Instead of using `address(this)`, it is more gas-efficient to pre-calculate and use the hardcoded address. Foundry's `script.sol` and solmate's `LibRlp.sol` contracts can help achieve this. - [Reference](https://twitter.com/transmissions11/status/1518507047943245824)

- [VaultBooster.sol#L144](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144)
- [VaultBooster.sol#L173](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L173)
- [VaultBooster.sol#L190](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L190)
- [VaultBooster.sol#L276](https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L276)

- [RngAuction.sol#L180](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180)
- [RngAuction.sol#L182](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L182)


## 9. Use of emit inside a loop

Emitting an event inside a loop performs a LOG op N times, where N is the loop length. Consider refactoring the code to emit the event only once at the end of loop

- [RngRelayAuction.sol#L171](https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L171)