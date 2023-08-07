PoolTogether is a no-loss prize savings protocol that allows users to deposit tokens and earn yield, with the yield being randomly awarded as prizes. Version 5 of PoolTogether brings massive improvements, including making the protocol autonomous and permissionless.

- The current audit scope includes several critical pieces:

                         - Liquidation Pair:
 This is the contract that liquidates yield for prize tokens.
Draw Auction: This suite of contracts auctions off RNG requests and then auctions off the bridge transaction to L2.
                        - Remote Owner:
 A contract that builds on ERC-5164 and allows a contract on one chain to control a contract on another.
                        - Vault Boost:
 A contract that allows anyone to liquidate any token and contribute to the prize pool on a vault's behalf.
Learnings

- In reviewing the PoolTogether V5: Part Deux codebase, I learned a number of things:

           1 . The PoolTogether team has done a great job of designing a modular and well-tested codebase.
            2 . The use of ERC-5164 to allow for remote ownership is a clever way to decentralize the protocol.
            3 . The Vault Boost contract is a powerful tool that allows anyone to contribute to the prize pool, regardless of the token they hold.

- Approach

I evaluated the codebase using a number of different approaches, including:

- Code review:
 I manually reviewed the code to identify any potential security vulnerabilities.

- Static analysis:
 I used a static analysis tool to identify any potential security vulnerabilities.

- Unit testing: I ran the unit tests to ensure that the code is working as expected.

- Integration testing:

 I ran integration tests to ensure that the code interacts with other contracts as expected.



- Architecture Feedback

The PoolTogether V5: Part Deux codebase is well-architected and easy to understand. The use of modules and interfaces makes the code easy to reuse and maintain.

- Centralization Risks

The only potential centralization risk I identified is the use of the Remote Owner contract. This contract allows a single entity to control multiple vaults, which could give that entity undue influence over the protocol.

- Systematic Risks

The PoolTogether protocol is designed to be as safe as possible, but there are still some potential systematic risks. For example, if the underlying yield sources become illiquid, the protocol could be unable to pay out prizes.

- Recommendations

I recommend that the PoolTogether team continue to monitor the codebase for any potential security vulnerabilities. I also recommend that the team consider adding additional safeguards to mitigate the centralization risk posed by the Remote Owner contract.

- Conclusion

The PoolTogether V5: Part Deux codebase is well-designed and secure. The use of modularity, testing, and static analysis has helped to ensure the quality of the code. The only potential centralization risk I identified is the use of the Remote Owner contract. However, this risk can be mitigated by adding additional safeguards. Overall, I believe that the PoolTogether V5: Part Deux codebase is a significant improvement over previous versions of the protocol.



### Time spent:
15 hours