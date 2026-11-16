---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- unikernels are bootable VM images that run ontop of a hypervisor -->
Unikernels don't have an OS. Instead of making system calls, call functions of the *library OS*. This library OS is linked with a single compiled application into an image. This allows a unikernel to have better performance and memory usage over containers running in a micro-VM. Unikernels are just an example of specialization. However, there is the drawback of compatibility, or lack thereof. 

{{< subtext >}}
    Unikernel Linux is a middle-of-the-road compromise between containers in a VM and unikernels—better compatibility than unikernels but larger since it requires the entire Linux library OS.
{{< /subtext >}}

A unikernel needs to be modular. As a counterexample, Linux's subsystems are all dependent on each other, and a change to one will affect all. To achieve modularity, Unikraft started from scratch. glibc was replaced with nolibc. nolibc interfaces with a number of libraries like ukalloc. These implement the C library functions like `malloc()` in a way more suitable to unikernels.

Compare this to MirageOS, where both the components and app must be written in the same language: OCaml. This obviously has poor compatibility. RumpRun has marginally better compatibility. If the kernel is modular, the Rump Runtime will simply takes the components it needs. Unikernel Linux is better in that it compiles the entire Linux kernel with the application. All these are a tradeoff in compatibility for less maintainer overhead.