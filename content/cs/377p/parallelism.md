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
<!-- TODO: does pinning prevent other cores from modifying the cache line -->
*Data-races* are the result of interleaving memory instructions. Atomic instructions seek to provide mutual exclusion against this problem. Since they operate on the size of a cache line, they just pin the cache line. However, if the data is too big for a cache line, locks should be used.

What we are familiar with are mutex-locks. **Spin-locks** (or trylocks) instead do not block the thread; they require the thread to manually retry. These types of locks are implemented with the `swap` instruction. For an acquire, a register with value `1` is swapped with the value of the lock. If the register still reads `1`, the lock wasn't acquired. ==However, the cache line of the lock is technically modified, invalidating every other cache's line==. This will lead to busy-waiting by ping-ponging. A solution is test-and-test-and-set, where a single `swap` is after a perpetual loop of just the check.

**Barriers** require all threads to reach it before continuing. It's a struct that contains an arrive count, leave count, flag, and a lock for incrementing counts. The flag is `true` if all threads have reached the barrier. The final thread sets this, and it also resets the arrive count to `0`. The first thread to leave sets the leave count to `1`. The flag is reset when the first thread arrives. 

Other than lock ordering, another way to address deadlock is self-preemption. In this scheme, if a thread cannot acquire a lock, it releases all the locks it holds. However, this introduces livelock—doing no useful work. Exponential backoff does address this. 



# {{< heading "Data-Parallelism" >}}
`map` performs an operation on each item in the passed in array. `reduce` applies an associative function on each item with everything before. `reduce-by-key` does `reduce` on values with matching keys. `filter` returns a subarray that satisfies a decision function. `scan` returns an array where `reduce` has been run up to that point. 

Parallelism of `reduce` and `reduce-by-key` uses divide-and-conquer, with each thread processing a subarray and writing the result to an array. Tree reduction is a strategy of starting from the root building a binary tree of threads until the leaves become manageable. Then the internal threads combine the partial sums from their children. 

{{< subtext >}}
    Tree reduction can turn algorithms into \[log(n)\].
{{< /subtext >}}

`scan` introduces the notion of the result of the previous index (`fromLeft`). There will be two passes. In the up-sweep, each thread performs `reduce` on their subarray and only write their final result to an auxiliary array. `scan` is run on this array. In the down-sweep, each thread then grabs its respective value and then calculates its output array. In tree reduction, the `fromLeft` of internal nodes is the the `fromleft` of the leftmost leaf. Therefore, the `fromLeft` of the right child is the sum of the left child plus the `fromLeft` of this node. Sums of nodes are from up-sweep. 

{{< subtext >}}
    This relies on the associativity of the function.
{{< /subtext >}}

Since `filter` produces an output array with unknown size, `map` it with the decision function, `scan` it with addition, then `map` it again into an output array. The size of the array is from the final value of the `scan`.

The best number of threads can be found as \[p = \sqrt{\frac{c}{o}}\], where \[c\] is the total computation cost with just one thread and \[o\] is the time to create, join, and close a thread.



# {{< heading "Memory Consistency Models" >}}
Reordering may change the order of stores or loads because of a lack of dependencies between them, which may affect the semantics for another thread. One reason this reordering happens because store operations must wait in the store buffer, while loads don't have an equivalent buffer, so they bypass. But although dependencies are not presents, multithreaded programs possess implicit dependencies.
<!-- only reason about program order since the interleavings are fine -->
- Sequential Consistency: don't reorder operations on shared-memory (but can reorder stack), but it prohibits store buffer optimizations and all global variable operation is slowed
<!-- dont interpret laymans definition strictly; sequential would put fences after every data operation -->
- Weak consistency: Fence instruction require all data operations before to be complete in program order, and data operations after must wait until instructions before complete. Fence is implemented with a count that is implemented when the operation is issued, and decremented when the operation returns
- Release consistency: just another relaxed consistency model

This is concerned with data operations within a core. It basically allows the enforcement that data operations are done in a potentially desired order by that particular core. However, these do not ensure deterministicity.

What was described was memory consistency models at the architecture level, but programming languages also have their own memory models.