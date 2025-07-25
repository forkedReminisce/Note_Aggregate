# Digital Circuits
The fundamental building block of digital logic circuits are digital logic gates. One would implement a simple boolean function (e.g., `NOT`, `AND`, `OR`). 

[TODO: table for `NOT`, `AND`, `OR`, `XOR` boolean notation and schematic]: #

## Combinational Logic Circuits
This type of circuit has no cycles. That is, the result of the function is not referenced in anyway the next time the function is run. 

One way to describe such a function is with the **sum-of-products** form. It is `AND` statements joined together with `OR`s. To create one, fill out the truth table for the function. For each `1` result, join the value of each input with `AND`s. Finally, join each statement with `OR`s.

[TODO: two-input multiplexer example here]: #
An example of a combinational circuit is the ripple-carry adder. These are chained full adders, each of which comprise of two half adders. A single full adder handles a single position's sum and carryOut. Since there are only two digits, `0` and `1`, if the sum would be `2`, carryOut would become `1` and sum "resets" to `0`. The carryOut of one full adder becomes the carryIn of the next full adder in a ripple-carry adder. Subtraction can be performed with a ripple-carry adder as well; just get the additive inverse of the second argument then set the initial carryIn to `1`.

[TODO: schematic here]: #
The barrel shifter is a way of performing shifts. Let $n$ be the number of bits in the argument, and $d$ the shift amount. This circuit comprises of $\log_2 n$ levels $l$, with each level containing $n$ `MUX2` circuits. The arguments to one of the multiplexers is the following: `MUX2(d[l], 0, 1)`. If true, the bit will go down the wire that shifts $2^l$ positions. Later multiplexers will have a bit `0` pass through in the case a bit did not pass through (i.e., the bit got shifted and needs to be replaced with a `0`).

## Sequential Logic Circuits
On the other hand, this kind of circuit has at least one cycle. It relies on state, a fixed finite-size "summary" of the history of input sequences to the circuit. The gory details are not recorded in state.

[input or output?]: #
The **set-reset latch** is the basic building block for history. It takes two inputs $R$ and $S$, which correspond to set and reset respectively. The outputs of the circuit are $Q$ and $\bar{Q}$. The definition is as follows:
- If $R$ is `1` and $S$ is `0`, $Q$ is `0` and $\bar{Q}$ is `1`
- If $R$ is `0` and $S$ is `1`, $Q$ is `1` and $\bar{Q}$ is `0`
- If $R$ and $S$ are `0`, latch mode is active: the previous values of $Q$ and $\bar{Q}$ are maintained
- $R$ and $S$ both being `1` is disallowed by design

[TODO: schematic here, 10.3]: #
A **clock** is a binary signal that switches between `0` and `1` each fixed cycle time. The time it takes from turning to one state then returning to it again is known as the *clock cycle time*. This is measured in seconds. Clock frequency can be obtained by taking the reciprocal of the clock period.

A finite-state machine is a form of sequential logic circuit. It has a combinational part that performs the computations. The inputs to the combinational portion are external and also the current state. The outputs are also external and constructs the next state. This "next state" is then looped around to become the "current state" and the cycle repeats. 

FSMs can be categorized. *Acceptors* produce a binary output that indicates whether or not the input is valid. *Transducers* performs calculations based on input and/or state to produce an output. Furthermore, if a transducer only refers to state, it is a *Moore machine*. If input is also needed, it is a *Mealy machine*.

## Pipelining
[TODO: probably talk about calculating clock cycle time and frequency and its units]: #
*Propagation delay* is the amount of time it takes for a signal to flow through a circuit. This information is the basis of two measures of performance:
- **Latency**: The amount of time it takes for the "bigger picture" to finish. The clock cycle time has to be this at minimum.
- **Throughput**: Number of requests that can be completed per unit of time.

**Pipelining** aims to increase throughput (with a small increase in latency) with parallelism. The "bigger picture" is broken up into stages, evenly breaking up the propagation delay. If the pipeline is *synchronous*, every stage shares a clock. Into each stage is a *pipeline register* that feeds it critical information. Each stage generates data to put into the next stage's pipeline register. Modules can be used in multiple stages.

-# The bottleneck / slowest link principle presents itself in non-uniform partitioning; the slowest stage will restrict the clock cycle time. Remember, the clock cycle time decides when the request can move onto the next stage.

## Storage
Data storage is grouped based on cost, performance, density, and some risks. Persistence is whether data can be kept after being powered infinitely. Volatility is if the data remains after a power cycle. Resiliency is the number of operations before the hardware degrades.

