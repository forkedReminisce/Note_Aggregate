---
draft: false
title: Parallelism 

params: 
    desc: Parallelizing processes is not a straight-forward ordeal. New problems get introduced and the solution itself may need to be rethought.
    author: Andrew Nguyen 
---



# {{< heading "DAGs" >}}
Programs want to exploit parallelism. Therefore, each program has a *DAG*, where the nodes are computations and edges represent dependencies (i.e., topological ordering). Each node has a weight for the amount of time it will require. Summing all the nodes gets *work* \(T_1\). The *critical path* (or span) \(T_\infty\) is the largest weight path. Any schedule cannot do better than this.
- Instantaneous parallelism: \(\operatorname{IP}(t)\), maximum number of processors that can be busy at \(t\)
- Maximal parallelism: \(\operatorname{MP}\), highest instantaneous parallelism
- Average parallelism: \(\mathnormal{AP} = \frac{T_1}{T_\infty}\).

{{< subtext >}}
    \(T_1\) is the amount of time to run the program on one processor.
{{< /subtext >}}

The speed-up from adding parallel processors is limited by the fact that not all of the program can run in parallel. Let \(p\) be the fraction of the program that can be done in parallel. Then the speed-up with unlimited parallel processors is \(\leq \frac{1}{1 - p}\) (i.e., \(\frac{T_1}{T_\infty}\)).

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
<!-- TODO: does pinning prevent other cores from modifying the cache line -->
*Data-races* are the result of interleaving memory instructions. Atomic instructions seek to provide mutual exclusion against this problem. Since they operate on the size of a cache line, they just pin the cache line. However, if the data is too big for a cache line, locks should be used.

What we are familiar with are mutex-locks. **Spin-locks** (or trylocks) instead do not block the thread; they require the thread to manually retry. These types of locks are implemented with the `swap` instruction. For an acquire, a register with value `1` is swapped with the value of the lock. If the register still reads `1`, the lock wasn't acquired. ==However, the cache line of the lock is technically modified, invalidating every other cache's line==. This will lead to busy-waiting by ping-ponging. A solution is test-and-test-and-set, where a single `swap` is after a perpetual loop of just the check.

**Barriers** require all threads to reach it before continuing. It's a struct that contains an arrive count, leave count, flag, and a lock for incrementing counts. The flag is `true` if all threads have reached the barrier. The final thread sets this, and it also resets the arrive count to `0`. The first thread to leave sets the leave count to `1`. The flag is reset when the first thread arrives. 

Other than lock ordering, another way to address deadlock is self-preemption. In this scheme, if a thread cannot acquire a lock, it releases all the locks it holds. However, this introduces livelock—doing no useful work. Exponential backoff does address this. 



# {{< heading "Data-Parallelism" >}}
Many algorithms can be parallelized with simple constructs.
- `map`: perform a function on each item of the array
- `reduce`: returns the final result of applying an associative function on each item and all those before it grouped together
- `reduce-by-key`: `reduce` values with matching keys
- `filter`: returns a subarray of items that satisfy the decision function
- `scan`: returns an array where `reduce` has been run up to that point

Parallelism of `reduce` and `reduce-by-key` uses divide-and-conquer, with each thread processing a subarray. *Tree reduction* is a strategy of building a binary tree where the leaves are a manageable amount of divided work. The partial results are then combined by the parent thread.

{{< subtext >}}
    Associativity of the function comes in handy here.

    Tree reduction can turn algorithms into \(\log(n)\).
{{< /subtext >}}

Since `scan` returns an array instead of a single value, a thread will keep track of the result of everything before with `fromLeft`. To do this, there will be two passes. 
1. Up-sweep: each thread performs `reduce` on their subarray and write the partial result to an auxiliary array
2. Run `scan` on the auxiliary array
3. Down-sweep: each thread grabs its `fromLeft` from the auxiliary array and can now begin computing the resultant array

In tree reduction, the `fromLeft` of the internal nodes is propagated from the leftmost leaf it can reach. To get the `fromLeft` of the right child, add this `fromLeft` with the partial result of the left child. 

To work around the fact that the size of the array `filter` produces is unknown: 
1. `map` it with the decision function
2. `scan` the resultant array with addition; final value is `filter` resultant array size
3. Separate `map` that adds an item to the array if the respective item of the `scan` array differs from the previous item

The best number of threads to divide an algorithm is \(\sqrt{\frac{c}{o}}\), where \(c\) is the serial computation cost and \(o\) is the time to create, join, and close a single thread.



# {{< heading "Memory Consistency Models" >}}
If there aren't any dependencies, stores or loads may be reordered. This may affect the intention for another thread. To retain these implicit dependencies, **memory consistency models** are employed. Do note that they do not guarantee deterministic behavior.
- Sequential Consistency: thread cannot reorder memory operations (but they may be interleaved between threads), but all memory operations are slowed
- Weak consistency: fence instruction require all prior data operations complete, and data operations after must wait

{{< subtext >}}
    The fence instruction waits for a particular count to hit 0. This count is incremented when a memory operation is issued, and decremented when it returns.
{{< /subtext >}}
