# gas

# summary

|      |  issue  | instance |
|------|---------|----------|
|[G-01]|Use do while loops instead of for loops|1|
|[G-02]|Functions Guaranteed to revert when called by normal users can be markd payable|5|
|[G-03]|Use calldata instead of memory for function arguments that do not get mutated|1|
|[G-04]|Internal functions not called by the contract should be removed to save deployment gas|1|
|[G-05]|Use assembly to write address storage values|5|
|[G-06]|Missing zero address check in constructor|4|
|[G-07]|Do not calculate constants|1|
|[G-08]|use Mappings Instead of Arrays|1|
|[G-09]|Avoid contract existence checks by using low level calls|6|
|[G-10]|Stack variable used as a cheaper cache for a state variable is only used once|1|
|[G-11]|Can Make The Variable Outside The Loop To Save Gas|1|
|[G-12]|>= costs less gas than >|11|
|[G-13]|Use nested if statements instead of &&|2|
|[G-14]|Structs can be packed into fewer storage slots by editing variables|2|
|[G-15]|Ternary operation is cheaper than if-else statement|2|
|[G-16]|Use hardcode address instead address(this)|6|
|[G-17]| Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it)|1|


## [G-01] Use do while loops instead of for loops
A do while loop will cost less gas since the condition is not being checked for the first iteration.


```solidity
File: libraries/RewardLib.sol
65   for (uint256 i; i < _auctionResultsLength; i++) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L65-L68

```solidity
File: src/RngRelayAuction.sol
167    for (uint8 i = 0; i < _rewards.length; i++) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L167



## [G‑02] Functions guaranteed to revert when called by normal users can be marked payable
If a function modifier such as onlyOwner is used, the function will revert if a normal user tries to pay the function. Marking the function as payable will lower the gas cost for legitimate callers because the compiler will not include checks for whether a payment was provided.

