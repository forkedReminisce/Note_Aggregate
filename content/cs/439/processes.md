---
draft: false
title: Processes

params: 
    desc: A process is an instance of an executing program. The OS oversees them.
    author: Andrew Nguyen 
---



A **process** is an instance of an executing program—the binary. It is what the scheduler picks. It receives an *address space*, the range of memory the process can legally access. This is how programs are isolated from one another. An address space consists of: stack, heap, Static Data Segment (globals and static variables), and code.

The OS tracks every process' state with Process Control Blocks (PCB). They are allocated in kernel memory for the lifetime of its process. In a PCB are pointers to the address space, register contents, open file table (open files used by the process), process id (PID), and *process execution state*. State describes where in the lifecycle the process is in:
1. New: the process has just been executed, and the OS needs to set it up
2. Ready: waiting to be selected by the scheduler
3. Running: could go back to ready because of time-sharing
4. Blocked: waiting for an external event (e.g., I/O)
5. Terminated: finished; the OS needs to deallocate the resources


## {{< heading "Dual Mode Execution" >}}
Processes need to have restricted privileges in order to protect the system. However, it must not be so restrictive as to hinder utilization or communication. Therefore, we have two possible execution modes: ==user and kernel==. In kernel mode, a process can do anything, in the perspective of the OS. In user mode, a process is prohibited from:
- Accessing I/O directly
- Manipulating OS memory (e.g., page tables)
- Setting the mode bit (for execution mode)
- Disabling or enabling interrupts
- Halting the machine

A process can enter the kernel in three ways: exceptions, interrupts, or system calls. Hardware will use the *interrupt vector* to pick the appropriate handler. Think of it as getting a function from a map. Then the handler will be executed by the OS in kernel mode. To get back to the process' original position, the *exception stack* is used. Think the process' stack.

{{< subtext >}}
    Signals don't have a key in the interrupt vector. Instead, the OS forwards it to the process, who has already defined a signal handler for it. The process executes the process handler.
{{< /subtext >}}



<!-- TODO: -->
# {{< heading "Deadlock: Revisited" >}}
There will be a slight modification from the conditions of deadlock for threads: mutual exclusion is loosened up to become *bounded resources*: resources are finite. As a result, the four conditions become necessary ==but not sufficient==. This means that if all four conditions are present, it must be investigated to truly determine deadlock.

So far, deadlock has been mitigated through *deadlock prevention*—breaking one of the four conditions. In the OS, this would be **resource ordering**. However, this may force a process to grab a resource earlier than it needs it, tying it up for longer than necessary.

*Deadlock avoidance* relies on an active algorithm that checks resources requests for any potential deadlock. It also ends up breaking a condition. **Banker's Algorithm** will accept more resource requests than is actually available. It achieves this by staying in a safe state—that there exists a schedule where the maximum possible request of each process can be eventually fulfilled. If a resource request leads to an unsafe state (but not necessarily deadlock), the request is denied. This algorithm is weak, though, because it's like predicting the future, cannot handle additional requests, nor can it handle resource failures.

*Deadlock detection* will allow deadlock to occur and try to recover from it. **Resource Allocation Graphs** model requests and allocations of resources and processes. If there's a cycle, there might be deadlock. The deadlock is broken by killing processes of the cycle or pre-empting resources. However, detecting cycles is very expensive.

In reality, though, deadlock doesn't really happen all too often. This results in the Ostrich Algorithm—ignoring it.