# VULN 1 

## [LOW] Use .call instead of .transfer to send ether
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 193 at 2023-08-pooltogether/src/VaultBooster.sol:

    _token.transfer(msg.sender, _amount);

------------------------------------------------------------------------ 

### Mitigation 

.transfer will relay 2300 gas and .call will relay all the gas. If the receive/fallback function from the recipient proxy contract has complex logic, using .transfer will fail, causing integration issues.Replace .transfer with .call. Note that the result of .call need to be checked.










# VULN 2 

## [LOW] Immutables should be in uppercase
------------------------------------------------------------------------ 

### Proof of concept 

Found in line 75 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  PrizePool public immutable prizePool;


Found in line 79 at 2023-08-pooltogether/src/RngRelayAuction.sol:

  address public immutable rngAuctionRelayer;


Found in line 41 at 2023-08-pooltogether/src/RemoteOwner.sol:

  uint256 internal immutable _originChainId;


Found in line 103 at 2023-08-pooltogether/src/VaultBooster.sol:

  PrizePool public immutable prizePool;


Found in line 106 at 2023-08-pooltogether/src/VaultBooster.sol:

  TwabController public immutable twabController;


Found in line 109 at 2023-08-pooltogether/src/VaultBooster.sol:

  address public immutable vault;


Found in line 79 at 2023-08-pooltogether/src/RngAuction.sol:

  uint64 public immutable auctionDuration;


Found in line 82 at 2023-08-pooltogether/src/RngAuction.sol:

  uint64 public immutable auctionTargetTime;


Found in line 85 at 2023-08-pooltogether/src/RngAuction.sol:

  UD2x18 internal immutable _auctionTargetTimeFraction;


Found in line 89 at 2023-08-pooltogether/src/RngAuction.sol:

  uint64 public immutable sequencePeriod;


Found in line 97 at 2023-08-pooltogether/src/RngAuction.sol:

  uint64 public immutable sequenceOffset;


Found in line 26 at 2023-08-pooltogether/src/RngAuctionRelayerRemoteOwner.sol:

    ISingleMessageDispatcher public immutable messageDispatcher;


Found in line 30 at 2023-08-pooltogether/src/RngAuctionRelayerRemoteOwner.sol:

    RemoteOwner public immutable account;


Found in line 33 at 2023-08-pooltogether/src/RngAuctionRelayerRemoteOwner.sol:

    uint256 public immutable toChainId;


Found in line 42 at 2023-08-pooltogether/src/LiquidationRouter.sol:

  LiquidationPairFactory internal immutable _liquidationPairFactory;


Found in line 29 at 2023-08-pooltogether/src/LiquidationPair.sol:

  ILiquidationSource public immutable source;


Found in line 32 at 2023-08-pooltogether/src/LiquidationPair.sol:

  address public immutable tokenIn;


Found in line 35 at 2023-08-pooltogether/src/LiquidationPair.sol:

  address public immutable tokenOut;


Found in line 38 at 2023-08-pooltogether/src/LiquidationPair.sol:

  SD59x18 public immutable decayConstant;


Found in line 41 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable periodLength;


Found in line 45 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable periodOffset;


Found in line 48 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint32 public immutable targetFirstSaleTime;


Found in line 54 at 2023-08-pooltogether/src/LiquidationPair.sol:

  uint256 public immutable minimumAuctionAmount;


Found in line 18 at 2023-08-pooltogether/src/abstract/RngAuctionRelayer.sol:

    RngAuction public immutable rngAuction;

------------------------------------------------------------------------ 

### Mitigation 

Immutables should be in uppercase, it helps to distinguish immutables from other types of variables and provides better code readability.
