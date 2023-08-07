
## [N-01] For modern and more readable code; update import usages

>Solidity code is also cleaner in another way that might not be noticeable: the struct Point. We were importing it previously with global import but not using it. The Point struct polluted the source code with an unnecessary object we were not using because we did not need it.
This was breaking the rule of modularity and modular programming: only import what you need Specific imports with curly braces allow us to apply this rule better.
> ### *Recommendation*
> `import {contract1 , contract2} from "filename.sol";`

```Solidity
5: import "./LiquidationPair.sol";
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L5

```Solidity
4: import "forge-std/console2.sol";
```

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L4

## [N-02] For immutable variables in Solidity, the convention is to use CAPS_CASE or UPPER_CASE_WITH_UNDERSCORES.
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L102C2-L109C34

## [N-03] Use a non-legacy and more battle-tested version
Use 0.8.10

`pragma solidity ^0.8.0;`

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L2

## [N-04] Generate perfect code headers every time
Description:
I recommend using header for Solidity code layout and readability

```
/*//////////////////////////////////////////////////////////////
                           TESTING 123
//////////////////////////////////////////////////////////////*/
```

## [N-05] Function writing that does not comply with the Solidity Style Guide
Context
All Contracts

Description
Order of Functions; ordering helps readers identify which functions they can call and to find the constructor and fallback definitions easier. But there are contracts in the project that do not comply with this.

https://docs.soliditylang.org/en/v0.8.17/style-guide.html

Functions should be grouped according to their visibility and ordered:

constructor
receive function (if exists)
fallback function (if exists)
external
public
internal
private
within a grouping, place the view and pure functions last


## [N-06] Use SMTChecker
The highest tier of smart contract behavior assurance is formal mathematical verification. All assertions that are made are guaranteed to be true across all inputs → The quality of your asserts is the quality of your verification.

##  [N-07] Inconsistent Solidity Versions
 
 `pragma solidity ^0.8.0;`

https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBoosterFactory.sol#L2

all other use 0.8.19 

`pragma solidity 0.8.19;`
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L2
  
  **Recommendation**
Versions must be consistent with each other.


## [N-08] Pragma version^0.8.19 version too recent to be trusted.
https://github.com/ethereum/solidity/blob/develop/Changelog.md
0.8.19 (2023-02-22)
0.8.17 (2022-09-08)
0.8.16 (2022-08-08)
0.8.15 (2022-06-15)
0.8.10 (2021-11-09)

Unexpected bugs can be reported in recent versions;

- Risks related to recent releases
- Risks of complex code generation changes
- Risks of new language features
- Risks of known bugs
Use a non-legacy and more battle-tested version
Use 0.8.10

##  [S-01] You can explain the operation of critical functions in NatSpec with an infographic.