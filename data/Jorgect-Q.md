# LOW ISSUES REPORT

## [L-01] TAKING ALL THE FEE IN CASE THAT THE CONTRACT HAS TOKEN TO PAY IT.

The rngAuction contract is paying the fee in case that it have it

```
function startRngRequest(address _rewardRecipient) external {
    ...
    (address _feeToken, uint256 _requestFee) = rng.getRequestFee();
    if (_feeToken != address(0) && _requestFee > 0) {
      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {
        // Transfer tokens from caller to this contract before continuing
        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee);
      }
    ...

```
https://github.com/GenerationSoftware/pt-v5-draw-auction/blob/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/RngAuction.sol#L170C3-L209C1

in case that the balance is to low, those token in gonna remain in the contract.

recomendation:

Take just the token that the user need it to complete the fee and not the whole fee:

```
function startRngRequest(address _rewardRecipient) external {
    ...
    (address _feeToken, uint256 _requestFee) = rng.getRequestFee();
    if (_feeToken != address(0) && _requestFee > 0) {
      if (IERC20(_feeToken).balanceOf(address(this)) < _requestFee) {
        // Transfer tokens from caller to this contract before continuing
        IERC20(_feeToken).transferFrom(msg.sender, address(this), _requestFee-IERC20(_feeToken).balanceOf(address(this))); // Take just the token that the user need it to complete the fee
      }
    ...

```

## [L-02] user can create a boots sending tokens to the contract address and then depositing.

The boots shoud be just create by the owner however is a user send tokens to the contract and then call deposit token a pass the token the contract its gonna accrued that token and create some varibales in the struck.

we can analize the functions deposit; if we call this function whit the _token and with 0 amount the contract is gonna accure that token. 

```
file:pt-v5-vault-boost/src/VaultBooster.sol

function deposit(IERC20 _token, uint256 _amount) external {
    _accrue(_token);
    _token.safeTransferFrom(msg.sender, address(this), _amount);

    emit Deposited(_token, msg.sender, _amount);
  }
```
https://github.com/GenerationSoftware/pt-v5-vault-boost/blob/9d640051ab61a0fdbcc9500814b7f8242db9aec2/src/VaultBooster.sol#L171C3-L176C4

```
 function _accrue(IERC20 _tokenOut) internal returns (uint256) {
    uint256 available = _computeAvailable(_tokenOut);
    _boosts[_tokenOut].available = available.toUint144();
    _boosts[_tokenOut].lastAccruedAt = uint48(block.timestamp);

    emit BoostAccrued(_tokenOut, available);

    return available;
  }
```

the _accrue function is gonna call _computeAvailable which take the balance of the token in the contract, do some calulations and return the available, this function is then creating a _boosts with available and lastAccruedAt values, even if i could no be able to find a big issue more than tricky the owner in some cases, it´s still something to avoid.

recomendation:

check in the _accrue function if the _boosts[tokenOut].liquidationPair is equal to 0 and if it is revert.


 
