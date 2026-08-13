# Core Devs Community Call 68

### Meeting Date/Time: August 12th, 2026, 6:00-7:00 UTC

### Meeting Duration: 60 Mins

### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/226)

### Agenda

- Sync the Upgrade Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum [[Issue](https://github.com/tronprotocol/tips/issues/916)] [[↓](#topic2)]
- TokenPocket Adapter for EVM Introduced in TronWalletAdapter v1.3.2 [[Release](https://github.com/tronweb3/tronwallet-adapter/releases/tag/v1.3.2)] [[↓](#topic3)]
- End-to-End Smart Contract Development Workflow Introduced in TronIDE v2.3.2 [[Release](https://github.com/tronweb3/TronIDE/releases/tag/v2.3.2)] [[↓](#topic4)]
- Updates on TIP-899: Post-Quantum Signature Support [[Issue](https://github.com/tronprotocol/tips/issues/899)] [[↓](#topic5)]

### Detail

- **Murphy**

    Welcome to the 68th TRON Core Devs Meeting. We have 5 topics on today's agenda. Let's start with an update on the v4.8.2 upgrade progress.

<span id="topic1"></span>
**Sync the Upgrade Progress of v4.8.2**

- **Murphy**

    So far, 21 SRs have completed the v4.8.2 upgrade, with 6 still remaining. Among the projects being tracked, about 90% have completed the upgrade. Overall, the upgrade is progressing normally.

    Any questions on the v4.8.2 upgrade status? If not, let's move on to the activation plan for the Prague and Osaka TVM compatibility upgrades. David, please walk us through the latest progress.

<span id="topic2"></span>
**Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum**

- **David**

    Discussion on the proposal is progressing smoothly, and the feedback so far is broadly in favor of activating the related parameters. Based on the current progress, it should be ready to move forward in mid-to-late August.

    Any thoughts on the timeline or the proposal itself?

- **Patrick**

    The previous deadline for the v4.8.2 upgrade was August 15th, and [GreatVoyage-v4.8.2.1 (Heraclitus)](https://github.com/tronprotocol/java-tron/releases/tag/GreatVoyage-v4.8.2.1) was released afterward. Based on the current upgrade progress, August 25th can be used as the tentative target for initiating the voting request.

    Before that, the change still needs to be validated on Nile Testnet. If the voting request is targeted for around August 25th, the related parameters could first be activated on Nile Testnet during the week of August 18th for validation. How does that timeline look?

- **David**

    Works for me.

- **Daniel**

    The community discussion so far is pretty consistent and supports activating the proposal. I think the plan makes sense.

- **Vivian**

    Agreed.

- **Patrick**

    If there are no other risks, this can continue in the same direction, with the latest progress reflected in the proposal.

- **Murphy**

    Sounds good. Discussion can continue in the issue. Next, Cathy will walk us through the updates in TronWalletAdapter v1.3.2.

<span id="topic3"></span>
**TokenPocket Adapter for EVM Introduced in TronWalletAdapter v1.3.2**

- **Cathy**

    TronWalletAdapter v1.3.2 mainly includes four areas of updates: adding the TokenPocket EVM Adapter, improving EVM provider detection, improving React/Vue Hooks, and optimizing the React/Vue UI.

    The main addition in this release is `TokenPocketEvmAdapter`. It supports TokenPocket on EVM-compatible chains across iOS, Android, and browser extensions. With the unified EVM Adapter interface, the same integration approach can be used across desktop browsers and mobile devices.

    The base capabilities include wallet connection, message signing through `personal_sign`, EIP-712 typed data signing through `eth_signTypedData_v4`, transaction submission through `sendTransaction`, and network switching through `switchChain`. Mobile also supports deep links. If no TokenPocket provider is detected, the wallet website can be opened based on the configured fallback.

    There is one known limitation: the TokenPocket iOS app currently cannot sign contract deployment transactions correctly because it requires the transaction to include a `to` field, while adding that field causes the transaction to be treated as a contract call. Contract deployment on Android and browser extensions is not affected.

    The second area is EVM provider detection, mainly around EIP-6963. Wallets that respond synchronously to `eip6963:requestProvider` no longer leave a polling timer running or incorrectly report that no provider was detected. Failed detection results are no longer cached, so a wallet injected or enabled later can still be discovered. Retry detection also skips the 3-second grace period, allowing a faster result when the wallet is not installed.

    The third area is React/Vue Hooks. `connected` and address are re-read from the adapter after disconnects or account changes, preventing the adapter state from drifting out of sync with the frontend Hooks. Vue Hooks also include improvements such as listener cleanup.

    The fourth area is React/Vue UI. A closed wallet-select modal is removed from the tab order and accessibility tree so invisible elements no longer participate in keyboard navigation. There are also fixes around modal state and interaction behavior.

    Overall, v1.3.2 expands TokenPocket EVM support and further improves EIP-6963 provider detection and frontend state synchronization.

    Any questions?

- **Patrick**

    There are also applications in the TRON ecosystem that use EVM tooling, such as USDD. Is it currently using this adapter?

- **Cathy**

    I can't confirm the current integration yet. We'd need to check which implementation is being used.

- **Murphy**

    Thanks, Cathy. Next, Gray will introduce the new features in TronIDE v2.3.2.

<span id="topic4"></span>
**End-to-End Smart Contract Development Workflow Introduced in TronIDE v2.3.2**

- **Gray**

    TronIDE v2.3.2 mainly improves the workspace, Git integration, AI development flow, compilation and verification, debugging, and deployment workflows.

    The home workspace now provides shortcuts for creating contracts, TRON workspace templates, workspace search, and wallet connection, along with recently added plugins. Release Notes now have a dedicated entry in this version. The current view is text-based, with richer presentation and additional links planned for later versions. Workspace creation also includes basic templates such as TRC-20 and TRC-721, with more to be added later.

    On Git, v2.3.2 now supports a more complete local workflow, including repository initialization, stage / unstage, commit, history, and branch creation and switching. GitHub integration uses OAuth, supports direct repository `clone`, and also supports `fetch`, `pull`, `push`, and `force-push` with explicit confirmation.

    The AI Assistant is also moving from a relatively standalone feature into the broader development workflow. It can now use tools for compilation and compiler version selection, testing, static analysis, contract deployment and interaction, workspace file search / read / edit, local Git, verification preparation, Deployment Recorder, and TronBox export.

    For write operations such as file edits, the changes are shown first and only executed after confirmation. Error handling is still being improved. The AI API key stays in browser memory, is only sent to the selected provider endpoint, and is not written to persistent browser storage.

    The compiler flow improves version loading and error handling, and provides a bundled Solidity compiler fallback when the selected compiler cannot be downloaded. The Contract Verification workflow has also been improved: it can select the actual deployable main contract, flatten Solidity dependencies, and preview and export source files for TRONSCAN verification.

    The editor and file area now include formatting and context menu improvements, along with Solidity Static Analysis and Solidity UML. Static Analysis shows findings directly, while Solidity UML generates contract relationship diagrams.

    Deployment Recorder tracks each step in the deployment flow, including deployed contract addresses and execution status, and can export the recorded deployment flow as a runnable TronBox project.

    The Debugger now supports TVM instruction stepping, along with locals, state, and call stack information. Environment diagnostics have also been improved: transactions retain their originating environment, and trying to inspect a transaction from a different environment now returns a clearer message instead of a generic error.

    Other improvements include workspace recovery, Git status detection, static analysis display, security boundaries, and dependency handling. Overall, TronIDE is continuing to fill out the end-to-end workflow from contract creation, editing, compilation, deployment, and debugging through verification and collaboration.

- **Patrick**

    Release Notes can now be viewed inside the IDE, but there isn't a URL that lands directly on the corresponding Release Notes page yet, right? A stable URL would make external references easier.

- **Gray**

    Right now the main entry is inside the IDE. This can be improved in v2.3.3.

- **Murphy**

    GitHub already has the corresponding Release page. It may also make sense to link that directly from the IDE.

- **Gray**

    Sure, that link can be added later.

- **Murphy**

    So the two more visible changes in this release are the more complete Git / GitHub workflow and the expanded AI-assisted development features, right?

- **Gray**

    Right. Those are the two main areas. The rest are mostly enhancements to existing functionality.

    AI integration with development tools will continue to be strengthened. v2.3.3 also plans to make Bank of AI the default option.

- **Murphy**

    OK. If there are no other questions on TronIDE v2.3.2, Federico will take us through the latest progress on TIP-899 and post-quantum signature support.

<span id="topic5"></span>
**Updates on TIP-899: Post-Quantum Signature Support**

- **Federico**

    First, a quick update on the Testnet. Post-quantum signatures have been running on Nile Testnet for over a month, and the network has been operating normally with no obvious issues observed. The Testnet is seeing roughly 20 to 30 post-quantum signature transactions per day, with more than 700 transactions in total over the past month.

    Discussion on [TIP-899](https://github.com/tronprotocol/tips/issues/899) is also ongoing, with attention around wallets and related development tools. Community contributors have developed a TypeScript client for constructing and signing post-quantum transactions on Nile Testnet.

    The main topic today is public key storage.

    In the current Phase 1 implementation, each post-quantum transaction carries both the signature and the full public key. The FN-DSA-512 public key is 896 B, while the ML-DSA-44 public key is 1312 B. Because post-quantum signatures and public keys are both large, the main performance bottleneck comes from transaction size rather than signature verification CPU. For FN-DSA-512, the current estimated TPS ceiling is around 400.

    One optimization direction is to move the public key out of every transaction and store it once in a separate public key database, while keeping the existing `Account`, `Permission`, weight, and threshold model unchanged. ECDSA and post-quantum signatures can still go through the same `Permission` weight checking flow.

    A full post-quantum public key is not a good fit for direct storage in `Account`. The key itself is large, and `Account` is read and written frequently. Carrying that data through every serialization would add unnecessary storage and I/O overhead. A separate `pq_pubkey` database is a better fit.

    The current per-transaction public key design is simple, requires relatively small protocol changes, and does not require advance public key registration. On LiteNode, transaction history can be pruned, so the impact is relatively limited. On FullNode, however, the same public key is written repeatedly into historical transaction data, which leads to much larger long-term storage growth.

    The current Phase 2 draft proposes that the first transaction using a post-quantum public key still carries the full key. After successful verification, the node writes `address → public_key` into `pq_pubkey`. Later transactions only need to carry the corresponding 21-byte address, and the node loads the full public key from the database before verification.

    On top of this, there are currently two candidate approaches for referencing the public key.

    The first is to use the address derived from the public key directly as a `key reference`. This does not depend on the ordering of keys in `Permission`, so the reference is relatively stable.

    The second is to use a `permission key index`. The node first uses the index to find the address in the corresponding `Permission`, then uses that address to look up the full public key. This can reduce the reference encoding further, but it is affected by changes in key ordering inside `Permission`.

    Both approaches require public key registration. In addition to implicitly registering the key through the first real transaction, another option is to add a dedicated public key registration transaction type so the key can be written into the public key database in advance.

    If the account's `Permission` is updated after a transaction has already been signed but before it is submitted, the earlier transaction may become invalid. Both designs need to account for this. The direct-address reference is not affected by key ordering, so the current discussion leans toward that approach.

- **Patrick**

    What other optimization areas are planned besides public key storage?

- **Federico**

    There's also wallet-side keystore design, including how post-quantum keys are derived and stored. Other ecosystems' post-quantum work will continue to be tracked as well, including recent developments around Ethereum.

- **Patrick**

    So the main goal of the current public key storage optimization is to reduce transaction size and improve TPS, right?

- **Federico**

    Right. Signature verification itself is not the main bottleneck. The bigger issue is the transaction size introduced by the public key and signature.

- **Leem**

    If the post-quantum public key were stored directly in `Account`, is the main concern security?

- **Federico**

    No, the main concern is performance and storage overhead. The public key is large, and `Account` is read, written, and serialized frequently. Adding large public key data directly would increase the cost of those operations, so separate storage is a better fit.

- **Murphy**

    Got it. Then why not keep the current design where every transaction carries the full public key?

- **Federico**

    The main issue is FullNode storage growth. Today the same public key is saved again with every transaction. With a separate state database, the public key only needs to be stored once, and later transactions can reference it.

    For LiteNode, historical transaction data can be pruned, so the impact of the current design is relatively limited. FullNode keeps the full transaction history, so repeated public keys create a much larger database increase over time.

- **Murphy**

    The separate database here is still part of the node's own state database, right? Not external decentralized storage like IPFS?

- **Federico**

    Right. It's node state storage, not external storage.

- **Cathy**

    The current post-quantum signature support covers both FN-DSA-512 and ML-DSA-44, right?

- **Federico**

    Right.

- **Cathy**

    What's the current standardization status of the two algorithms?

- **Federico**

    ML-DSA-44 has been finalized as FIPS 204. FIPS 206 for FN-DSA-512 is still in draft and has not been finalized yet.

- **Cathy**

    Can SDKs and wallets already integrate against both algorithms on Nile Testnet?

- **Federico**

    Yes. Nile Testnet already supports the related post-quantum signature capabilities, and SDKs can support them as well. There is already a community TypeScript client that can construct the related transactions.

- **Cathy**

    The client signs locally, right?

- **Federico**

    Right. Signing is done locally.

    Any other comments on public key storage or the broader design can continue in the TIP-899 issue.

- **Murphy**

    OK. If there are no other questions, that's it for today's meeting. Thanks, everyone. See you next time.

### Attendance

- Blade
- Patrick
- Cathy
- Wayne
- Federico
- Gray
- Sunny
- Steven
- Gordon
- Leem
- Daniel L.
- Sam
- Tina
- Vivian
- Jeremy
- David
- Zeus
- Murphy
- Erica
