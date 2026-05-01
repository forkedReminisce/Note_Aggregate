---
draft: false
title: MPI

params: 
    desc: MPI facilitates communication between nodes. This communication can either be point-to-point or collective, each offering various methods.
    author: Andrew Nguyen 
---



A group is has a communicator (i.e., name), and each of its processes gets a rank (i.e., name). Each process runs `main`. Member processes can communicate with other processes either through:
- Point-to-point: to one other process
- Collective: to every other process of the group

There are three options for point-to-point communication: 
- Blocking: sending and receiving process must be ready, offers synchronization
- Nonblocking: if the receiver isn't ready, buffer the message
- Asynchronous:  receiver registers beforehand where to place the data when it comes in

{{< subtext >}}
    Nonblocking has a single flag for receive, while asynchronous has two flags.
{{< /subtext >}}

Collective communication comes in many forms. In any case, they're implemented by MPI. For many of the functions, a `MPI_Datatype` needs to be specified as an intermediate formate for translation from the sender to the receiver. 
- One-to-all broadcast: one process sends to all other processes in group
- Reduction: each process sends their result the root for it to combine (and hold result)
- All-to-all broadcast: every process sends to every other process
- Barrier
- Gather: each process sends a value to the root in rank order
- Scatter: root sends a value from a array to each process in rank order 

{{< subtext >}}
    Root is not some predefined process; it is specified per call.

    Broadcast also acts like a barrier, requiring every member process to execute it.
    
    Reduction has an `MPI_OP` for allowed associative operations, and `recvbuf` is only updated for root.
    
    All-reduce is reduce but every process has the final result.
    
    Parallelism can be achieved by assigning certain ranks to do the intermediary steps, like a tree.
{{< /subtext >}}

When working with matrices, there are a number of ways to partition rows/columns. Block does it like contiguous, while cyclic alternates assignment. 2D partitioning (or block-block partitioning) gives each machine a block from a checkerboard pattern. Partitioning strategy is done by the programmer. The right partitioning strategy can optimize communication. 