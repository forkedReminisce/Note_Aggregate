---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Clusters might be preferred over scaling up because adding more machines is cheaper than increasing the number of cores. Scaling out also have a higher aggregate memory ceiling.



# {{< heading "MPI" >}}
Processes are grouped. The group is gets a communicator (i.e., name), and each process gets a rank (i.e., name). Each process runs `main`, but their rank is different. 

There are two types of communication. *Point-to-point* has one process send a message (e.g., `int` or even arrays or structs) to another process. *Collective communication* offers ways for processes to communicate to other processes of the group (e.g., broadcast or barriers). This allows the overlap of computation and communication.

For point-to-point, a blocking send/receive requires both the sending and receiving process to be ready for the communication to happen. This brings synchronization as well. Nonblocking allows the sender to just send. If the receiving process is not ready, MPI will buffer the message. However, the sender can send multiple messages before the receiver is ready. There may be a reordering when they arrive, but the receiver can specify which message to read. There is a flag that is set when the receive is conducted; it is set to true if the data is transferred or false if there is no data to read. 

Asynchronous send/receive gets the benefits of both blocking and nonblocking. On send, flag1 is set when the data is sent out. Receive needs to register beforehand where to place the data, and flag2 is set when the data is written. 

<!-- root is not a particular process; it is specified per call -->
Collective communication has a multitude of options. Parallelism can be achieved by assigning certain ranks to do the intermediary steps (i.e., like a tree).
- One-to-all broadcast: one process sends data to all other processes in group
- Reduction: each process sends their result to a process for it to combine
- All-to-all broadcast: every process sends data to every other process
- Barrier
- Gather: each process sends a value to the root process in rank order
- Scatter: root sends a value from a array to each process in rank order 

{{< subtext >}}
    All-reduce is reduce but every process has the final result.
{{< /subtext >}}

MPI implements these for you. `MPI_Datatype` exists as an intermediate format for translation from the sender to the receiver. For a broadcast, every process of the process group have to execute broadcast before any process can prove forward. Reduction has an `MPI_OP` that defines allowed associative operations, and `recvbuf` is only updated for root. 

When working with matrices, there are a number of ways to partition rows/columns. Block does it like contiguous, while cyclic alternates assignment. 2D partitioning (or block-block partitioning) gives each machine a block from a checkerboard pattern. Partitioning strategy is done by the programmer. The right partitioning strategy can optimize communication. 