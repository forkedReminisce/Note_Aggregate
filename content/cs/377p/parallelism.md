---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



# {{< heading "DAGs" >}}
Programs want to exploit parallelism. Therefore, each program has a *DAG*, where the nodes are computations and edges represent dependencies (i.e., topological ordering). Each node has a weight for the amount of time it will require. Summing all the nodes gets *work* \(T_1\). The *critical path* (or span) \(T_\inf\) is the largest weight path. Any schedule cannot do better than this.
- Instantaneous parallelism: \(\operatorname{IP}(t)\), maximum number of processors that can be busy at \(t\)
- Maximal parallelism: \(\operatorname{MP}\), highest instantaneous parallelism
- Average parallelism: \(\mathnormal{AP} = \frac{T_1}{T_\inf}\).

{{< subtext >}}
    \(T_1\) is the amount of time to run the program on one processor.
{{< /subtext >}}

The speed-up from adding parallel processors is limited by the fact that not all of the program can run in parallel. Let \(p\) be the fraction of the program that can be done in parallel. Then the speed-up with unlimited parallel processors is \(\leq \frac{1}{1 - p}\) (i.e., \(\frac{T_1}{T_\inf}\)).

Scheduling policies become very important in the presence of dependencies. A key idea is to make sure that a node on the critical path is always scheduled. If the DAG is a tree, the optimal schedule is to go level-by-level. Otherwise, a good heuristic give higher priorities the further a node is from the final node.

DAGs can be generated at compile-time or runtime in what is known as static scheduling or dynamic scheduling respectively. These look at the various dependencies (e.g., data dependencies). However, the use of conservative approximation most frequently suffers from may-aliasing. Additionally, static scheduling struggles with instructions that have variability in latency (e.g., load).



# {{< heading "Cache Coherence" >}}
Every processor has a cache of their own. Caches must be *transparent*, meaning the result should be the same even without caches. Since shared-memory programs jeopardize this property, **cache coherence** is necessary.

This problem can be solved with **bus snooping**—every cache snoops the bus. When a cache writes to a cache line, there are two potential protocols. *Write invalidate* sends a message telling every other cache to invalidate their copy, or *write broadcast* has this cache just send everyone a new copy. The latter isn't as widespread because bus transactions are expensive.

<!-- TODO: ??? -->
{{< subtext >}}
    Explanations get complicated with multi-level caches. If the levels are inclusive (e.g., all of L1 is in L2), only the lowest cache needs to snoop. However, cache capacity becomes smaller. This design can be ditched to have L1 snoop directly, but it adds latency to all cache accesses.
{{< /subtext >}}

The **MESI Protocol** implements write invalidate. Cache lines in each cache have four states: 
- Modified: (implies it is the only cached copy that is valid)
- Exclusive: same as main memory and the only cached copy
- Shared: same as main memory but other copies exist
- Invalid: not in cache or out of date

{{< subtext >}}
    On a read miss, the cache will make a bus request to main memory. If other caches have that cache line, they will prematurely end this memory access by copying the value onto the bus.

    Caches are write-back. If there is a read miss and a cache has that cache line in the modified state, it will also write the updated value to memory. The state of its cache line then becomes shared.
{{< /subtext >}}

<!-- TODO: do i need directory-based schemes -->

The MESI Protocol introduces a fourth type of C miss: **coherence misses**. This is because a cache line can become invalid, resulting in a cache miss when it is referenced again by that particular processor. This tends to pop up in the ping-ponging of cache lines between cores.
- True-sharing: multiple cores are reading and writing to the same memory address
- False-sharing: multiple cores are reading and writing to what happens to be the same cache line



# {{< heading "Synchronization" >}}
Interleaving of memory instructions leads to a *data-race*. These can "bypass" the MESI Protocol because they work with memory, not registers (or cache lines? cache miss?). Atomic instructions solve this issue. They operate on the size of a cache line.

Atomic instructions rest on ==pinning the cache line==. This allows operations to not fear the cache line getting evicted. The cache line is unpinned when the instruction finishes. If the operation is not implemented in the ISA, loop the operation and `compare-and-swap` until CAS succeeds. `compare-and-swap(addr, reg1, reg2)` checks if the value in `addr` is equal to the value in `reg1`, and if it is, it swaps the value in `addr` with `reg2`. However, if the size is too big for a cache line, a lock is used.

What we are familiar with are mutex-locks. *Spin-locks* (or trylocks) instead do not block the thread when a busy error code is returned; they require the thread to manually retry. These types of locks are implemented with the `swap` instruction. For an acquire, a register with value `1` is swapped with the value of the lock. If the register still reads `1`, the lock wasn't acquired. ==However, the cache line of the lock is technically modified, invalidating every other cache's line==. This will lead to busy-waiting by ping-ponging. A solution is test-and-test-and-set, where a single `swap` is after a perpetual loop of just the check.