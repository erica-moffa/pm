# Core Devs Community Call 54

### Meeting Date/Time: January 28th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/183)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342) [↓](##1)
  - [TIP-6780: SELFDESTRUCT only in same transaction](https://github.com/tronprotocol/tips/issues/765) [↓](##2)
  - [Parallelize eth_newFilter event matching](https://github.com/tronprotocol/java-tron/issues/6510) [↓](##3)
  - [Support parameter passing via the input field for eth_call](https://github.com/tronprotocol/java-tron/issues/6517) [↓](##4)
  - [Optimize the node connection logic](https://github.com/tronprotocol/libp2p/issues/129) [↓](##5)


### Detail

- **Murphy**
            
    Alright, let's get started with our 54th TRON Core Devs meeting. We have 5 topics on the agenda today. First, I’ll hand it over to Zeus to sync us on the development progress of version 4.8.1.

<a id="#1"></a>**Syncing the development progress of v4.8.1**

- **Zeus**
        
    Over the past two weeks, we’ve integrated three new PRs, which have slightly pushed back our release schedule. The specific changes are as follows:
        
    The first PR focuses on [supporting custom JDK installation](https://github.com/tronprotocol/java-tron/pull/6528). It now features automatic environment detection to install either JDK 8 or JDK 17. Whether deploying via Git or direct installation, the process is now more flexible, which significantly improves the developer experience.
        
    The second PR involves [optimizing the Asset Issue executor](https://github.com/tronprotocol/java-tron/pull/6525). We’ve added validation logic for asset issuance, specifically by adding overflow checks when calculating start times and frozen periods to prevent data out-of-bounds. While this isn't a critical issue, we’ve included it to ensure the rigor of the underlying logic.
    
    The third PR addresses [fixing data loss in event synchronization](https://github.com/tronprotocol/java-tron/pull/6526). We discovered that previous concurrency logic could lead to data loss when syncing between V1 and V2 (and vice versa). This concurrency bug has now been fully resolved to ensure the total accuracy of data synchronization.
    
    Given the progress of these 3 PRs, we estimate that version 4.8.1 will officially launch next Tuesday or Wednesday. In addition to this preliminary plan, I’d also like to sync that Trident SDK 0.1.1 has been officially released this week.
    

- **Murphy**
    
    Great. Any questions regarding the progress of 4.8.1? If not, let’s move on to the next topic.

<a id="#2"></a>**TIP-6780: `SELFDESTRUCT` Only in Same Transaction**

- **Murphy**
            
    Next, Aiden, please sync the updates on TIP-6780. Since 4.8.1 is approaching its release and this `SELFDESTRUCT` opcode change is a major update, I want to re-align with everyone on the details, especially for those who have recently joined the discussion.

- **Aiden**
    
    To provide some background, we are moving toward deprecating the `SELFDESTRUCT` opcode and discouraging its use. This change modifies the original behavior where the opcode allowed a contract to self-terminate, transfer assets to a target, and wipe all contract data.
        
    The updated behavior now covers 2 scenarios: if the opcode is called within the same transaction, its behavior remains the same as before. However, if called in a separate transaction, it can no longer delete account data. Specifically, the account will not be deleted, the current contract call will stop immediately, and the contract's stored values, code, and the account itself will be preserved. That said, the asset transfer to the target address will remain unchanged.
        
    Additionally, if the destination address is a contract, assets will no longer be burned. The Energy cost will be adjusted from 0 to 5,000.
    
    The main impacts are: usage outside of the same transaction will be affected; the method of burning TRX via this opcode will become obsolete; and the use of `CREATE2` combined with self-destruct for contract upgrades will fail, as the account is no longer deleted, preventing a subsequent `CREATE2` at the same address.
    
    We analyzed the on-chain data: there are currently about 50 million contract accounts using this opcode, but they hold very few assets, so the impact is manageable. Analysis of transactions since 2025 found that most self-destruct transactions complete the creation and destruction within a single transaction. This usage is primarily to delete an account after creating it for a transfer. This change will roll out with version 4.8.1, and is currently performing well in the test environment.
    
- **Robert** 
    
    I have a question. I’ve seen contracts that create a "child" contract, fund it, and then trigger a self-destruct on that child within the same flow. Does this nested creation and destruction fall under the first or second scenario?

- **Aiden**
        
    As long as the creation and self-destruction happen within the same transaction, it falls under the same-transaction scenario and will behave as it did before, so it won't be affected.

- **Murphy**
    
    So it executes first, and then deletes the contract after execution, right? This part won't be affected. (Aiden: Correct.) Additionally, Aiden, please gather the data from the Nile testnet so we can sync up on those findings at the next meeting. (Aiden: Okay.)
        
    If there are no other questions, let's proceed to the third topic. Zeus will introduce the proposal for parallelizing the `eth_newFilter` endpoint.

<a id="#3"></a>**Parallelization of `eth_newFilter` Event Matching**

- **Zeus** 
    
    This issue aims to address the efficiency of event matching, specifically for event subscriptions. As a reminder, java-tron offers event subscription and consumption via `eth_newFilter` and `eth_getFilterChanges`, respectively. Once a user subscribes, our background service matches every generated event against the user's request, and the user retrieves results by manually polling the system, similar to a WebSocket approach. 
    
    The current bottleneck is that the matching process is executed serially in a single thread. When many users subscribe simultaneously, the latency becomes problematic. For example, if the block height reaches 10,000 but the matching has only reached 9,900, users won't receive events immediately. This lag means matching performance is falling behind block synchronization, causing significant delays for users. 
    
    This proposal aims to optimize the current serial execution to improve overall matching speed and ensure a smoother user experience without these delays.
        
    The current implementation logic is: after a FullNode processes a block, all events are written to a FIFO queue. A background thread continuously consumes this queue and calls the `handleLogFilter` interface to match against user-created subscriptions. Successful matches are placed in a cache for the user. If the user doesn't access it within 300 seconds (5 minutes), the event and subscription request expire and are cleared.
                
    Theoretically, with $m$ subscribers and $n$ events per block, the time complexity for a single match is $O(m \times n)$. On Mainnet, if a block has 300 events and we hit 200,000 subscribers, matching one block would take over 20 seconds, which significantly exceeds the 3-second block interval and causes processing delays for users.
    
    Our optimization plan involves two key changes: first, switching from single-threaded to multi-threaded matching to better utilize the CPU; and second, introducing a cap on the number of active filters to prevent timeouts. 
    
    To support this, we’ll add a new configuration, `node.jsonrpc.maxLogFilterNum`. Preliminary tests show that with 30,000 subscriptions, processing time drops below 2 seconds, which comfortably fits within the 3-second block window.
    
    To maintain balance during parallelization, we are using **Streams** for load balancing. By splitting the user list for matching, we can ensure that the event order for each individual user remains intact.
    
    We also considered using an inverted index for even faster matching. While feasible, the implementation is complex, and given how frequently subscriptions expire, the maintenance cost is quite high. Therefore, we’re still weighing whether that's worth the risk.

    Ultimately, we will expose the `node.jsonrpc.maxLogFilterNum` setting, allowing users to adjust it based on their hardware. If you provide a public API service, you can increase it; otherwise, it can be kept low. We will determine the optimal default value through further functional testing.

- **Lucas** 
    
    One question. You mentioned matching user-defined filters with block events: what does that look like in the implementation, and how exactly are we optimizing it?
    
- **Zeus**
    
    Essentially, it's about grouping the users for parallel processing. By utilizing parallel Streams, we can group users using the default logic or specify a custom group count. This doesn't require massive code changes; we are essentially switching the existing logic from serial to parallel execution.

- **Lucas**

    And what about the time complexity after optimization?
    
- **Zeus**
    
    Theoretically, the complexity remains $O(m \times n)$, but the processing latency is reduced. For instance, if serial processing took 10 seconds, a parallelism factor of 2 could bring it down to 5. We discussed inverted indexing as a follow-up, but since that's a riskier change, we’re starting with parallelization and will re-evaluate later.
    
- **Murphy**

    I have another question. Whether before or after optimization, if there's a long backlog, will the user's call hang or eventually timeout?
    
- **Zeus**

    This interface won't timeout. Since the call reads directly from memory, it returns immediately. The matching itself happens asynchronously in the background, so it doesn't block the user's API call thread.

- **Murphy**

    Understood. Any other questions on this? Since this is still in the discussion phase, it would likely land in a version after 4.8.1, correct?
        
- **Zeus**
    
    Yes, that's correct. Everyone is welcome to leave comments on the GitHub [Issue](https://github.com/tronprotocol/java-tron/issues/6510).
    
- **Murphy**
    
    Great. Let's move to the fourth topic. Tina, please introduce the support for passing parameters via the `input` field in `eth_call`.
    
<a id="#4"></a>**Support on Parameter Passing via the `input` Field for `eth_call`**

- **Tina**
    
    The community has recently requested this several times: in our JSON-RPC interfaces, such as `eth_call`, energy estimation, and transaction building, we currently use the `data` field, but the latest Ethereum standard has transitioned to using `input`.
    
    The change is primarily for semantic consistency. For example, when querying a transaction, the return uses `input`, but to initiate a call, you must use `data`. This forces developers to manually remap fields if they want to modify and resend a transaction, which increases development overhead.
        
    Since most other Ethereum clients have already made this switch, java-tron’s lack of support for `input` creates a standard mismatch and makes third-party tool integration inconvenient.
            
    Our plan is to have these interfaces support both `data` and `input` fields, with `input` taking priority if both are present. This is a minor change that will greatly improve developer friendliness and ease of integration.

- **Zeus**

    What’s the ETA for this change? Are there any other incompatibility gaps we're looking at?

- **Tina**
    
    We’re aiming for v4.8.2. We are also reviewing other gaps, such as supporting `blockHash` where we currently only take `blockNumber`. Another suggestion is adding a `Timestamp` to `getTransactionReceipt`, which would save developers from making two separate calls (checking the block first and then the transaction). Ethereum already supports this, and we’ll look into it for the next version.
    
- **Murphy**
    
    Got it. Any other questions on this topic? If not, we’ll move to the final topic.

<a id="#5"></a>**Node Connection Logic Optimization**

- **Lucas**
            
    Currently, our P2P logic attempts a connection every 3.6 seconds upon failure. However, the underlying TCP connection timeout is actually 60 seconds.
        
    If a connection is still "in-progress" due to network latency or a busy peer, our logic layer doesn't currently track that status and initiates a new request every 3.6 seconds. This results in up to 14 attempts in a single minute, which is redundant and only increases the burden on a peer that’s already under heavy load.

- **Zeus**

    Usually, if it can't connect, it should disconnect immediately. How did you determine the peer was under high load and couldn't process the requests?
    
- **Lucas**

    This scenario is actually quite rare; I recall seeing it only once in production, where we saw the peer processing a massive stack of duplicate requests simultaneously.
    
    Our solution is to leverage the existing caching mechanism. Once a connection fails, we won't try that peer again for one minute (or two minutes if it was an active connection).
            
    The update is that we’re now including the "in-progress" TCP handshake in this cache management. Before initiating a connection, we check the cache; if a connection is already pending, we won't start a new one.
        
    The distinction for active nodes is that if a proactive connection returns an immediate failure, we remove it from the cache to allow a quick retry after 3.6 seconds. Regular nodes, however, do not follow this logic. This approach reduces redundant connections during high-latency periods while ensuring rapid reconnection for core nodes.
        
- **Murphy**
        
    So the goal is essentially to throttle how aggressively we poll other nodes for connections, right?

- **Lucas**
    
    Exactly. Since the underlying timeout defaults to one minute, the logic layer might trigger a new attempt while a connection is still being established. By implementing this two-minute mechanism, we effectively prevent duplicate connection attempts.
    
- **Murphy**
    
    Perfect. If there are no more questions, please feel free to comment on [Issue #129](https://github.com/tronprotocol/libp2p/issues/129). That’s all for today's meeting. Thank you everyone for participating!

### Attendance

* Aiden
* Boson
* Elvis
* Federico
* Gordon
* Lucas
* Mia
* Tina
* Vivian
* Robert
* Zack
* Jeremy
* Murphy
* Zeus
* Erica

