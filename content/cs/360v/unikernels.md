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

If one is willing to modify their program to specialize for unikernels, they can make the full use of the high-performance API. 

<!-- both rumprun and unikernel linux support posix applications -->
<!-- rumprun is not portable; only supports a specific type of kernel -->
RumpRun makes use of a concept known as anykernel. If the kernel is modular, the Rump Runtime can just pull the components it needs without having to rewrite and maintain it. That way, the kernel is essentially in the application namespace. However, this does leave specialization on the table that cannot be achieved. 

Unikernel Linux compiled the kernel with the application. System calls were just functions, so there was no context switch. However, this did not offer any meaningful performance boost.