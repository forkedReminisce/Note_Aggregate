---
draft: false
title: Taking Measurement

params: 
    desc: Whether it's to measure execution time or events, there are easy options available. However, that's not to say they don't come with compromises.
    author: Andrew Nguyen 
---



Before improving program performance, it is necessary to understand its current behavior.

One potential observation is execution time. However, there are some challenges:
- Initial conditions matter: machine state (e.g., caches) when timing starts
- Resolution and precision of timer
- Heisenberg effect: actions relating to taking measurement may affect time
- Compiler optimizations
- Context-switching: measuring stuff other than program
- Out-of-order execution

There are ways to address these issues. Before taking measurement, the cache can be cleaned up with code that loads it with garbage. The resolution problem can be handled by executing the code multiple times in a loop, but that introduces the Heisenberg effect. Therefore, take a separate measurement of an equivalent empty loop. This loop might be optimized away, so check the assembly code. 

Disabling context-switching is pretty much not an option, so use a clock that advances only when the program is executing. Regardless, though, is that the cache will get polluted by other processes. OOO executions can be addressed with serializing instructions.

Modern CPUs can track many events (e.g., cycles, cache misses, etc.). However, there are many complications: not all events can be tracked at once, accessing them is not easy, and it isn't portable across architectures. Performance Application Programming Interface (PAPI) solves these issues.