# Analysis  -  PoolTogether V5
### Summary
| List |Head |Details|
|:--|:----------------|:------|
|a) |The approach I followed when reviewing the code | Stages in my code review and analysis |
|b) |Test analysis | Test scope of the project and quality of tests |
|c) |Architectural | Architecture feedback |
|d) |Documents  | What is the scope and quality of documentation for Users and Administrators? |
|e) |Centralization risks | How was the risk of centralization handled in the project, what could be alternatives? |
|f) |Systemic risks | Potential systemic risks in the project |
|g) |Security Approach of the Project | Audit approach of the Project |
|h) |Other Audit Reports and Automated Findings | What are the previous Audit reports and their analysis |
|i) |Gas Optimization | Gas usage approach of the project and alternative solutions to it |
|j) |New insights and learning from this audit | Things learned from the project |


## a) The approach I followed when reviewing the code

First, by examining the scope of the code, I determined my code review and analysis strategy.
https://github.com/code-423n4/2023-08-pooltogether#scope

Accordingly, I analyzed and audited the subject in the following steps;

| Number |Stage |Details|Information|
|:--|:----------------|:------|:------|
|1|Compile and Run Test|[Installation](https://github.com/code-423n4/2023-08-pooltogether#setup)|Test and installation structure is simple, cleanly designed|
|2|Architecture Review| [PoolTogether v5 Protocol](https://dev.pooltogether.com/protocol/next/design/) |Provides a basic architectural teaching for General Architecture|
|3|Graphical Analysis  |Graphical Analysis with [Solidity-metrics](https://github.com/ConsenSys/solidity-metrics)|A visual view has been made to dominate the general structure of the codes of the project.|
|4|Slither Analysis  | [Slither](https://github.com/crytic/slither)| |
|5|Test Suits|[Tests](https://github.com/code-423n4/2023-08-pooltogether#tests)|In this section, the scope and content of the tests of the project are analyzed.|
|6|Manuel Code Review|[Scope](https://github.com/code-423n4/2023-08-pooltogether#scope)||
|7|Infographic|[Figma](https://www.figma.com/)|I made Visual drawings to understand the hard-to-understand mechanisms|
|8|Special focus on Areas of  Concern|[Areas of Concern](https://github.com/code-423n4/2023-08-pooltogether#additional-context)||



## b) Test analysis

- Test coverage is based on weighted unit tests, I recommend increasing integration tests
- More of the tests; It should also be done on attack vectors, for example, if there is a reentrancy modifier, the re-entrancy test should be done.
- In order for the tests to be more effective, the contracts to be tested will be deployed to the testnet and tested there, closer to the scope.


## c) Architectural 

## PoolTogether Projesi Abstract: 

<img width="1686" alt="image" src="https://github.com/code-423n4/2023-08-pooltogether/assets/104318932/01b79907-f867-4e9f-949f-f935d2cbfbe4">


<img width="677" alt="image" src="https://github.com/code-423n4/2023-08-pooltogether/assets/104318932/1845dd80-7d11-45b3-8ef7-63fe9e44fd26">


Mathematical Modeler used in the project;
It is Continuous Gradual Dutch Auction and Parabolic Fractional Dutch Auction, it is important to understand the concepts in the tree below in order to understand these concepts;
<img width="362" alt="image" src="https://github.com/code-423n4/2023-08-pooltogether/assets/104318932/b448e77b-9080-419a-9030-67ae61ce163d">


## d) Documents 
- Documentation should be increased further, it is recommended to add the architectural design to the documents as infographic

- I would also recommend adding quality Medium articles, it's a great way to provide an in-depth look at many of the topics in the project and is used by many blockchain projects.

##  e) Centralization risks 
There is a risk of centrality in the project as the onlyOwner
The owner role has a single point of failure and onlyOwner can use critical functions, posing a centralization issue. There is always a chance for owner keys to be stolen, and in such a case, the attacker can cause serious damage to the project due to important functions.

The code detail of this topic is specified in automatic finding;
https://github.com/code-423n4/2023-08-pooltogether/blob/main/bot-report.md#medium-2-privileged-functions-can-create-points-of-failure


Reducing the onlyOwner centrality risk in blockchain projects is essential to enhance security, decentralization, and community trust. Relying too heavily on a single owner or administrator can lead to various issues, such as single points of failure, potential abuse of power, and lack of transparency. Here are some strategies to mitigate the onlyOwner centrality risk:

1-Multi-Signature (Multi-Sig) Wallets: Instead of having a single owner with full control, you can implement multi-signature wallets. Multi-sig wallets require multiple parties to sign off on transactions or changes to the smart contract, making it more decentralized and secure.

2-Timelocks and Governance Delays: Introduce timelocks or governance delays for critical operations or updates. This means that proposed changes need to wait for a specified period before being executed. During this period, the community can review and potentially veto the changes if they identify any issues.

3-Decentralized Governance Mechanisms: Implement decentralized governance mechanisms, such as DAOs (Decentralized Autonomous Organizations) or community voting systems. This allows token holders or stakeholders to have a say in important decisions, making the project more community-driven.

Great summary of GalloDaSballo with examples of centrality risk and how to reduce it in previous Code4rena audits;
https://gist.github.com/GalloDaSballo/881e7a45ac14481519fb88f34fdb8837

##  f) Systemic risks 
The project uses the EIP-5164 architecture, a cross-chain execution interface for EVM-based blockchains, and the risk of cross-chain transactions is the biggest systematic risk. Therefore, more focus should be placed on this part.


We can talk about a systemic risk here because there are some Layer2 special cases.
Some conventions in the project are set to Pragma ^0.8.19, allowing the conventions to be compiled with any 0.8.x compiler. The problem with this is that Arbitrum is [Compatible](https://developer.arbitrum.io/solidity-support) with 0.8.20 and newer. Contracts compiled with these versions will result in a non-functional or potentially damaged version that does not behave as expected. The default behavior of the compiler will be to use the latest version which means it will compile with version 0.8.20 which will produce broken code by default.





## g) Security Approach of the Project

Successful current security understanding of the project;
1- The first audit was done in a good bug bounty program like Code4rena and all security concerns were resolved

https://github.com/code-423n4/2023-07-pooltogether

2- It is better that the project will be made in Code4rena in v5 upgrade, as it is a continuation of the previous audit.



What the project should add in the understanding of Security;
1- By distributing the project to testnets, ensuring that the audits are carried out in onchain audit. (This will increase coverage)
2- After the project is published on the mainnet, there should be emergency action plans (not found in the documents)
3- After the inspection, the concept of "continuous inspection" should be adhered to with bug bounty programs such as ImmuneFi.


## h) Other Audit Reports and Automated Findings 

**Automated Findings:**
https://github.com/code-423n4/2023-08-pooltogether/blob/main/bot-report.md

Especially  Medium detections in the Automated Finding Report should be taken into account;


| Number | Details | Instances |
|----------|----------|----------|
| [MEDIUM-1] | SafeTransfer should be used in place of transfer | 2 |
| [MEDIUM-2] | Privileged functions can create points of failure | 6 |
| [MEDIUM-3] | Contracts are vulnerable to fee-on-transfer accounting-related issues | 2 |
| [MEDIUM-4] | Gas grief possible on unsafe external calls  | 1 |
| [MEDIUM-5] | Return values of transfer()/transferFrom() not checked | 4 |
| [MEDIUM-6] | Some tokens may revert when zero value transfers are made | 1 |
| [MEDIUM-7] | Remaining eth may not be refunded to users | 1 |
| [MEDIUM-8] | Function which returns data from a call has no validation | 1 |
| [MEDIUM-9] | Polygon chain reorgs affect Chainlink randomness  | 1 |
| [MEDIUM-10] | Divide before multiply | 7 |



## i) Gas Optimization

The project is generally efficient in terms of gas optimizations, many generally accepted gas optimizations have been implemented, gas optimizations with minor effects are already mentioned in automatic finding, but gas optimizations will not be a priority considering code readability and code base size

Highest gas optimization: It is Storage Packed, high gas gain will be achieved in case of packaging in the following structs



https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L59-L61
```javascript
58:   function rewards(
59:     AuctionResult[] memory _auctionResults, // <= FOUND
60:     uint256 _reserve
61:   ) internal pure returns (uint256[] memory)  // <= FOUND

```

https://github.com/GenerationSoftware/pt-v5-draw-auction/tree/f1c6d14a1772d6609de1870f8713fb79977d51c1/src/libraries/RewardLib.sol#L80-L80
```javascript
79:   function reward(
80:     AuctionResult memory _auctionResult, // <= FOUND
81:     uint256 _reserve
82:   ) internal pure returns (uint256) 

```

The variables mentioned above can be packaged in several ways; The number of storage slots can be reduced by changing the order and decreasing the uint value.


## j) New insights and learning from this audit 

- I learned the ERC-5164 Specification and its use in codes
- I understood the architecture of Continuous Gradual Dutch Auction and Parabolic Fractional Dutch Auction and its use in code
- I learned Twab Controller mechanics
 


### Time spent:
14 hours