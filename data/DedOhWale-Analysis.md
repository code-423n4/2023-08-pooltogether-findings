# PoolTogether Analysis Report

## Table of Contents

1. [LiquidationPair](#LiquidationPair)
2. [ContinuousGDA](#ContinuousGDA)
3. [RngAuction](#RngAuction)

# LiquidityPair

### 1. **Analysis of the codebase (What’s unique? What’s using existing patterns?)**:

   - **Unique Features**: Implementing a continuous function to handle decay in emission rates. This unique combination facilitates an automated auction mechanism with decay functions.
   - **Existing Patterns**: Usage of interfaces, immutable variables, and error handling are consistent with established Solidity practices. The contract is structured around the continuous generalized Dutch auction pattern.

### 2. **Architecture feedback**:

   - **Parameter Management**: The contract parameters are clearly defined in the constructor and include validations for different conditions such as decay constants, period lengths, and initial amounts. This enhances the clarity of the contract's initialization.
   - **Auction Handling**: The contract uses continuous generalized Dutch auctions to manage liquidation pairs. The calculations are based on the emission rate, initial price, and decay constant. This ensures a gradual reduction in the purchase price over time.
   - **Error Handling**: The contract includes specific error types to handle different error cases, such as zero amounts, exceeding swap limits, or misconfiguration. This contributes to better code readability and debugging.

### 3. **Centralization risks**:

   - **Lack of Governance**: The contract does not seem to include any governance mechanisms or ownership restrictions, potentially exposing it to centralization risks if not handled properly in other parts of the system.

### 4. **Systemic risks**:

   - **Dependency Risk**: The contract relies on external libraries and interfaces (`ILiquidationSource`, `SD59x18`, `ContinuousGDA`). Any changes or vulnerabilities in those dependencies could affect the behavior of this contract.
   - **Mathematical Complexity**: The contract uses mathematical operations involving fixed-point arithmetic. Errors or oversights in these calculations could lead to unexpected behavior or vulnerabilities.

### 5. **Other recommendations**:

   - **Events**: Emitting events for key state changes (e.g., auction updates, swaps) could aid off-chain monitoring and interactions with the contract.

### Conclusion

The `LiquidationPair` contract appears to be a well-structured implementation of a continuous generalized Dutch auction system for liquidations. Its unique blend of continuous price decay with emission rates adds to the functionality. Careful consideration of dependencies, mathematical operations, and potential centralization risks should be part of the overall review and testing strategy for this contract.

# State Machine Report for LiquidationPair Contract

## States

- `AuctionNotInitialized`
- `AuctionInitialized`
- `AuctionRunning`
- `AuctionUpdated`
- `SwapProcessed`

## Transitions

### AuctionNotInitialized -> AuctionInitialized
- **Function:** constructor
- **Attack vectors:**
    - Validate the correct initialization of constants like decayConstant, periodLength, etc.
    - Ensure proper restrictions on initial amounts (e.g., AmountInZero and AmountOutZero errors).
    - Check if decayConstant and periodLength are properly validated.

### AuctionInitialized -> AuctionRunning
- **Transition happens as part of the contract's execution when the auction is initialized.**
- **Attack vectors:**
    - Ensure that the emission rate and initial price are computed correctly.
    - Validate that the auction starts and runs as expected according to the defined parameters.

### AuctionRunning -> AuctionUpdated
- **Functions:** `_updateAuction`, `_checkUpdateAuction`
- **Attack vectors:**
    - Validate that auction updates happen correctly based on the time period and other conditions.
    - Ensure proper computation of values like emission rate and initial price during the update.

### AuctionRunning -> SwapProcessed
- **Functions:** `swapExactAmountOut`
- **Attack vectors:**
    - Ensure that swaps are executed correctly with proper checks (e.g., SwapExceedsMax, SwapExceedsAvailable).
    - Validate proper calculation of swap amount and updating of variables like `_lastAuctionTime`.

### SwapProcessed -> AuctionRunning
- **Transition happens automatically after a swap is processed.**
- **Attack vectors:**
    - Ensure that the system returns to a consistent state after processing a swap.

### AuctionUpdated -> AuctionRunning
- **Transition happens automatically after the auction is updated.**
- **Attack vectors:**
    - Ensure that the system returns to a consistent state after updating the auction.

## General Security Considerations
- **Access Control:** Ensure that functions like liquidation and updates are appropriately protected.
- **Integer Overflow/Underflow:** Verify that no arithmetic operation can overflow or underflow.
- **Time Dependence:** Be cautious of relying on block.timestamp, especially in functions that depend on time calculations like `_computePeriod`.
- **Precision and Rounding:** Ensure that fixed-point arithmetic is handled with care, especially when dealing with the custom SD59x18 type.
- **Contract Interaction:** Ensure that interactions with the ILiquidationSource are handled safely and that re-entrancy is not a concern.
- **Gas Limits:** Be aware of loops or operations that could consume a lot of gas, such as during calculations involving fixed-point arithmetic.
- **Initialization:** Ensure that the contract's constructor initializes all necessary variables properly.


# ContinuousGDA

# Analysis of the ContinuousGDA Library

### 1. **Analysis of the codebase (What’s unique? What’s using existing patterns?)**:

   - **Unique Features**:
     - **Decay Function Implementation**: The library implements continuous functions to calculate purchase price and amount based on parameters like decay constants and emission rates. This mathematical approach is distinct and complex.
   - **Existing Patterns**:
     - **Fixed-Point Arithmetic**: The library leverages the `SD59x18` data type from the prb-math library to perform fixed-point arithmetic, consistent with established practices for handling precision in Solidity.

### 2. **Architecture feedback**:

   - **Parameter Management**:
     - The functions take clearly defined parameters relating to the mathematical concepts of decay and emission. Though the library itself does not perform validations, the assumptions on input constraints are clear.
   - **Mathematical Handling**:
     - Complex mathematical operations such as exponential and logarithmic functions are handled using the prb-math library, indicating careful design and consideration of numerical stability.

### 3. **Centralization risks**:

   - **Dependency on External Libraries**:
     - Dependence on prb-math for core calculations places trust in an external library, which could introduce risk if the external library is flawed or compromised.

### 4. **Systemic risks**:

   - **Mathematical Complexity**:
     - The complexity of the mathematical functions and potential for precision errors may pose risks in unexpected inputs or edge cases.
   - **Gas Costs**:
     - Some of the operations, like `exp()` and `ln()`, can be computationally expensive, leading to potential concerns around gas efficiency.

### 5. **Other recommendations**:

   - **Testing and Validation**:
     - Thorough testing of the mathematical functions with varied inputs, including edge cases, would ensure robustness.


# Analysis of the ContinuousGDA Functions

Due to the mathematical nature of this library, this was done in LaTeX and hosted on a gist: [ContinuousGDA LaTeX](https://gist.github.com/DedOhwale/0d891b52f07c53d88b0441b52aba5cc7)

# RngAuction

## Any comments for the judge to contextualize your findings

This contract is an auction system that relies on a Random Number Generator (RNG) interface. There are checks and balances in place to ensure fair sequencing and timing. The contract is built upon widely-used libraries and standards, like OpenZeppelin's ERC20 and Ownable patterns, and includes several error handlers to provide clear failure messages.

## Approach taken in evaluating the codebase

The evaluation of the codebase includes an analysis of the overall architecture, contract design, usage of external libraries, error handling, and the implementation of specific features such as random number generation and auction timing.

## Architecture recommendations

- **Code Modularity**: The contract is well-structured with a clear separation of concerns, using libraries and interfaces.
- **Error Handling**: Custom error messages provide clear context for failure cases, enhancing readability and debugging.

## Mechanism review

- **Auction Timing**: The contract manages the timing of auctions, including sequence periods, offsets, and durations. It also ensures that auctions do not overlap or occur outside of the prescribed times.
- **Random Number Generation**: The contract interacts with an external RNG interface. Depending on the specific implementation of this interface, randomness could introduce risks or benefits.

## Systemic risks

- **Potential Manipulation**: If the RNGInterface or the external time source (block.timestamp) can be manipulated, it may compromise the integrity of the auction.
- **Access Controls**: The owner has specific privileges, including changing the RNG service. If the owner account is compromised, it could have severe implications for the contract's operation.

# State Machine for the RngAuction contract

## States

- `AuctionNotInitialized`: Before the contract is created.
- `AuctionInitialized`: After the contract is created with the proper parameters.
- `AuctionRunning`: During the sequence period when auctions are ongoing.
- `AuctionExpired`: When an auction has passed its expiration time.
- `AuctionUpdated`: When an RNG request has been completed and the auction result has been updated.

## Transitions

### AuctionNotInitialized -> AuctionInitialized
- **Function:** `constructor`
- **Attack vectors:**
    - Check proper initialization of `sequencePeriod`, `sequenceOffset`, `auctionDuration`, `auctionTargetTime`.
    - Validate that `auctionTargetTime` does not exceed `auctionDuration`.
    - Ensure that the RNG service address is non-zero.

### AuctionInitialized -> AuctionRunning
- **Transition happens as part of the contract's execution when the auction is initialized.**
- **Attack vectors:**
    - Ensure that the proper sequence ID is calculated based on current time and sequence period.
    - Validate that a new sequence can start based on the last auction and sequence ID.

### AuctionRunning -> AuctionExpired
- **Function:** `startRngRequest`
- **Attack vectors:**
    - Ensure that auction does not exceed its duration, and it can be expired properly.
    - Validate that RNG request is created properly with correct fees and allowances.

### AuctionRunning -> AuctionUpdated
- **Function:** `startRngRequest`
- **Attack vectors:**
    - Validate that RNG request is created properly and that the last auction is updated with correct values.

### AuctionUpdated -> AuctionRunning
- **Transition happens automatically after the auction is updated.**
- **Attack vectors:**
    - Ensure that the system returns to a consistent state after updating the auction.

## General Security Considerations
- **Access Control:** Ensure that only the owner can change the next RNG service.
- **Time Manipulation:** Ensure that timestamps are used in a manner that does not allow for manipulation by miners.
- **Randomness Integrity:** Ensure that the RNGInterface used is secure and provides true randomness.
- **Fee Handling:** Validate proper fee transfer and allowances in the `startRngRequest` function.
- **State Integrity:** Validate proper state transitions and checks in all functions.

## Additional General Considerations
- **Gas Limitations**: Ensuring that complex operations within functions do not exceed the block gas limit.
- **Mitigation**: Optimization and gas analysis.

- **Oracles**: Dependence on an RNG service (oracle) for randomness introduces centralization risk and the risk of manipulation from the RNG provider.
- **Mitigation**: Choose a well-established RNG provider with strong cryptographic guarantees.

- **Token Allowances**: The contract increases allowance for the RNG service; this may be manipulated by an attacker if the RNG service is compromised.
- **Mitigation**: Be cautious with allowances, monitor the RNG service's behavior.

### Time spent:
8 hours