Dynamic RAM is not persistent, volatile, and resilient. It is usually described in the form $d \times w$, where $d$ is the total number of supercells, and $w$ is the number of bits per supercell. When accessing a particular supercell, the memory controller will send a row address strobe to the DRAM chip. The DRAM chip will put this row into the internal row buffer, and will send the data when the memory controller sends the column address strobe. The memory controller can optimize by remembering whether or not the row is in the internal row buffer, and just pull from the buffer if so.

-# Quantities are always of the -bibyte variant for DRAM.

Static RAM is a special kind of DRAM. To be SRAM, memory must be partitioned into banks and have a pipeline going through each bank. This allows for concurrency. However, this means that addressing a supercell requires extra bits to distinguish the bank. SRAM is persistent, volatile, and resilient. SRAM lives on the CPU for its speed, but it is expensive.

DIMMs are comprised of several DRAM chips mounted on a printed circuit board. This is what's commonly associated with "memory." Each DRAM chip does not hold "contiguous" memory; every $w$ bits are put into the next DRAM chip. This is because each DRAM chip contributes to part of the word residing in the memory controller. All DRAM chips are addressed using the same row and column addresses.

Hard drives are comprised of platters stacked on top of each other. Both sides of a platter can each serve as a surface. On each surface are many, many concentric tracks. Each track is divided into sectors separated by gaps. A sector is the smallest amount that can be read or written to. A cylinder are the tracks that are vertically stacked. The outermost cylinder is 0. The head is what does the reading and writing. It is attached to the end of an arm off to the side. It will locate the desired track and wait for the sector to rotate around. All this, of course, takes time. 

-# Zone bit recording is a practice of grouping tracks together to have the same number of sectors. This has the consequence of throughput being better on outer cylinders than inner.

Errors can occur in data during storage or transmission. To counter a single bit flip, hamming codes are used. `XOR` a particular subset of data bits $m$ times. These parity bits are placed in positions that are powers of 2 under 1-indexing. Data bits have to work around these parity bits. This bit string is then sent, and the receiver has to recompute the `XOR`. To find where the bit flip is, `XOR` the original and recomputed parity bit strings. This syndrome vector gives the position, after the hamming code portion begins, under 1-indexing (0 means no error). 

-# The notation is $\mathnormal{Hamming}(2^m - 1, 2^m - m - 1)$, where $m$ is the number of parity bits.

In reality, access times of storage lag way behind the CPU (clock cycle time). To mitigate this, the **Principle of Locality** is used as guidance. Temporal locality notices that recently-referenced items are likely to be referenced again soon. This is typically handled by the compiler. Spatial locality recognizes that nearby items will likely be referenced close in time. **Caches** at the hardware level handle this kind of locality.

These two facts lead to a hierarchical view of storage. The top is fast but can hold few things, while the bottom is slow but sizable. With this in mind, a cache $\$$ serves as a staging area between levels. In effect, improving speed. A cache functions like a hash set with chaining without resizing, with the hash function being the middle bits of the address. The indexes link to a cache set out of $\mathbb{S}$, of which comprises of cache ways. Each cache way consists of a cache line of size $B$, valid bit, and the block's tag. Cache associativity $A$ is the number of ways per cache set. This leads us to the *ABC model of cache structure*, $C = \mathbb{S} \times A \times B$. This can be used to find the cache capacity $C$ or other variables about the cache.

-# In this class, memory on the CPU is considered fast, while elsewhere is slow.

There are a couple cache geometries. In general, caches are set-associative. However, if the cache has a cache associativity of 1, it is a direct-mapped cache. Contrarily, if there is only one cache set, the cache is fully-associative.

When data from a lower level is requested by a higher level, the cache is checked first. When indexed into the cache set, the upper bits of the requested address are used to compare to the tag associated with each cache line. If the valid bit is asserted and the tags are the same, that is a cache hit. If it's a cache miss, it is retrieved and copied from storage. If the valid bit for a cache line is `0`, the block is placed in the cache line and the valid bit becomes a `1`. Alternatively, if all cache lines are valid, a cache line must be replaced. Cache misses are categorized by the 3C model:
- Compulsory: first access to the memory block
- Capacity: the non-compulsory miss is a miss in an equivalent fully-associative cache
- Conflict: the non-compulsory miss is a hit in an equivalent fully-associative cache

Try to avoid cache misses if possible; the entire procedure for a cache miss takes very long compared to that of a cache hit. That is why code must exhibit good spatial locality as well. To assess the spatial locality of a program, analyze the *access stride* of each memory structure (e.g., array). Access stride measures the amount of elements being skipped over each access.