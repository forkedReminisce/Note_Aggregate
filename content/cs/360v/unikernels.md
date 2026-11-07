---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Unikernels don't have an OS. Instead of making system calls, call functions of the *library OS*. This library OS is linked with a single compiled application into an image. This allows a unikernel to have better performance and memory usage over containers running in a micro-VM. Unikernels are just an example of specialization. However, there is the drawback of compatibility, or lack thereof. 

{{< subtext >}}
    Unikernel Linux is a middle-of-the-road compromise between containers in a VM and unikernels—better compatibility than unikernels but larger since it requires the entire Linux library OS.
{{< /subtext >}}

A unikernel needs to be modular. As a counterexample, Linux's subsystems are all dependent on each other, and a change to one will affect all. To achieve modularity, Unikraft started from scratch. glibc was replaced with nolibc. nolibc interfaces with a number of libraries like ukalloc. These implement the C library functions like `malloc()` in a way more suitable to unikernels.

If one is willing to modify their program to specialize for unikernels, they can make the full use of the high-performance API. 