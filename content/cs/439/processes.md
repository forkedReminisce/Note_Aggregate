---
draft: false
title: Processes

params: 
    desc: A process is an instance of an executing program. The OS oversees them.
    author: FREEZURN 
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