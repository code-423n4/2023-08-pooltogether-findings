# [A-01] Any comments for the judge to contextualize your findings

My primary objective is to enhance the gas efficiency and cost-effectiveness of the protocol. Throughout my analysis, I have diligently pursued opportunities to optimize gas consumption and have prepared a comprehensive report comprising `23` detailed gas optimization recommendations tailored to the specific needs of the protocol.

It is my firm belief that the implementation of these optimization techniques can lead to substantial improvements in the protocol's performance, especially concerning gas fees. By carefully reviewing and implementing these suggestions, the protocol stands to benefit from reduced transaction costs and enhanced overall efficiency.

I am confident that my gas optimization expertise and the comprehensive report submitted will prove instrumental in streamlining the protocol's gas usage and making it more competitive in an environment of ever-changing gas fees.

# [A-02] Approach taken in evaluating the codebase

Accordingly, I analyzed and audited the subject in the following steps;

1. **Core Protocol Contracts Overview**: In this step, I will conduct a comprehensive evaluation of the core smart contracts that constitute the PoolTogether V5 protocol. The contracts within the scope of this audit encompass critical components such as LiquidationPair, Draw Auction, Remote Owner, and Vault Boost, among others.The PoolTogether V5 protocol introduces novel auction mechanisms, with specific contracts utilizing Continuous Gradual Dutch Auction (CGDA) and Parabolic Fractional Dutch Auction (PFDA) models. These models enable effective yield liquidation and RNG request auctions while allowing seamless interaction between L1 and L2 Prize Pools.

2. **Documentation Review**:

   - Liquidation Pair:The Liquidation Pair in PoolTogether V5 is a crucial contract responsible for executing Periodic Continuous Gradual Dutch Auctions (CGDA) to convert yield into prize tokens. It ensures that each Vault has its own Liquidation Pair, aligns with draw lengths for regular auctions, and aims for a 95.2% liquidation efficiency. Security concerns include the potential for permanent breakage or deliberate/accidental bricking.

   - Draw Auction: The Draw Auction is a pivotal component of the PoolTogether V5 protocol, specifically designed to ensure a seamless and permissionless process for conducting daily prize pool draws. In contrast to previous versions, the V5 protocol eliminates the need for permissioned controls, relying instead on a sophisticated incentivization mechanism through timed auctions, the Draw Auction empowers the PoolTogether V5 protocol with a robust and transparent process for conducting prize pool draws. By leveraging timed auctions and PFDA pricing, this incentivization approach fosters an ecosystem of self-sufficiency, enabling seamless daily operations across all deployed prize pools.

   - Remote Owner: The Remote Owner is a pivotal contract within the PoolTogether V5 protocol, extending its control from one chain to another through ERC-5164 integration. This unique contract empowers a contract deployed on one chain to govern and control a contract situated on a different chain. Its role is critical in achieving cross-chain functionality and interconnectivity within the protocol,By building upon ERC-5164, the Remote Owner facilitates seamless and secure communication between contracts on separate chains, allowing for efficient cross-chain operations. It enables actions initiated on one chain to have corresponding effects on a contract deployed on another, fostering a unified and decentralized ecosystem.

   - Vault Boost: Vault Boost is a fundamental contract within the PoolTogether V5 protocol, introducing a novel mechanism that empowers anyone to liquidate any token and make contributions to the prize pool on behalf of a vault. This feature enhances the protocol's flexibility and inclusivity, enabling participants to actively participate in the growth and success of the prize pool,With the power to facilitate token liquidation and direct contributions to the prize pool, Vault Boost significantly enhances the overall liquidity and utility of the protocol. By permitting users to take a proactive role in supporting the prize pool, it strengthens the protocol's sustainability and attractiveness.

