# Analysis Report for Arcade.xyz

## Codebase quality
The overall quality of the codebase for PoolTogether can be classified as "*Good*".

**Strengths**
- Natspec was really helpful and detailed.
- It tries to achieve complete decentraliztion. It’s remarkable they can do this without any governance.

**Weaknesses**
- Majority of the tests were using mock addresses instead of actual contracts. This is not recommended.
- Almost zero documentation for 50% of the contracts in-scope.

## Mechanism Review

- PoolTogether is a decentralized finance protocol that combines savings and lottery, allowing users to deposit funds and have a chance to win rewards in daily prize draws, while maintaining the ability to withdraw their initial deposits at any time.
- Following are the major parts of Pooltogether system:
    - **Prize Pool (OOS):** The Prize Pool receives POOL tokens from Vaults, and releases the tokens as prizes in daily Draws.
    - **Prize Vaults (or Vaults) (OOS):** Users deposit tokens into Vaults in order to be eligible to win prizes. Vaults generate yield, liquidate the yield for POOL tokens, and contribute the POOL to the Prize Pool. The amount of contributed POOL determines the Vault's portion of the odds.
    - **TwabController (OOS)**: A system that keeps track of users token balances, their historic balances and their average balances of historic time periods.
    - **Prize Claimer (OOS)**: Instead of users claiming their prize, the system incentivizes bots to claim the user’s prizes for them.
    - ****Draw Auction:**** A system that leverages an incentivisation mechanism to encourage competition among third parties to complete the draws in a timely manner while maximizing cost efficiency.
    - **Vault Boosters**: Allows to boost a vault winning chance by providing more liquidity to be exchanged for Pool Tokens.
    - **Liquidation Router:** End-users (liquidators) use this contract to swap vault shares or ERC20 tokens (like USDC) for POOL Tokens from the Vault or Vault Boosters. More sophisticated liquidators can directly call the Liquidation Pair.
    - **Liquidation Pair**: The call of Liquidation Router goes through a liquidation pair. It calculates how much amount should be swapped for the provided tokens. Vaults only accept call from their assigned liquidation pair to liquidate the tokens/shares.
- **Deposit & Withdrawal Flow**
    ![https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/1_qscfy0.jpg](https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/1_qscfy0.jpg)
    
    - You have `1000 USDC`. You deposit that into the Pooltogether’s USDC Prize Vault .
    - This vault sends the USDC amount to a yield vault (e.g. AAVE) and in-turn recieves yield vault’s shares. Then the prize vault will mint shares to the user.
    - All of this happens in the same transaction.
    - The withdrawal process is vice-versa.
    - So, multiple users deposit their USDC into the prize vault which is then deposited into yield vault. This vault recieves yield generated from the Yield Vault. This yield is auctioned off for POOL tokens which are deposited in Prize Pool to increase the vault’s chances of winning.
    - ***Note**: Minting and burning of shares of all vaults happen through a singleton contract - `TWABController`. It is omitted for simplicity.*
- **Liquidation Flow**
    ![https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/2_abf87b.jpg](https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/2_abf87b.jpg)
    
    - When a liquidator initiates the liquidation process, they call `swapExactAmountOut` on the LiquidationRouter. The router then transfers the POOL tokens from the liquidator and to the Prize Pool on behalf of the vault. The router also checks that the liquidation pair beinng called is deployed by the liquidation factory.
    - It then calls the `swapExactAmountOut` on Liquidation Pair associated with the vault. In the LiquidationPair, there are two tokens involved:
        - `tokenIn`  : POOL token
        - `tokenOut` : Vault shares
        
        The liquidation pair then calls `liquidate` on the Vault. 
        
    - The vault then calls `contributePrizeTokens` on the PrizePool to register the vault’s contribution. Then, it mints the vault shares to the liquidator.
    - ***Note**: The liquidation pair uses Continuous Gradual Dutch Auction system to sell the vault shares or ERC20 tokens (in case of Vault Boosters).*
- **Vault Boosters**
    ![https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/3_gblrty.jpg](https://res.cloudinary.com/duqxx6dlj/image/upload/v1691410971/Pooltogether/3_gblrty.jpg)
    
    - A prize vault can increase its chances of winning by using a Vault Booster.
    - Here's how it works: When a vault is created, a corresponding Vault Booster is deployed and linked to the vault's address (set in the constructor). The owner of the Vault Booster sets up corresponding Liquidation Pair for all the supported tokens.
    - Liquidators can then use the `swapExactAmountOut` function on the LiquidationRouter, providing the address of the Vault Booster's liquidation pair.
    - In this case, instead of receiving vault shares, the liquidator will receive ERC20 tokens, such as USDC or WETH. Additionally, the router will transfer POOL tokens from the liquidator to the Prize Pool as a contribution for the vault.
    - Hence, with the help of vault boosters, a vault can improve its odds of winning by offering additional ERC20 tokens to liquidators in return for Pool Tokens.

## Centralization risks
- The protocol has made significant progress towards decentralization from V4 to V5, and the efforts to achieve this are commendable.
- Regarding the auctions used to obtain a random number from a third party for the draw, it is crucial to carefully assess this aspect. There might be a centralization risk if the third party wins the auction and attempts to manipulate the Prize Pool draws by providing a non-random number.

## Systemic Risks
- Chainlink VRF is critical for fair prize draws. Any issues or unavailability with Chainlink VRF could impact the integrity of the draws.
- Like any smart contract-based system, PoolTogether is exposed to potential coding bugs or vulnerabilities. Exploiting these issues could result in the loss of funds or manipulation of the protocol.
- The continuous gradual Dutch auction (CGDA) mechanism is sensitive to market dynamics and potential manipulation. Fluctuations in token prices and the CGDA can influence prize distributions and introduce economic uncertainties.

## Architecture Recommendations
- I recommend rewriting the tests in the codebase for this audit to use the actual contracts instead of mock addresses. This will offer greater confidence during system deployment.
- Unfortunately, due to time constraints, I was unable to do so and preparing the PoC took longer because the tests lacked actual contracts. Since most of the PoolTogether system (PrizePool, TwabController, etc.) is out-of-scope, it was challenging to create a comprehensive integration test.

## Approach
- During this audit, my main focus was on examining the Liquidation system and the Vault Boosters in the Pooltogether V5 protocol.
- **Day 1:** I spent time understanding the overall working of the Pooltogether V5 system and getting an overview of the codebase.
- **Day 2:** I conducted a detailed exploration of the Liquidation and Vault Booster mechanisms.
- **Day 3:** I identified potential attack vectors and edge cases in the Liquidation flow and Vault Boosters. I also created PoC to demonstrate the issue found.
- **Day 4:** I dedicated this day to preparing the final report and analysis, summarizing the findings and recommendations.

## Learnings
- PoolTogether is a unique protocol that was new to me during this audit. It introduced me to the concept of a prize savings account, where users can securely deposit their funds and have the opportunity to win rewards in return.
- To be honest, PoolTogether is a fascinating protocol that stands out due to its innovative approach in both daily prize draws and the underlying continuous gradual Dutch auction math and the RNG system. The combination of these features makes it an exciting and captivating platform for users and participants in the DeFi ecosystem.

### Time spent:
28 hours