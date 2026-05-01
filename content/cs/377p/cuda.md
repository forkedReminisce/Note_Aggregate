---
draft: false
title: CUDA Programming

params: 
    desc: An overview of CUDA programming, specifically thread hierarchies.
    author: Andrew Nguyen 
---



The CPU is connected to host memory while the GPU has its global memory. The host and device are connected via the PCIe Bus. One of the functions facilitated by this bus is the transfer of data between these two memories.

CPUs possess latency-oriented design (latency avoidance). On the other hand, GPUs have throughput-oriented design (latency tolerance). GPUs have a lot of threads running in program order. This also eliminates the need of context switching because each thread owns its registers and other context information thanks to the large register file.

# {{< heading "Hierarchy" >}}
**Warps** group 32 threads by their order of creation. By *co-scheduling*, all the member threads execute when all are ready. Additionally, they share a PC so that they execute instructions in lock-step. Warps tend to suffer from thread divergence, where some threads would be turned off while others stay on. That is why it is common to strategically create threads so that they get assigned to warps that minimize this issue. Hardware also performs coalesced memory access to avoid having to go to global memory for each thread, minimizing bus transactions. 

A level above are **thread blocks**. Threads are manually assigned to one by the programmer, and they can hold up to 1024 threads. Co-scheduling is still a thing, but not necessarily lock-step execution. Each thread block has its own shared memory that can only be accessed by its member threads. Data must be manually brought in from global memory, and it is often useful to split the work amongst member threads. Since shared-memory is literally a cache, it is not worth trying to minimize traffic like it is for global memory and coalesced memory access. Instead, since it is banked, avoid bank conflicts.