```solidity
File: src/LiquidationRouter.sol
68  ) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L68

```solidity
File: src/VaultBooster.sol
142  function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {

188  function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {

217  ) external override onlyPrizeToken(_tokenIn) onlyLiquidationPair(_tokenOut) returns (bool) {

242  function targetOf(address _tokenIn) external view override onlyPrizeToken(_tokenIn) returns (address) {
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L142

## [G‑03] Use calldata instead of memory for function arguments that do not get mutated
When you specify a data location as memory, that value will be copied into memory. When you specify the location as calldata, the value will stay static within calldata. If the value is a large, complex type, using memory may result in extra memory expansion costs.

```solidity
File: src/libraries/RewardLib.sol
59    AuctionResult[] memory _auctionResults,
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L59


## [G‑04] internal functions not called by the contract should be removed to save deployment gas
If the functions are required by an interface, the contract should inherit from that interface and use the override keyword.

```solidity
File: src/abstract/RngAuctionRelayer.sol
31    function encodeCalldata(address rewardRecipient) internal returns (bytes memory) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/RngAuctionRelayer.sol#L31


## [G‑05] Use assembly to write address storage values
By using assembly to write to address storage values, you can bypass some of these operations and lower the gas cost of writing to storage. Assembly code allows you to directly access the Ethereum Virtual Machine (EVM) and perform low-level operations that are not possible in Solidity.

example of using assembly to write to address storage values:
```
contract MyContract {
    address private myAddress;
    
    function setAddressUsingAssembly(address newAddress) public {
        assembly {
            sstore(0, newAddress)
        }
    }
}
```

```solidity
File: src/LiquidationPair.sol
106    tokenIn = _tokenIn;

107    tokenOut = _tokenOut;
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L106

```solidity
File: src/RngRelayAuction.sol
116    rngAuctionRelayer = _rngAuctionRelayer;
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L116

```solidity
File: src/VaultBooster.sol
125    vault = _vault;
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L125

```solidity
File: src/RemoteOwner.sol
106    _originChainOwner = _newOriginChainOwner;
```
https://github.com/GenerationSoftware/remote-owner/blob/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L106

## [G-06] Missing zero address check in constructor
Missing checks for zero-addresses may lead to infunctional protocol, if the variable addresses are updated incorrectly. It also wast gas as it requires the redeployment of the contract.

```solidity
File: src/LiquidationPair.sol
95     address _tokenIn, 

96     address _tokenOut,
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/tree/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L95-L96


```solidity
File: src/RngAuction.sol
142     address owner_, 
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L142

```solidity
File: src/VaultBooster.sol
119    PrizePool _prizePool,

120     address _vault, 

121     address _owner 

123     prizePool = _prizePool;
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/tree/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L119-L123

```solidity
File: src/RemoteOwner.sol
53       address executor_, 

54       address __originChainOwner
```
https://github.com/GenerationSoftware/remote-owner/tree/285749ab51e98afc8ebb4e4049a4348d669a3e9d/src/RemoteOwner.sol#L53-L54


## [G‑07] Do not calculate constants
Due to how constant variables are implemented (replacements at compile-time), an expression assigned to a constant variable is recomputed each time that the variable is used, which wastes some gas
```solidity
File:src/libraries/ContinuousGDA.sol
14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14


## [G-08] use Mappings Instead of Arrays
Arrays are useful when you need to maintain an ordered list of data that can be iterated over, but they have a higher gas cost for read and write operations, especially when the size of the array is large. This is because Solidity needs to iterate over the entire array to perform certain operations, such as finding a specific element or deleting an element.

Mappings, on the other hand, are useful when you need to store and access data based on a key, rather than an index. Mappings have a lower gas cost for read and write operations, especially when the size of the mapping is large, since Solidity can perform these operations based on the key directly, without needing to iterate over the entire data structure.
```solidity
File: src/LiquidationPairFactory.sol
43  LiquidationPair[] public allPairs;
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPairFactory.sol#L43

## [G‑09] Avoid contract existence checks by using low level calls
Prior to 0.8.10 the compiler inserted extra code, including EXTCODESIZE (100 gas), to check for contract existence for external function calls. In more recent solidity versions, the compiler will not insert these checks if the external call has a return value. Similar behavior can be achieved in earlier versions by using low-level calls, since low level calls never check for contract existence.
```solidity
File: src/LiquidationPair.sol
141    return source.targetOf(tokenIn);
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L141

```solidity
File: src/LiquidationRouter.sol
69    IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L69

```solidity
File: src/RngAuction.sol
180      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {


182        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);


185      IERC20(_feeToken).safeIncreaseAllowance(address(rng), _requestFee);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L180

```solidity
File: src/VaultBooster.sol
226    IERC20(_tokenOut).safeTransfer(_account, _amountOut);
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L226



## [G‑10] Stack variable used as a cheaper cache for a state variable is only used once
If the variable is only accessed once, it's cheaper to use the state variable directly that one time, and save the 3 gas the extra stack assignment would spend

```solidity
File: src/libraries/ContinuousGDA.sol
14   SD59x18 internal constant ONE = SD59x18.wrap(1e18);
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L14



## [G‑11] Can Make The Variable Outside The Loop To Save Gas 
When you declare a variable inside a loop, Solidity creates a new instance of the variable for each iteration of the loop. This can lead to unnecessary gas costs, especially if the loop is executed frequently or iterates over a large number of elements.

By declaring the variable outside the loop, you can avoid the creation of multiple instances of the variable and reduce the gas cost of your contract. Here's an example:

```
contract MyContract {
    function sum(uint256[] memory values) public pure returns (uint256) {
        uint256 total = 0;
        
        for (uint256 i = 0; i < values.length; i++) {
            total += values[i];
        }
        
        return total;
    }
}
```

```solidity
File: src/RngRelayAuction.sol
168    uint104 _reward = uint104(_rewards[i]);
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L168


## [G-12] >= costs less gas than >
The compiler uses opcodes GT and ISZERO for solidity code that uses >, but only requires LT for >=, which saves 3 gas

```solidity
File: src/RngRelayAuction.sol
113    if (auctionTargetTime_ > auctionDurationSeconds_) {

140    if (_auctionElapsedSeconds > (_auctionDurationSeconds-1)) revert AuctionExpired();

169      if (_reward > 0) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L113

```solidity
File: src/libraries/ContinuousGDA.sol
38    if (_emissionRate.unwrap() > 1e18) {
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38


```solidity
File: src/LiquidationPair.sol
114    if (_decayConstant.mul(period59).unwrap() > uEXP_MAX_INPUT) {

218    if (swapAmountIn > _amountInMax) {

302    if (_amountOut > maxOut) {

332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L114

```solidity
File: src/RngAuction.sol
149    if (auctionTargetTime_ > auctionDurationSeconds_) revert AuctionTargetTimeExceedsDuration(uint64(auctionTargetTime_), uint64(auctionDurationSeconds_));

176    if (_auctionElapsedTimeSeconds > auctionDuration) revert AuctionExpired();

179    if (_feeToken != address(0) && _requestFee > 0) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L149


## [G‑13] Use nested if statements instead of &&
If the if statement has a logical AND and is not followed by an else statement, it can be replaced with 2 if statements.
```
contract NestedIfTest {

    //Execution cost: 22334 gas
    function funcBad(uint256 input) public pure returns (string memory) { 
       if (input<10 && input>0 && input!=6){ 
           return "If condition passed";
       } 

   }

    //Execution cost: 22294 gas
    function funcGood(uint256 input) public pure returns (string memory) { 
    if (input<10) { 
        if (input>0){
            if (input!=6){
                return "If condition passed";
            }
        }
    }
   }
}
   
```


```solidity
File: src/LiquidationPair.sol
332    if (_amountInForPeriod > 0 && _amountOutForPeriod > 0) {
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L332

```solidity
File: src/RngAuction.sol
179    if (_feeToken != address(0) && _requestFee > 0) {
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L179


## [G‑14] Structs can be packed into fewer storage slots by editing variables
Each slot saved can avoid an extra Gsset (20000 gas) for the first setting of the struct. Subsequent reads as well as writes have smaller gas savings.

```solidity
File: src/VaultBooster.sol
struct Boost {
  address liquidationPair;
  UD2x18 multiplierOfTotalSupplyPerSecond;
  uint96 tokensPerSecond;
  uint144 available;
  uint48 lastAccruedAt;
}
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L29-L35

```solidity
File: src/RngAuction.sol
struct RngAuctionResult {
  address recipient;
  UD2x18 rewardFraction;
  uint32 sequenceId;
  RNGInterface rng;
  uint32 rngRequestId;
}
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L23-L29

## [G-15] Ternary operation is cheaper than if-else statement
ternary operations can be more gas-efficient than if-else statements. This is because ternary operations are often more concise and can be optimized by the compiler to generate more efficient bytecode.
```solidity
File: src/abstract/AddressRemapper.sol
    if (_destinationAddress[_addr] == address(0)) {
      return _addr;
    } else {
      return _destinationAddress[_addr];
    }
```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/abstract/AddressRemapper.sol#L33-L37

```solidity
File: src/libraries/ContinuousGDA.sol

    if (_emissionRate.unwrap() > 1e18) {
      result = _k.div(_emissionRate).mul(topE).div(bottomE);
    } else {
      result = _k.mul(topE.div(_emissionRate.mul(bottomE)));
    }
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/libraries/ContinuousGDA.sol#L38-L42

## [G-16] Use hardcode address instead address(this)
it can be more gas-efficient to use a hardcoded address instead of the address(this) expression, especially if you need to use the same address multiple times in your contract.

The reason for this is that using address(this) requires an additional EXTCODESIZE operation to retrieve the contract's address from its bytecode, which can increase the gas cost of your contract. By pre-calculating and using a hardcoded address, you can avoid this additional operation and reduce the overall gas cost of your contract.

Here's an example of how you can use a hardcoded address instead of address(this):

```
contract MyContract {
    address public myAddress = 0x1234567890123456789012345678901234567890;
    #L
    function doSomething() public {
        // Use myAddress instead of address(this)
        require(msg.sender == myAddress, "Caller is not authorized");
        
        // Do something
    }
}
```
In the above example, we have a contract MyContract with a public address variable myAddress. Instead of using address(this) to retrieve the contract's address, we have pre-calculated and hardcoded the address in the variable. This can help to reduce the gas cost of our contract and make our code more efficient.

[References](https://book.getfoundry.sh/reference/forge-std/compute-create-address)

```solidity
File: src/VaultBooster.sol
144      uint256 balance = _token.balanceOf(address(this));

173    _token.safeTransferFrom(msg.sender, address(this), _amount);

190    uint256 availableBalance = _token.balanceOf(address(this));

276    uint256 availableBalance = _tokenOut.balanceOf(address(this));
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L144


## [G‑17] Caching global variables is more expensive than using the actual variable (use block.timestamp instead of caching it)

```solidity
File:  src/LiquidationPair.sol
378    uint256 _timestamp = block.timestamp;
```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L378
