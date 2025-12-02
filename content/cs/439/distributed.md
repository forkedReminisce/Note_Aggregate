---
draft: false
title: Distributed Systems

params: 
    desc: Distributed systems are loosely-coupled systems. The nodes work together to achieve some sort of common goal.
    author: Andrew Nguyen 
---



Distributed systems are loosely-coupled systems. Processors communicate with each other via some sort of connection. 

Sometimes, requests need to be ordered based on when it was generated. Time stamps won't work because of clock drift—out of sync clocks because they run at different speeds. The solution is **total event ordering**: each process has a logical clock. When it does anything locally, it increments its logical clock. When it sends a message to another process, it will also send a time stamp of its logical clock. When the other process receives the message, it will increment its logical clock and set it to the first process' clock +1 if its logical clock is less. 

Distributed Consensus describes two machines that will always agree at any time. This is not possible because of the possibility of link failures (constant dilemma of needing to send a confirmation message for the received message [of a received message of a received message...]). 

The **two-phase commit protocol** will just provide atomicity, but not necessarily at the same time. The first phase will have the coordinator make a request to every relevant machine. The participants will then see if they can execute the request. It will then write `VOTE_COMMIT` or `VOTE_ABORT`, depending on whether it can or not, to its local log and send the same response to the coordinator.

If any participant sends a `VOTE_ABORT` or times out, the coordinate will write and send a `GLOBAL_ABORT` to every participant. Otherwise, it will write and send a `GLOBAL_COMMIT`. The participants write either to their logs. In the case of a `GLOBAL_COMMIT`, the participants will also execute the actions. If the coordinator crashes and there is no `GLOBAL_*` in its log, it will just send out `GLOBAL_ABORT`. 

<!-- TODO: are numbers arbitrary in a sense or is it priority -->
Some algorithms require a leader process. If the leader dies, the *bully algorithm* figures out who becomes the next leader. The setup is numbered processes. If the leader is not heard from a while, it is assumed that it crashed. Processes that think they have a high enough number will broadcast their running for election to all involved processes. Processes with even higher numbers will tell lower numbered candidates to back off. The last process to claim leadership is recognized as leader by all the processes. If a former leader arises from the dead, they bully their way back into leadership.



# {{< heading "Networked And Distributed File Systems" >}}
<!-- TODO: for mounting, is a file added to the directory as an entry? -->
The **Network File System** (NFS) is the most widespread distributed file system. One thing it does is *location-transparency*, meaning that it's hidden that the file is stored remotely. Implicit naming doesn't include the server's name in the full file name. And the NFS Mount protocol creates the illusion that a remote file is locally stored with a mapping in the map table.

<!-- TODO: why do RPCs exist over system calls -->
Servers can provide operations to clients with *Remote Procedure Calls* (RPC). So when a client calls one, it is directed to a stub that sends data to the server. The server has a stub of its own whose sole purpose is to receive and make local procedure calls to the server. The flaw with RPC is hanging the program because of failures on the remote machine. It's also really slow.

{{< subtext >}}
    Disk latency and network latency are roughly equivalent.
{{< /subtext >}}

To optimize the number of RPC operations, the client will get a cache. ==But what if a client is reading a file another client is writing?== This necessitates **consistency**: 
- *Coherence*: a read should return the latest write 
- A lack of *staleness* says that when a data store is written to, it must start returning the new value by some time period 
- *Consistency*: maintaining order of reads and writes across locations (not necessarily machines).

Sequential Consistency requires a total ordering. However, the *CAP theorem* states that one of (strong) consistency, availability, and partition tolerance must be dropped. The choice is consistency. FIFO Consistency has a single process' writes in order, but it may be interleaved with another process' writes. Some writes may be dependent on a prior write, however, so Causal Consistency requires these dependent writes to be in order.

NFS' *client-initiated weak consistency protocol* has clients periodically check in for file updates. When a client changes a file, it lets the server know. It uses a stateless protocol, meaning the server maintains no state (e.g., file position) about the client. As such, operations must be idempotent. Server failures are also transparent to clients, allowing them to continue operating without issue.

The context for the **Google File System** (GFS) is extensive. Files are huge, most modifications are appends, many files are written once and are read sequentially, and there will be many clients accessing the same files (i.e., scalability). Its to be hosted on clusters and must handle component failures. Finally, sustained bandwidth is more important than latency. 

Each cluster has a single master that stores metadata. The actual data lives on chunkservers, all the other nodes. Files are divided into 64MB-sized chunks and are 3-way replicated for scalability. 
1. Client interfaces with GFS client to get a file
2. GFS client communicates with the master for the chunkserver and where in that chunkserver
3. Master tells chunkserver the GFS client has permission
4. GFS client requests chunkserver for data
5. Chunkserver returns data

{{< subtext >}}
    The master keeps metadata in memory. Since this is not durable, it keeps an operation log and also periodically stores a snapshot of memory onto disk. 

    If the master is temporary down, there is a dedicated shadow master ready to take over.
{{< /subtext >}}

GFS' consistency semantic is *relaxed consistency*. Concurrent changes are undefined, meaning not all clients will see the change in its entirety (not atomic). Unless it's an append. Across all replicas, the updates are in the same order. Sometimes, staleness is permitted.