3. **Graphical Analysis**: Solidity-metrics is a popular tool used to analyze and visualize Solidity code. It provides various metrics and visualizations to gain insights into the codebase's complexity and maintainability.
   [solidity-metrics](https://github.com/ConsenSys/solidity-metrics)

4. **Utilizing Static Analysis Tools**: During the static analysis phase, I have discovered specific vulnerabilities in the smart contract. However, I want to bring to your attention that these vulnerabilities are already known issues and are associated with `BotRace`.

5. **Test Coverage Evaluation**: I currently encounter challenges with the foundry and, regrettably, am not proficient in resolving these issues. As a result, I am unable to run the tests effectively.

6. **Manuel Code Review**

# [A-03] Architecture recommendations

1. **Decentralized RNG Source**: While Chainlink VRF V2 is a solid initial choice for the RNG source, consider exploring the integration of multiple decentralized RNG solutions. This approach can provide additional layers of security and decentralization, reducing the risk of a single point of failure and enhancing the overall robustness of the Draw Auction system.

2. **Dynamic Auction Duration**: Implement an adaptive auction duration based on the auction's reserve size and expected demand. Shorter auction durations during periods of high activity can increase auction efficiency, while longer durations during periods of low activity can give participants more time to bid accurately. This dynamic approach can optimize the auction process and adapt to varying market conditions.

3. **Incentives Mechanism**: The architecture should implement a rewards and incentives mechanism to motivate users to participate in liquidations actively. Users can earn governance tokens, yield farming rewards, or other benefits based on their liquidation contributions.

4. **Multi-Token Reward System**: Allow users to deposit different tokens into the RngAuction contract, each representing a separate share in the prize pool. This would provide more flexibility for users and allow the protocol to accept a broader range of assets.

# [A-04] Codebase analysis

Overview of the Codebase:

- LiquidationPair.sol: This contract facilitates Periodic Continuous Gradual Dutch Auctions (CGDA) for yield. It utilizes the prb-math library to implement the CGDA formulas.

- LiquidationPairFactory.sol: This contract is responsible for creating new LiquidationPairs.

- LiquidationRouter.sol: This contract acts as the user-facing interface for LiquidationPairs. Users interact with this contract to initiate liquidations.

- ContinuousGDA.sol: This library provides implementations of the Continuous Gradual Dutch Auction (CGDA) formulas using the prb-math library.

- RngAuction.sol: This contract auctions off an RNG (Random Number Generator) request. It uses the prb-math, openzeppelin, owner-manager-contracts, and pt-v5-rng-contracts.

- RngAuctionRelayerDirect.sol: This contract relays RNG auction results to a listener directly.

- RngAuctionRelayerRemoteOwner.sol: This contract relays RNG auction results over an ERC-5164 bridge to a listener. It uses ERC5164.

- RngRelayAuction.sol: This contract auctions off the RNG result relay. It uses ERC5164.

- RewardLib.sol: This library implements Parabolic Fractional Dutch Auction math using the prb-math library.

- IRngAuctionRelayListener.sol: This interface defines the listener for the RNG auction relay.

- IAuction.sol: This interface defines common auction functions.

- RngAuctionRelayer.sol: This is the base class for relayers.

- AddressRemapper.sol: This contract allows addresses to remap themselves for relayers.

- VaultBooster.sol: This contract allows anyone to liquidate tokens to boost the chances of a Vault winning.

- VaultBoosterFactory.sol: This contract creates new Vault Booster contracts.

- RemoteOwner.sol: This contract enables a contract on one chain to control a contract on another using ERC-5164.

- RemoteOwnerCallEncoder.sol: This library handles encoding remote calls for the RemoteOwner contract.

The codebase consists of several contracts and libraries that facilitate different functionalities related to auctions, RNG requests, liquidation, and boosting the chances of a Vault winning. Some contracts are responsible for relaying information between different chains, and others implement auction-related logic using mathematical formulas. The contracts utilize external libraries like prb-math and openzeppelin to enhance their functionality.

# [A-05] Centralization risks

Centralization of Vaults in the PoolTogether V5 protocol can indeed pose significant risks to the overall decentralization and security of the system. When a substantial number of Vaults are controlled by a few entities or individuals, it creates a concentration of power and control within the protocol.

Unfair Competition: If a few entities control a large number of Vaults, it may deter new participants from entering the system. This lack of competition can harm the protocol's overall health and growth, limiting the potential benefits of decentralization and open participation.

Price Manipulation: Centralized control of Vaults can lead to market manipulation. Entities with significant control over Vaults may coordinate their actions to influence token prices or the outcome of auctions, impacting the fair operation of the protocol.

# [A-06] Systemic risks

1. **Liquidity Risk**: Inadequate liquidity in the auction or liquidation processes could lead to inefficient price discovery, potentially causing large price fluctuations or failed liquidations.

2. **RNG Manipulation**: The randomness provided by the RNG service is critical for the fairness of the protocol. Any manipulation or compromise of the RNG service could result in predictable outcomes, impacting the integrity of the entire system.


### Time spent:
16 hours