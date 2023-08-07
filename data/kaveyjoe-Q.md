Target : https://github.com/GenerationSoftware/pt-v5-cgda-liquidator/blob/7f95bcacd4a566c2becb98d55c1886cadbaa8897/src/LiquidationRouter.sol

 LiquidationRouter does not check whether LiquidationPairFactory is still active

## Description:
 The LiquidationRouter contract only checks whether the provided LiquidationPair instance was deployed by the specified LiquidationPairFactory. It does not check whether the factory is still active. This means that it is possible for a malicious actor to create a fake LiquidationPairFactory and then deploy a fake LiquidationPair that appears to be legitimate. If a user were to interact with this fake LiquidationPair, they could lose their funds.





##Impact: 
If the onlyTrustedLiquidationPair modifier is not updated, it is possible for malicious actors to create fake LiquidationPairFactory contracts and deploy fake LiquidationPairs that appear to be legitimate. This could lead to users losing their funds if they interact with these fake LiquidationPairs.

## Mitigation:
 The onlyTrustedLiquidationPair modifier should be updated to check whether the LiquidationPairFactory is still active. This could be done by checking whether the factory's address is still in the list of deployed factories.

## Recommendation: 
The onlyTrustedLiquidationPair modifier should be updated to check whether the LiquidationPairFactory is still active. This would help to mitigate the risk of users losing their funds to malicious actors.

 here is a suggested fix for the potential security issue in the LiquidationRouter contract:

modifier onlyTrustedLiquidationPair(LiquidationPair _liquidationPair) {
  if (address(_liquidationPairFactory) == address(0)) {
    revert UndefinedLiquidationPairFactory();
  }
  if (!_liquidationPairFactory.isDeployed()) {
    revert LiquidationPairFactoryNotActive();
  }
  if (!_liquidationPairFactory.deployedPairs(_liquidationPair)) {
    revert UnknownLiquidationPair(_liquidationPair);
  }
  _;
}

This fix adds a new check to the onlyTrustedLiquidationPair modifier to see if the LiquidationPairFactory is still active. This check is done by calling the isDeployed function on the LiquidationPairFactory contract. If the factory is not active, the modifier will revert with the LiquidationPairFactoryNotActive error.

This fix would help to mitigate the risk of users losing their funds to malicious actors by preventing them from interacting with fake LiquidationPairs that are deployed by inactive LiquidationPairFactory contracts.