# Core Devs Community Call 67

### Meeting Date/Time: July 29th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/222)

### Agenda

- Sync the Development Progress of v4.8.2 [[Issue](https://github.com/tronprotocol/pm/issues/192)] [[↓](#topic1)]
- Introduce TRON Solidity Compiler v0.8.28 [[Release](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.28)] [[↓](#topic2)]
- Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum [[Issue](https://github.com/tronprotocol/tips/issues/916)] [[↓](#topic3)]
- Introduce `wallet-cli` v4.9.7–v4.10.0 [[Releases](https://github.com/tronprotocol/wallet-cli/releases)] [[↓](#topic4)]

### Detail

- **Murphy**

    Welcome to the 67th TRON Core Devs Meeting. We have four topics on today's agenda. First, Boson will give an update on the v4.8.2 upgrade.

<span id="topic1"></span>
**Sync the Development Progress of v4.8.2**

- **Boson**

    `java-tron` [GreatVoyage-v4.8.2 (Pyrrho)](https://github.com/tronprotocol/java-tron/releases/tag/GreatVoyage-v4.8.2) was released on July 15th. Tronscan currently shows that 19 Super Representatives (SRs) have upgraded to v4.8.2, while 8 have not. So far, no major issues have been reported.

- **Murphy**

    Other projects and service operators across the ecosystem are also moving to v4.8.2. Since broader adoption only began about a week ago, the overall status can be reviewed again at the next Core Devs Meeting.

    The network parameter proposals related to v4.8.2 are now under discussion. Is there a proposed order for advancing the TVM-related parameters and the calculation-hardening parameters?

- **Boson**

    There are two groups of proposals to consider. One covers the Prague and Osaka parameters for the TVM. The other covers the calculation-hardening parameters associated with [TIP-833: Harden ResourceProcessor Resource Window Calculations](https://github.com/tronprotocol/tips/issues/833) and [TIP-836: Harden Exchange Transaction Calculations](https://github.com/tronprotocol/tips/issues/836). The activation timing and order remain open for discussion in the relevant issues.

- **Murphy**

    Got it. Any other questions on the v4.8.2 upgrade? If not, next up is David, who will introduce the TRON Solidity Compiler v0.8.28 and the Prague and Osaka activation proposal.

<span id="topic2"></span>
**Introduce TRON Solidity Compiler v0.8.28**

- **David**

    I'll start with [TRON Solidity Compiler v0.8.28](https://github.com/tronprotocol/solidity/releases/tag/tv_0.8.28), tagged `tv_0.8.28`.

    While remaining compatible with Ethereum Solidity v0.8.28, this version further improves support for TRON-native capabilities and fixes related issues. The changes cover TRX and TRC-10 asset operations, resource delegation, voting, and on-chain resource queries.

    The main upstream change is full Solidity-language support for value-type transient state variables. State variables can now be declared as `transient`. A typical use case in the Solidity documentation is a reentrancy lock, which sets a lock during a call and clears it before the call returns.

    When code using this feature is compiled with v0.8.28, the resulting bytecode contains the `TSTORE` and `TLOAD` opcodes. Transient storage reads and writes are cheaper than their persistent-storage equivalents, so using it for this kind of temporary state can reduce execution costs. The release also includes several smaller fixes and improvements. Feel free to try this version and explore how transient storage works in practice.

<span id="topic3"></span>
**Proposal: Enable the Prague and Osaka Upgrades to Enhance Compatibility with Ethereum**

- **David**

    The second topic is the Prague and Osaka network parameter proposal mentioned earlier. The current draft proposes enabling `ALLOW_TVM_PRAGUE` and `ALLOW_TVM_OSAKA` together. Their network parameter IDs are 95 and 96, respectively.

    These two parameters enable support for the Ethereum Prague and Osaka upgrades. The details are listed in [issue #916](https://github.com/tronprotocol/tips/issues/916). The proposal creation and activation dates are still TBD. The timing may be clarified further at the next meeting based on SR upgrade progress and the ongoing public discussion.

    From an implementation perspective, the recommendation is to move forward soon. The discussion in the issue has also been generally supportive, and feedback on enabling both parameters has been positive.

- **Murphy**

    Any questions on the new Solidity compiler version or this proposal? So the initial direction is to advance the TVM-related parameters first, with the timing of TIP-833 and TIP-836 still open for discussion. Is that right?

- **David**

    Yes, that's the initial direction. Parameters 95 and 96 would come first. Whether TIP-833 and TIP-836 should be included in the same round or handled separately still needs further discussion in the relevant issues.

- **Murphy**

    Got it. If there are no other questions, let's move to the last topic. Steven will introduce the features and changes in the new `wallet-cli` versions.

<span id="topic4"></span>
**Introduce the `wallet-cli` TypeScript Implementation (v4.9.7–v4.10.0)**

- **Steven**

    The latest `wallet-cli` release is [v4.10.0](https://github.com/tronprotocol/wallet-cli/releases/tag/wallet-cli-4.10.0). A TypeScript implementation was introduced in [v4.9.7](https://github.com/tronprotocol/wallet-cli/releases/tag/wallet-cli-4.9.7). This section covers the progress from v4.9.7 to v4.10.0, the agent-first architecture, and an example agent workflow with `wallet-cli`.

    The TypeScript implementation is designed primarily for AI agent workflows and follows an agent-first architecture. The goal is to give agents a common CLI for TRON-related operations. The first phase is to gradually cover the major features already available in the Java implementation. Under the current plan, the core feature set is expected to be largely complete around v4.12.0.

    The current feature set includes basic wallet management, balance queries, token and contract operations, transactions, staking, voting, and rewards. Next steps include Account Permission Management, GasFree, and other capabilities. For now, development is focused on the TypeScript implementation.

    Traditional CLI output is designed for human users. It often uses tables, colors, and emoji, and may omit some context to make the result easier to read. Agents have different requirements. They need stable, structured output that programs can process. For example, errors need a clear error code, while successful results can consistently include fields such as `success` and `data`.

    For this reason, `wallet-cli` introduces a machine-readable result envelope and routes all output through a single module. This avoids scattered `print` calls across the project and keeps output formats consistent. In addition to `--help`, the `--json-schema` flag can output more complete parameter and field information, such as an address field's name, type, and constraints, which helps agents understand and compose commands.

    Secret handling has also been standardized. The TypeScript implementation uses an encrypted local keystore, with private keys and other sensitive information protected by a master password. The password does not need to be passed through a flag or environment variable; it can be supplied via standard input (stdin) or a TTY prompt. With `--password-stdin`, `wallet-cli` reads the password from stdin (FD 0). This allows a password stored in 1Password, a system keychain, or another password manager to be supplied via stdin without writing it to disk or exposing it in shell history or logs.

    The overall flow is as follows: a command first enters the parser and router, then the relevant policy checks run, such as whether a password or network is required. The command then enters the execution layer. If signing is needed, it calls the wallet signer. If chain access is needed, it goes through the TRON RPC gateway abstraction, whose backend can connect over gRPC or HTTP. The result then enters the unified output normalization module.

    Both text mode and JSON mode are supported. Text mode is easier for human users to read and can retain tables. JSON mode is designed for agents and provides a fixed structure that reduces ambiguity during parsing.

    For example, suppose an agent is asked to send 1 TRX to an address. It first uses `wallet-cli` to collect the balance, account details, transfer amount, fee, and other information. It then asks for confirmation and indicates that a password is required. If the password is stored in a 1Password vault, the output of `op read` can be piped directly to `wallet-cli`. This avoids placing the password in command arguments. After the transaction is completed, `wallet-cli` returns a structured result.

    For human users, supplying a password via stdin is less intuitive than interactive input. Some commands involving sensitive input already support hidden TTY input. The longer-term direction is to improve human-friendly interaction, while the current priority is to make all commands easy for agents to compose and call.

    Under the current plan, v4.11.0 and v4.12.0 are expected to continue adding and improving TRON-related commands. Later work may add more DeFi and protocol-transaction features and explore multichain support, possibly starting with EVM-compatible chains. These ideas are still at the planning stage.

- **Murphy**

    After this `wallet-cli` redesign, does the command flow distinguish between human users and agents at any point? Or is the overall design now more agent-oriented?

- **Steven**

    There is currently no distinction between agents and human users at the entry point. The main distinction is between interactive and non-interactive input. All commands remain available to human users, although supplying a password via stdin is not the typical interactive workflow.

    For example, a password can be entered through a hidden interactive prompt or supplied via stdin. Putting a password directly in a flag is less secure because it may appear in shell history or logs. This part is currently designed mainly for agent scenarios. Whether every command that requires a password should support a complete interactive flow still needs further evaluation.

- **Murphy**

    When a human user runs a command, does the output stay the same as before? Or has all output been changed to a format that is easier for agents to read?

- **Steven**

    The default remains text mode, which is easier for human users to read and still includes tables. An agent can set the output mode to JSON to receive a fixed, structured result. Both modes are supported, so the existing human-facing experience is largely unchanged.

- **Zeus**

    Which model does `wallet-cli` integrate with? If the code is public, could API keys or similar information be exposed?

- **Steven**

    `wallet-cli` itself does not integrate with a large language model or require a model API key. It is a standalone CLI tool. The agent or model calls `wallet-cli`; `wallet-cli` does not embed a model. It therefore does not depend on any specific model. Claude, Codex, Kimi, and other agents can all call it.

- **Murphy**

    Are there already similar DeFi CLIs available for agent use? Is there a plan to bring this kind of protocol interaction into `wallet-cli` and make it available through the same interface?

- **Steven**

    There is already a similar DeFi CLI that agents can use. These capabilities could later be integrated directly into `wallet-cli`, or `wallet-cli` could call the existing CLI. Both options remain under consideration. Longer term, `wallet-cli` may also support more DeFi functions.

- **Zeus**

    Recent `wallet-cli` releases include both Java and TypeScript implementations. Why was the TypeScript implementation added? Will both versions need to be maintained going forward?

- **Steven**

    Java has relatively high JVM startup overhead in short-lived, repeated-call scenarios. The original Java architecture was also not designed around agent-first usage, and its output was spread across different modules. The TypeScript implementation was added to refactor the architecture around agent-first usage.

    For now, the Java implementation will continue to be maintained. When new TRON protocol features or TIPs need support, the Java implementation will also be updated, while continuing to use a long-running, interactive mode. The long-term status of the Java implementation remains open as the TypeScript implementation matures.

- **Murphy**

    Got it. Further discussion on the CLI can continue in the relevant issues.

    Thanks everyone for joining the 67th Core Devs Meeting. That's all for today. See you next time!

### Attendance

* Blade
* Boson
* David
* Federico
* Gordon
* Leem
* Daniel
* Patrick
* Lucas
* Mia
* Neil
* Neo
* Steven
* Tina
* Wayne
* Robert
* Zeus
* Jeremy
* Vivian
* Murphy
* Erica


