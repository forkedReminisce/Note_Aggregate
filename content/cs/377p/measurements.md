---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



To understand and improve program performance, one will need insight into program behavior. This include execution time and how well it exploits hardware resources. 

There is a difference between precision and accuracy. Precision is how close measurements are to each other. Random error can be removed by taking the mean of a bunch of samples. Accuracy is how close the measurement is to the actual quantity. Systematic error can undermine this. 

Timing code is not as trivial as one might think. The issues are:
- Initial conditions matter: machine state (e.g., caches) when timing starts
- Resolution and precision of timer: granularity and random error
- Heisenberg effect: actions relating to taking measurement may affect quantity
- Compiler optimizations
- Context-switching: measuring stuff other than program
- Out-of-order execution

Empty the cache (cache-cleanup) before measurement. This is basically manually filling it with garbage. Solving resolution requires a loop and an extra empty loop. However, because of compiler optimizations, looking at the assembly to make sure the loop wasn't optimized out is required. Context-switching can be handled by disabling them (largely impossible) or using a timer that advances only when the program is executing. OOO executions use serializing instructions around the interested code.

{{< subtext >}}
    When handling compiler optimizations, make sure not to disable too many optimizations or else the measurements won't be realistic.

    The solution to context-switching can still pollute the cache.
{{< /subtext >}}

Modern CPUs can track many events (e.g., cycles, l-cache misses, etc.). However, there are many complications. Not all events can be tracked at once, accessing them is not easy, and it isn't portable across architectures. Performance Application Programming Interface (PAPI) is the solution.