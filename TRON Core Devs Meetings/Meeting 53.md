# Core Devs Community Call 53

### Meeting Date/Time: January 14th, 2026, 6:00-7:00 UTC
### Meeting Duration: 60 Mins
### [GitHub Agenda Page](https://github.com/tronprotocol/pm/issues/181)
### Agenda

  - [Syncing the development progress of v4.8.1](https://github.com/tronprotocol/java-tron/issues/6342)
  - [Discussion on Secure Administrative Framework for FullNode Based on IPC and JSON-RPC](https://github.com/tronprotocol/java-tron/issues/6497)
  - [Issue - Node fails to synchronize blocks after a peer is randomly disconnected](https://github.com/tronprotocol/java-tron/issues/6504)

### Detail

- Murphy

    Welcome to the 53rd TRON Core Devs Community Meeting. We have three topics on the agenda today. First, I’ll invite Neo to sync the development progress of version 4.8.1.

**Syncing the development progress of v4.8.1**

- Neo 

    Version 4.8.1 recently incorporated two bug fixes and merged the v4.8.0.1 patch. Testing was initiated yesterday. If testing proceeds smoothly, we will move forward with a progressive deployment of Mainnet nodes, which is expected to take 1 to 2 weeks, followed by the official release.

- Murphy 

    So the release of v4.8.1 is expected within 1 to 2 weeks starting from this week? Will it be delayed due to the release of v4.8.0.1?

- Neo 

    We are currently weighing our options. Releasing versions too frequently may increase the upgrade burden on the community. I will further confirm the specific timeline soon.

- Murphy 

    Does anyone have other questions regarding v4.8.1? If not, let’s move to the second topic. Zeus, please introduce the FullNode administrative framework based on IPC and JSON-RPC.

**Discussion on Secure Administrative Framework for FullNode Based on IPC and JSON-RPC**

- Zeus
    
    Alright. This issue introduces a mechanism for dynamic interaction with FullNode. For instance, reading information, modifying parameters, and adding or removing peers, and these operational parameters can be adjusted dynamically. It is quite similar to Ethereum’s Geth Console, which provides a way to interact with a running node. Our framework is designed with more room for expansion and is currently undergoing deep adaptation for our specific features.

    Interacting with nodes requires a security mechanism to prevent remote hacking. Therefore, a strict security protocol is necessary; generally, only local access is permitted, while remote access is restricted.
        
    There are two ways to implement this mechanism: one based on HTTP interfaces and the other based on inter-process communication, or IPC. These two interfaces are identical in terms of definition, parameter validation, and execution logic; the only differences lie in the communication channel and the security margin. One is implemented via the HTTP protocol, and the other via Sockets.
    
    Regarding the security margin, IPC only allows local access, making it absolutely secure. For HTTP, we can control both local and remote access by default.
    
    For example, let’s define a JSON-RPC interface: a function named `admin_example` with two parameters, `param1` and `param2`, which returns their sum.
        
    One approach is the HTTP-based interface. This method involves starting an independent HTTP service, using POST calls to pass parameters, and returning results through the frontend. This method has extremely strict security requirements. As an independent HTTP service, it listens to `localhost` by default. This listening address can be specified in the configuration, with the default set to `localhost` to avoid external exposure. If necessary, the listening address can be adjusted. This method is suitable for automated O&M, remote management, and flexible integration with external systems.
    
    The other approach is an IPC mechanism based on Unix Domain Sockets. It does not expose any ports. When the FullNode starts, we generate a Socket file named after the Process ID (PID) in a temporary directory. Then, an IPC Service is started within the FullNode and bound to this Socket, with a thread listening for request messages from that Socket file.


- Blade 
    
    We could consider placing it in a unified directory along with the database.

- Zeus
        
    That might not be ideal. We don't want it to access the database directory. It would be quite troublesome if something were deleted by mistake. I suggest placing it in the temporary folder (`/tmp`). Due to security permissions, deleting a Socket file generated this way is harmless, but deleting database files would be a major issue. Currently, the file is named using the PID, which changes with every startup to ensure no naming conflicts. The Socket file is deleted when the FullNode shuts down normally, so no residue is left behind.

- Blade 

    Won't we end up with residue buildup if these files continue to be created?

- Zeus
    
    There won’t be residue. It will be deleted upon a normal shutdown. If it weren't deleted, you'd end up with a pile of junk files after several runs. If two nodes are started, they cannot use files with the same name, so keeping it simple is best. This naming convention refers to many third-party projects, such as Ethereum’s Geth. Upon startup, the log will display the path of the temporary directory. Being too flexible isn't good; providing a fixed method actually reduces the learning cost for users.

- Mia

    Is there a possibility of JS attacks in that directory? How is security guaranteed?

- Blade 

    It needs to be deleted upon closing, right? If we used a unified name, deletion wouldn't be necessary; we could just reuse the same name next time. Or we could allow the filename to be specified in the configuration.

- Zeus
    
    I recommend against unified naming primarily because this interface is highly sensitive and access shouldn't be handled casually. For example, we need to strictly prevent unauthorized cross-user access to the process. If the permission logic becomes too complex, it’s actually more likely to create security loopholes. Therefore, we are keeping the architecture simple and avoiding unnecessary complexity. This not only reduces risk but also ensures that the interaction system can be smoothly rebuilt and recovered in special circumstances.


- Murphy

    Will this JSON-RPC service be included in the java-tron client?

- Zeus
    
    Yes, it will be included in the java-tron client. It will start alongside the FullNode unless you manually specify a different instruction.

- Murphy
    
    Also you mentioned this feature can query all running non-sensitive parameters. How do you ensure it can reach full coverage?

- Zeus
    
    Right. The parameters come from `CommonParameter`. Once loaded, they go through a processing and validation layer; the data coverage is quite thorough. I’ve implemented masking for sensitive information like private keys or specific directories. The non-sensitive parameters are more than enough for operational needs, and all control items are currently accessible. There are relatively few commands at the moment because changing them is quite complex. If we read directly from `CommonParameter`, no code changes are required. (Murphy: Got it.)

- Wayne
    
    By default, are the command interfaces, including the IPC method, enabled or disabled?

- Zeus

    IPC should be enabled by default, but JSON-RPC will be disabled by default.

- Wayne
    
    Why not disable both of them?

- Zeus

    Because the risk associated with IPC is low; it only allows local access and is based on file system permissions.

- Wayne

    The current plan is for the IPC method to function the same way as the `Admin` Namespace, correct?

- Zeus

    Yes. Since the underlying implementation is the same, this is currently for the sake of scalability.

- Wayne
    
    It’s not that starting via IPC provides more features?

- Zeus

    Currently, the functions are exactly the same.

- Wayne
    
    Ethereum’s IPC method supports many functions, including interfaces that standard Admin interfaces do not support, such as creating accounts or querying transactions. I’m concerned that if we don't plan this well now, we might find the code too tightly coupled to modify during later expansion.

- Zeus
        
    That's fine. We can support more commands in the future. For the first version, we've made the two functions identical for simplicity. We can support more features later; it doesn't have to be strictly bound to the existing Admin functionality.

- Wayne

    Alright. As long as it's planned well in the early stages. I have no other questions, thank you.

- Murphy

    
    Alright. If there are no more questions on this topic, Lucas will discuss the bug involving failed reconnections following a random peer disconnection.


**Issue - Node fails to synchronize blocks after a peer is randomly disconnected**

- Lucas
    
    Ok. This is an issue regarding Random Elimination. It’s a low-probability event that has only occurred once in the last six months. In this scenario, a node reaches its connection limit (e.g., 30 peers), but it is only actually receiving blocks from one peer. The other peers are likely "attack nodes" that occupy slots without providing data. When the "Random Elimination" mechanism triggers, it may accidentally disconnect the only healthy peer, causing the node to stop syncing.

    Regarding a fix, we considered several options. The first was to disable the "Random Elimination" feature entirely. However, doing so reduces the efficiency of finding new connections. Previously, when nodes started, they frequently triggered "Too Many Peers" errors; the connection slots would be full, yet the node couldn't connect to valid peers, sometimes making the search process take 30 minutes.
    
    The "Random Elimination" feature was originally launched to solve this: when connections are full, it proactively disconnects a random peer to free up a slot for a new one, keeping the connection pool active. To clarify, the high number of invalid connections is caused by "attack nodes" that occupy slots by mimicking normal nodes but do not broadcast block data.
    
    Our current optimization plan is to retain the feature but refine the algorithm to prevent the "accidental killing" of the only healthy connection. We will add a "Last Block Received Time" field to the `Peer` object and sort peers based on this timestamp. We will protect the one or two most active nodes from being disconnected, only randomly dropping peers that have had no data interaction for a long time.

-  Zeus
    
    What is the primary indicator used for this interaction?

- Lucas
    
    The primary indicator is the "Last Block Received Time". Specifically, recording the most recent time a block was received from a given peer.

    This approach is relatively conservative and unlikely to cause major issues. If we were to remove the feature entirely, nodes might struggle to find valid connections for a long time after startup.

- Murphy

    To clarify, when you say "one healthy connection," do you mean it was the only one capable of receiving blocks, or was it a specific type of connection?

- Lucas

    It was a completely normal, functional connection that simply got "unlucky" with the random algorithm.

- Murphy
    
    In the background description, the node already has 30 connections and is at full capacity, with only one connection being normal. After the Random Elimination mechanism disconnected that normal connection, it resulted in an inability to receive blocks from other nodes. I want to confirm: every such normal connection is capable of receiving blocks, right? This normal connection isn't pointing to a specific link. Since it is normal, why was it randomly disconnected?

- Lucas
    
    This is because the random elimination algorithm did not sufficiently consider connection quality factors; its execution was too random. Previously, a "weighted random" elimination was used, meaning every peer, regardless of quality, still had a probability of being dropped.

- Leem

    Is this "Random Elimination" logic handled at the P2P network layer or within the upper java-tron application layer?

 - Lucas

    It is currently implemented within the java-tron layer.
    
-  Zeus
    
    The current implementation uses a weighted random function. We look at the time difference between the local node and the peer; a smaller gap suggests an active, syncing node, which theoretically lowers its probability of being disconnected. However, "lower probability" is not a guarantee.
    
    The underlying issue is that our P2P identification mechanism (Peer Selection) is still not robust enough to perfectly distinguish between an honest node and an attack node. Improving the Random Elimination algorithm is a necessary "bandage" to mitigate this while we work on better identification.

- Murphy

    So the optimized algorithm mentioned just now determines that if a connection has recently received a block, it will not be disconnected, right?

- Lucas

    Correct. We will ensure the top one or two active peers are never disconnected, because these two or three nodes may be "good neighbors."

- Murphy

    Okay. Does anyone have further suggestions or alternative strategies? If not, please continue the technical deep dive on GitHub Issue #6504. If there is nothing else, we will conclude today’s meeting. Thank you all for participating.



### Attendance

* Aiden
* Patrick
* Blade
* Mia
* Jeremy
* Boson
* Federico
* Gordon
* Leem
* Daniel
* Neo
* Tina
* Vivian
* Wayne
* Elvis
* Erica
* Robert
* Murphy
* Zeus
