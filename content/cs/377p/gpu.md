---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



The CPU is connected to host memory. It reaches the GPU through the PCIe Bus. The GPU has its global memory. Host and device memory can transfer between each other.

<!-- TODO: how is context switching eliminated -->
CPUs possess latency-oriented design (latency avoidance). On the other hand, GPUs have throughput-oriented design (latency tolerance). They have a lot of threads running in program order. This also eliminates the need of context switching.

CUDA programs consist of a host program and kernel programs. The host consists of `main` and just invokes kernel programs. The kernel program assigns work to threads. The kernel program needs to specific what needs to run on the CPU and what runs on the GPU. Transferring data from host and device memory is done explicitly. There are special dynamic memory functions for the GPU that even the CPU can use. 

Kernel calls are non-blocking, but `cudaMemcpy` is blocking. This function only begins when all kernel threads finish.