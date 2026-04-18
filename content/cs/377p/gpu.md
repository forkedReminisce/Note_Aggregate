---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The CPU is connected to host memory while the GPU has its global memory. The host and device are connected via the PCIe Bus. One of the functions facilitated by this bus is the transfer of data between these two memories.

CPUs possess latency-oriented design (latency avoidance). On the other hand, GPUs have throughput-oriented design (latency tolerance). GPUs have a lot of threads running in program order. This also eliminates the need of context switching because each thread owns its registers and other context information thanks to the large register file.

<!-- TODO: may delete -->
CUDA programs consist of a host program and kernel programs. The host program contains `main()` and invokes kernel programs. There are special dynamic memory functions for global memory that the GPU and CPU can use. `cudaMemcpy()` allows transfer between host and global memory. Interestingly, kernel calls are non-blocking, but `cudaMemcpy()` is blocking. It begins once all kernel threads terminate.

<!-- streaming processor has 32 CUDA cores that support warps??? -->
Warps basically group threads of similar purpose so that a number of them will execute at the same time (i.e., co-scheduling). Scheduling is done at the level of warps, only getting scheduled when all member threads are ready. Let's just place 32 threads into a warp based on sequential order of thread id. Each thread in a warp share a PC so they execute instructions in lock-step. 

<!-- TODO: memory access are not like scatter/gather, but pthreads implementations might make it like that. minimizing bus transactions (coalesce into one)? -->
At each step, they're either executing or turned off (like predicated execution). This idea is known as thread divergence, and it would be best for performance if all threads of the warp aren't turned off by arranging threads to go to particular warps. Memory accesses is very much like scatter/gather, so it would be best to achieve spatial locality. Hardware performs coalesced memory access by figuring out which lines need to be read. Unlike the CPU implementation of parallel reduction with divide-and-conquer, the GPU needs to work around these concerns.

{{< subtext >}}
    Coalesced memory access avoids buffer nonsense.
{{< /subtext >}}

<!-- on coalesced memory access, is shared-memory access back-to-back or simultaneous? multiple wires to? -->
Similar but separate to warps are thread blocks. These can hold up to 1024 threads, and which threads to into it is defined by the programmer. Co-scheduling is still a thing, and a block only leaves once all its threads complete. Each thread block has its own shared memory that cannot be accessed by threads outside the block. The data is stored in the block's own cache, which greatly improves bandwidth and latency. Shared-memory is banked, and it is not worth trying to minimize traffic like it is to global memory (one transaction is expensive). Therefore, it is more important to avoid bank conflicts. Typically copying data from global memory to shared-memory is worth it for reuse, and this is done "manually" and likely cooperatively between threads in the block.

{{< subtext >}}
    Remember that warps work in conjunction with blocks.
{{< /subtext >}}

The hierarchy goes from threads to warps to blocks to grids. 