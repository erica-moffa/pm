# Core Devs Community Call 52

### Meeting Date/Time: December 24th, 2025, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/179)
### Agenda

  - [Syncing the development progress of java-tron v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [TIP-7702: Add a new tx type that permanently sets the code for an EOA](https://github.com/tronprotocol/tips/issues/728)
  - [TIP-8004: Introduce ERC-8004 Agent Identity / Reputation / Validation Registry + X402 Proof-of-Execution Support for TRON](https://github.com/tronprotocol/tips/issues/807)
  - [Introduce TronWeb v6 features](https://github.com/tronprotocol/tronweb/releases)

### Detail

- **Murphy**
        
    Welcome, everyone, to the 52nd TRON Core Dev Meeting. We have four items on the agenda today. First, I’d like to invite Neo to introduce the development progress of java-tron v4.8.1.
    
**Syncing the development progress of v4.8.1**

- **Neo**
        
    Thanks. Regarding the progress of version 4.8.1, the Nile Testnet completed its full upgrade two weeks ago. The two associated proposals are currently active on the testnet and running smoothly. Based on the current schedule, we expect to start Mainnet testing as early as next week, which should take about 1 to 2 weeks. After that, we will officially enter the Mainnet release phase.

- **Murphy Zhang** 
    
    Alright, so based on that rhythm, the official release should be around early January, correct?
    
- **Neo Yue** 

    Yes, the official release is expected in early January. We estimate the entire network will complete the upgrade by early February.
    
- **Murphy**
        
    Understood. Next, we need to check the release readiness, including updating the Release Notes, updating TIP statuses, and confirming that all items in PM are set to the "Pending Release" state. (Neo: Will do.)
    
- **Murphy**
    
    Great. Are there any other questions regarding the 4.8.1 release progress? If not, let’s move to the second topic: Sunny will lead the discussion on TIP-7702.

**TIP-7702: Add a new tx type that permanently sets the code for an EOA**

- **Sunny** 

    I’d like to start by getting an update on the current progress of TIP-7702.
    
- **Aiden** 
    
    Right now, we are primarily evaluating the specific impacts TIP-7702 might have. There are no updates on substantive development progress for the time being.

- **Sunny**
    
    While researching TIP-7702, I found that one of its core use cases is for **Paymasters** or **Bundlers**. In Ethereum’s practice, handling untrusted `UserOperations` requires following the ERC-7562 framework for verification. This typically relies on the EVM’s Tracer functionality, specifically the `debug_traceCall` interface. Currently, TVM does not support this. Without `debug_traceCall`, the application scenarios for TIP-7702 might be limited to trusted scenarios, such as self-bundling by the account owner. I’d like to know your thoughts on supporting `debug_traceCall` and whether we should include it in the development plan. Additionally, supporting this feature would require MPT (Merkle Patricia Tree) support.

- **Aiden** 
    
    First, TIP-7702 and Bundler functionality are not technically mandatory for each other; they are relatively independent. Second, the changes required for `debug_traceCall` are very significant, and we currently lack the foundation for it. If we were to support it, the workload and difficulty might exceed that of implementing TIP-7702 itself. We need to discuss this internally in more detail.

- **Sunny** 
    
    Understood. If the priority of `debug_traceCall` is raised, we definitely need to resolve the state tree support issue first. 
    
    Also, I've updated TIP-7951 with a new example. Please verify the accuracy of the `Code Size` description in the `Response` field.

- **Murphy** 

    
    Got it. I’ve also shared feedback with Aiden that the community is very interested in TIP-7702. If implementing Paymasters indeed requires the coordination of `debug_traceCall`, please evaluate it to see if these features should be synchronized into TRON’s future network upgrade plans.

- **Aiden** 

    Okay, we will conduct an internal discussion.

- **Murphy** 

    Alright, any other questions on this topic? If not, let's move to the third topic. Hades, please introduce TIP-8804.

**TIP-8004: Introduce ERC-8004 Agent Identity / Reputation / Validation Registry + X402 Proof-of-Execution Support for TRON**

- **Hades** 
    
    This time, we are primarily researching and proposing the introduction of the TIP-8804 protocol on TRON. Simply put, 8804 addresses the identity, trust, and reputation of Agents in a decentralized network. 

    This includes: how to discover an Agent, confirm its capabilities, verify its work results, and how to establish and maintain a reputation system. Regarding the current state of 8804 on Ethereum, our research shows that the work of related vendors can be categorized into three directions: infrastructure, which focuses on Agent Launchpads and discovery or management platforms; agent development, where DeFi-related agents are currently the most active; and anti-Sybil mechanisms, which are aimed at trust evaluation to prevent malicious rating manipulation."
    
    We decided to push for 8804 on TRON based on a few core judgments. First, we believe in the future trend: deep interaction between Agents and smart contracts is undoubtedly the next major direction for blockchain. Second, the current industry status is that both TRON and Ethereum lack a unified identification standard, and we want to lead the way in filling this gap. Finally, in terms of ecosystem scale, thousands of Agents have already emerged on Ethereum, proving the market potential. We hope to leverage TRON’s advantages in TPS and energy consumption, combined with the strong liquidity of USDT, to provide these developers with a more seamless and efficient migration and management environment.
    
    Our goal is to establish a set of standardized registration and verification mechanisms that are fully EVM-compatible, allowing Ethereum developers to migrate seamlessly. The initial phase focuses on the protocol layer to build a foundation for advanced iterations.
        
    I will now introduce the overall design of TIP-8804, which is structured into three primary layers:

    - Application Layer: Includes interactions between users and Agent clients.
    - On-chain Portion: Contains three core contracts—Identity Registry, Reputation Management, and Work Validation.
    - Off-chain Portion: Primarily consists of the Aggregator, Agent services, and validation nodes. The Aggregator acts like an app store, providing efficient query and filtering services based on on-chain data; Agent services and validation typically run off-chain. Validation nodes verify the Agent's work using methods like staking, TEE (Trusted Execution Environment), or zkML (Zero-Knowledge Machine Learning). Simple verification can be done by on-chain contracts, while complex computations are handled by off-chain validators.

    A typical interaction flow would be:
    1. User queries the Aggregator for a list of Agents and selects a provider.
    2. User initiates a request, potentially combined with a payment layer (like ERC-4337).
    3. Agent receives the request, executes the task (e.g., AI computation), and generates a Proof of Execution.
    4. Agent submits the result and proof to the Validation Contract.
    5. The Validation Contract (or off-chain validator) verifies it and writes the result back on-chain.
    6. Agent returns the final result and payment proof to the user.
    7. User submits a review based on the payment proof; the review hash goes on-chain, while detailed content is stored on BTFS to reduce costs and ensure tamper-resistance.
    
    Regarding core contracts:
    - Identity Contract: Assigns a unique ID (similar to an NFT) to each Agent via an Agent Card (URI) describing its capabilities, endpoints, and pricing.
    - Reputation Contract: Manages ratings on-chain, with details on BTFS. Uses payment proofs to prevent fake reviews.
    - Validation Contract: Supports multiple methods. Phase 1 may support staking (Optimistic challenge mechanism), with future extensions for TEE and zkML.
    
    The preliminary research is finished, and we are currently designing the MVP. Development is expected to start soon.

- **Sunny** 

    Does this protocol require changes to the TRON protocol layer?

- **Hades** 
    
    Currently, the three core contracts and payment logic do not require protocol-level changes. However, when implementing advanced features, like complex authorization, Ethereum has proposals like EIP-3074. TRON’s native permission mechanism might not have direct support for delegating contract call permissions to an Agent. I wonder if the previously discussed TIP-7702 could solve this?

- **Sunny**
    
    TIP-7702 can indeed solve part of the authorization issue. Also, the original ERC-4337 (AA wallets) can handle some of it through off-chain authorization. We can further assess the necessity for underlying protocol changes based on the specific implementation details.

- **Hades** 
    
    Great. Our approach is to stay as compatible as possible with Ethereum’s EIP-7702 and ERC-4337. We will confirm later if TRON requires extra protocol changes.

- **Sunny** 

    How much traffic do you expect these contracts to bring after deployment? Any estimates on TPS pressure?

- **Hades** 

    If the ecosystem matures, the interaction volume between clients and Agents could be very high. But since Ethereum supports this volume, TRON’s higher throughput should comfortably handle the demand.

- **Robert** 

    Regarding the validation part, for example, submitting a ZK Proof, Ethereum has existing storage patterns. Will we be passing the entire `calldata` or just the Hash, considering we have contract size limits?
    
- **Hades** 
    
    It depends on the size of the proof. If it doesn't fit on-chain, we can store the Hash and put the raw data on BTFS. During validation, the validator pulls data from BTFS. For on-chain validation, the contract can be treated as a black-box function: input the proof and get the verification result.

- **Neo** 
    
    Is the validation computation running on-chain or on the Validator? Currently, running ZK computations on-chain isn't very efficient; we are working on optimizations.

- **Hades** 
    
    ZK computation is indeed heavy. For the Phase 1 MVP, we won't do TEE or complex ZK; we’ll focus on simple validation based on payment proofs. Complex validation logic is deferred to future iterations or handled via off-chain Validators.

- **Neo** 
    
    Alright. Does that mean Phase 1 needs to restrict Agents from submitting unsupported proof types? Or should we assess the required OpCode compatibility?

- **Hades** 
    
    Yes. Validators are registered as plugins. Since Phase 1 won't support TEE, if an Agent submits that type, it won't succeed.

- **Sunny** 
    
    To summarize, the public chain cares about two things: First, does it require protocol-layer changes? Second, what is the impact on network stability (TPS increase, execution time, storage growth)?

- **Hades** 
    
    On the first point, the MVP stage requires no protocol changes. On the second, TPS depends on market scale. If we’re worried about too many Agents causing pressure, we can add limits. But since core AI computation is off-chain and the on-chain part is mostly payment and simple review interactions, the pressure should be manageable.

- **Neo** 

    Will there be a strategy within the contract to limit the number of registered Agents?

- **Hades** 
    
    We can, if necessary, but if on-chain throughput exceeds TRON’s capacity, the off-chain AI compute demand would be astronomical, which is unlikely. The bottleneck should be off-chain.

- **Neo** 

    True. It mostly depends on the performance of the contract methods. The Agent’s task logic is off-chain and doesn't heavily impact the chain.

- **Hades** 

    Exactly. On-chain pressure comes from "Discovery", which is handled by the Aggregator, so low pressure, and "Reviews", which have a cost, so frequency won't be too high. On-chain pressure will scale linearly with the actual business volume of the Agents.
    
- **Robert** 
    
    What is the approximate data packet size for each interaction in the review system?

- **Hades** 
    
    The MVP interface design is simple. Not considering rich text, it’s just a 0-100 score, maybe a few dozen KB. It won’t be huge.

- **Neo** 
    
    And the query function is also primarily on the off-chain Aggregator, right?

- **Hades** 
    
    Right, performing complex queries and traversals on-chain is too inefficient. The off-chain Aggregator provides a much better experience.

- **Murphy** 
    
    I have two small questions. First, is this implemented on L1 for Ethereum? Is there an L2 implementation? Second, you mentioned thousands of Agents registered on Ethereum — is that on Mainnet?

- **Hades** 
    
    The protocol includes a Chain ID, so the information in the Registry can point to other chains; L2 should be supported. Regarding the second question, yes, it’s on Mainnet. Many vendors have deployed similar contracts themselves because Ethereum doesn't require underlying changes. You just deploy the contract.

- **Sunny** 
    
    Keep us posted if there’s new data, especially regarding traffic and performance estimates. (Hades: Will do.)

- **Murphy** 
    
    Any other questions? If not, we’ll wrap up the TIP-8804 discussion for now. Feel free to sync with us when there’s major progress. Finally, let’s move to the last topic: Cathy will introduce the TronWeb v6 upgrade and changes.

**Introduce TronWeb v6 features**

- **Cathy** 
    
    I’ll be discussing the upgrade from TronWeb v5 to v6. Although v6 has been running stably for over a year and three months, we’ve noticed many projects are still using v5 or even older versions. Since v5 is no longer maintained, I’ll cover the new features of v6 and how to complete the migration. I'll break it down by: why upgrade, new features, v5 vs. v6, migration notes, and feedback channels.
        
    First, why upgrade? v6 is a version designed for sustainable iteration, with many optimizations in security and interaction. Since v5 will no longer receive new features, we encourage projects to upgrade as soon as possible for security and stability. Key new features in v6 include: 1) Full TypeScript support; 2) As a long-term production-grade SDK, we’ve optimized boundary case handling to make it more stable; 3) Stricter security library audits to minimize third-party dependencies; 4) More standardized handling of on-chain transaction data compared to v5.
        
    Regarding packaging and builds, as a JS project, v6 now offers richer output formats. Previously, we only bundled one full TronWeb JS file. Now, to be compatible with modern frameworks and legacy projects, we simultaneously provide ESM, CommonJS, and UMD formats. This modular standard output allows for better Tree Shaking, effectively reducing bundle size for frontend projects and offering better compatibility with new tech frameworks.
        
    In comparison, v6 offers better compatibility for both new and old projects, clearer boundary handling, and lower maintenance costs than v5. v5 is end-of-life and is less secure for ABI contract calls than v6. v6 enforces strong validation for ABI formats, aligning with the JS VM, whereas v5 had some ambiguous logic. Additionally, v6 introduces automated `npm audit` checks, which allow us to fix security issues promptly.
        
    Finally, regarding migration, our website provides detailed migration notes and FAQs. If you run into issues, you can reach us via the [TronWeb website](tronweb.network), the official support email (support@tronweb.network), or by submitting an issue on [GitHub](github.com/tronprotocol/tronweb). You can also apply to join the TRON mailing list through that same email.

- **Murphy** 
        
    Great. If there are no other questions regarding TronWeb v6, we will conclude today’s meeting. Thank you all for attending.


### Attendance

* Aiden
* Patrick
* Boson
* Brown
* Cathy
* Hades
* Sunny
* Gorden
* Leem
* Daniel
* Mia
* Neo
* Parson
* Sam
* Star
* Tina
* Vivian
* Wayne
* Robert
* Jeremy
* Erica
* Murphy
