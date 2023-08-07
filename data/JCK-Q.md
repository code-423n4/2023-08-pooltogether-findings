
# LOW RISK

| Number | Issue | Instances |
|--------|-------|-----------|
|[LOW-01]| Execution at deadlines should be allowed  | 2 | 
|[LOW-02]| Large transfers may not work with some ERC20 tokens | 5 | 
|[LOW-03]| Missing limits when setting min/max amounts | 3 | 

## [L-01] Execution at deadlines should be allowed

The condition may be wrong in these cases, as when block.timestamp is equal to the compared > or < variable these blocks will not be executed.

There are 2 instances of this issue.

```solidity
file:  src/LiquidationPair.sol

288    if (block.timestamp < _lastAuctionTime) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationPair.sol#L288

```solidity
file:  src/RngRelayAuction.sol

139    uint64 _auctionElapsedSeconds = uint64(block.timestamp < _rngCompletedAt ? 0 : block.timestamp - _rngCompletedAt);

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngRelayAuction.sol#L139


## [L-02] Large transfers may not work with some ERC20 tokens

Some IERC20 implementations (e.g UNI, COMP) may fail if the valued transferred is larger than uint96. Source

There are 5 instances of this issue.

```solidity
file:  src/VaultBooster.sol

173   _token.safeTransferFrom(msg.sender, address(this), _amount);

193   _token.transfer(msg.sender, _amount);

226   IERC20(_tokenOut).safeTransfer(_account, _amountOut);

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L173

```solidity
file:  src/LiquidationRouter.sol

96      IERC20(_liquidationPair.tokenIn()).safeTransferFrom(
      msg.sender,
      _liquidationPair.target(),
      _liquidationPair.computeExactAmountIn(_amountOut)
    );

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L96-L100

```solidity
file:  src/RngAuction.sol

182    IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L182


## [L-03] Missing limits when setting min/max amounts

There are some missing limits in these functions, and this could lead to unexpected scenarios. Consider adding a min/max limit for the following values, when appropriate.

There are 3 instances of this issue.


```solidity 
file:  src/LiquidationRouter.sol

63     function swapExactAmountOut(
    LiquidationPair _liquidationPair,
    address _receiver,
    uint256 _amountOut,
    uint256 _amountInMax
  ) external onlyTrustedLiquidationPair(_liquidationPair) returns (uint256) {

```
https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol#L63-68

```solidity
file:  src/VaultBooster.sol

142    function setBoost(IERC20 _token, address _liquidationPair, UD2x18 _multiplierOfTotalSupplyPerSecond, uint96 _tokensPerSecond, uint144 _initialAvailable) external onlyOwner {

189    function withdraw(IERC20 _token, uint256 _amount) external onlyOwner {

```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L124

