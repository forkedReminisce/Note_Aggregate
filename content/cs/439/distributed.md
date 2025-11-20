---
draft: true
title: 

params: 
    desc: 
    author: FREEZURN 
---



# {{< heading "Parallel Computing" >}}
Processors will share the same hardware clock, physical address space, and encompassing OS. As in, they're in the same machine.

The Symmetric Multi Processor (SMP) evolved from the single CPU setup. It put multiple CPUs on the motherboard. Then came multicore processors, which is how a lot of modern hardware is designed.

Virtualization allows multiple OSs run on the same machine at the same time. This introduces the *hypervisor*, that intercept and handle hardware requests from the OSs.   



# {{< heading "Distributed Systems" >}}
Distributed systems are loosely-coupled, unlike parallel systems. Processors communicate with each other via some sort of connection. As a result, communication is more expensive than parallel computing. 

Sometimes, requests need to be ordered based on when it was generated (e.g., FIFO). Time stamps won't work because of clock drift—out of sync clocks because they run at different speeds. The solution is **total event ordering**: each process has a logical clock. When it does anything locally, it increments its logical clock. When it sends a message to another process, it will also send a time stamp of its logical clock. When the other process receives the message, it will increment its logical clock and set it to the first process' clock +1 if its logical clock is less. Now, if messages from other processes are time stamped before the receiver's logical clock, nothing can be said about those messages' orderings.

Distributed Consensus describes atomicity between two machines at a specific time. This is not possible because of the possibility of link failures (constant dilemma of needing to send a confirmation message for the received message (of a received message of a received message...)). 

The two-phase commit protocol will just provide atomicity. The first phase will see the coordinator (requesting machine) will send a request to every machine with work to do. The participants will then see if they can execute the request. It will then write `VOTE_COMMIT` or `VOTE_ABORT`, depending on whether it can or not, to its local log and send the same response to the coordinator.

If any participant sends a `VOTE_ABORT` or times out, the coordinate will write and send a `GLOBAL_ABORT` to every participant to discard the changes. Otherwise, it will write and send a `GLOBAL_COMMIT`. The participants write this response to their logs. If the coordinator crashes and there is no `GLOBAL_*` in its log, it will just send out `GLOBAL_ABORT`. Then the participants will execute the actions.

<!-- TODO: are numbers arbitrary in a sense or is it priority -->
Some algorithms require a leader process. If the leader dies, the *bully algorithm* figures out who becomes the next leader. The setup is numbered processes. If the leader is not heard from a while, it is assumed that it crashed. Processes that think they have a high enough number will broadcast their running for election to all involved processes. Processes with even higher numbers will tell lower numbered candidates to back off. The last process to claim leadership is recognized as leader by all the processes. If a former leader arises from the dead, they bully their way back into leadership.



<!-- transparent file access: use it like it was local (but it isn't!) -->
# {{< heading "Networked And Distributed File Systems" >}}
<!-- TODO: when i want a remote file, how is the mount table used -->
The **Network File System** (NFS) is the most widespread distributed file system. It uses *implicit naming*, meaning the full file name doesn't include the server's name. It does this through the NFS Mount protocol, which maps a directory to a remote directory. 

Performance is granted by **Remote Procedure Calls** (RPC). When the client calls one, it actually gets sent to a stub, which is library code that handles communication. When the server receives something, its stub will make a local procedure call to let the server do its job. ==The client will have to be granted another cache, which is typically in memory==.

{{< subtext >}}
    Disk latency and network latency are roughly equivalent.
{{< /subtext >}}

But what if a client is reading a file another client is writing? *Coherence* states that a read should return the latest write. A lack of *staleness* says that when a data store is written to, it must start returning the new value by some time period. *Consistency* is maintaining order of reads and writes across locations (not necessarily machines). These three define **consistency**.

Sequential Consistency requires a total ordering. However, the *CAP theorem* states that one of consistency, availability, and partition tolerance must be dropped. The choice is consistency. FIFO Consistency has a single process' writes in order, but it may be interleaved with another process' writes. Some writes may be dependent on a prior write, however, so Causal Consistency requires these dependent writes to be in order.

NFS' *client-initiated weak consistency protocol* has clients periodically check in for file updates. When a client changes a file, it lets the server know. It uses a stateless protocol, meaning the server maintains no state (e.g., file position) about the client. As such, operations must be idempotent. Server failures are also transparent to clients, allowing them to continue operating without issue.

{{< subtext >}}
    No state does require file requests to take in complete information (e.g., `ReadAt(inode, position)` rather than `Read(inode)` since position state is not kept).
{{< /subtext >}}

The **Google File System** (GFS). Files are huge, most modifications are appends (very few random writes), many files are written once and read sequentially, large streaming reads and small random reads in the forward direction. Hosted on clusters each with tens to hundred thousand nodes, component failures are normal, and sustained bandwidth is more important than latency. Many people want to access the files (scalability). Scalability is achieved with distribution, replication (duplicate the file across multiple servers), and allow the client to cache.

Each cluster has a single master that stores metadata. It is a single point of failure. Client talk to chunkservers, which are all the remaining servers. Files are divided into 64MB-sized chunks. They are 3-way replicated and stored on chunkservers. Chunkservers may request to be the temporary primary chunkserver for a chunk. Client application is on top of a GFS client, which is equivalent to a stub.
1. Application interfaces with GFS client to get a file
2. GFS client communicates with master to get the chunkserver and where inside that chunkserver the file is
3. Master tells chunkserver the GFS client has permission
4. GFS client requests chunkserver for data
5. Chunkserver returns data

{{< subtext >}}
    The master keeps metadata in memory. Since this is not durable, it keeps an operation log and also periodically stores a snapshot of memory onto disk. 

    If the master is temporary down, there is a shadow master ready to take over.

    Chunkservers can fail without hindering the entire cluster. They do periodically send a heartbeat message back to the master to let it know that it's still alive.
{{< /subtext >}}

GFS' consistency model is **relaxed consistency**. Concurrent changes are undefined, meaning they don't appear atomic. Appends are atomic, though. Updates are applied in the same order at all replicas. Sometimes, staleness is permitted.