---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Unikernels are their own VM. They consist of only one application with a single address space. However, there is no guest OS. So when the unikernel needs to do a privileged instruction, it makes a call to a function of the library OS. In fact, the unikernel is stripped down to only the bare essentials.

{{< subtext >}}
    The library OS is compiled with the application into the image.
{{< /subtext >}}

<!-- better density, performance, memory -->
For security, containers have been deployed in a micro-VM. However, this eats at the benefit containers have over VMs. Unikernels embrace and specialize in this. The improvement comes from less demands from the library OS over even a mini guest OS. However, unikernels do suffer from compatibility issues; containers will work almost always. Unikernel Linux improves on compatibility (not as much as containers), but requires shipping the entire Linux library OS.

<!-- ukboot??? nolibc interfaces with a bunch of libraries like ukmalloc??? -->
A unikernel needs to be modular. As a counterexample, Linux's subsystems are all dependent on each other, and a change to one will affect all. Unikraft delivers this by doing everything from scratch. For example, instead of glibc, Unikraft made its own posix-layer called nolibc. The program can and should still be coded as though it isn't intended for a unikernel (i.e., as normal), but the underlying implementation of the functions like `malloc`.

If one is willing to modify their program to specialize for unikernels, they can make the full use of the high-performance API. 