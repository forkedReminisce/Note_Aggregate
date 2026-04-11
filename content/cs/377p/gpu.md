---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The CPU is connected to host memory while the GPU has its global memory. The host and device are connected via the PCIe Bus. One of the functions facilitated by this bus is the transfer of data between these two memories.

<!-- TODO: how is context switching eliminated -->
CPUs possess latency-oriented design (latency avoidance). On the other hand, GPUs have throughput-oriented design (latency tolerance). GPUs have a lot of threads running in program order. This also eliminates the need of context switching.

CUDA programs consist of a host program and kernel programs. The host program contains `main()` and invokes kernel programs. Code that runs on the GPU needs to be denoted with decorators. There are special dynamic memory functions for global memory that the GPU and CPU can use. `cudaMemcpy()` allows transfer between host and global memory. Interestingly, kernel calls are non-blocking, but `cudaMemcpy()` is blocking. It begins once all kernel threads terminate.