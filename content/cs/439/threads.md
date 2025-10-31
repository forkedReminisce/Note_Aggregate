---
draft: false
title: Threads Of Control

params: 
    desc: Threads are the executions of a process. Since they share the process' address space, there are some important synchronization ideas to grasp. 
    author: FREEZURN 
---



A thread, for short, is an abstract entity of a process that executes a subset of the code. Every process has at least one: running `main()` and its function calls. It sounds like mini processes of the same program, but ==threads share the address space==—heap, SDS, and code. Notice that each thread gets its own stack. Additionally, a thread gets a Thread Control Block (TCB) of its register values, state, pointer to the PCB, interrupt state, etc.

{{< subtext >}}
    The OS doesn't protect against threads reaching into each others' stacks.

    Interrupt state is a bit saying whether interrupts are disabled or enabled.
{{< /subtext >}}

*User-level threads* are not tracked by the OS. This means that when one thread blocks, the process blocks too. They are created and managed via user-level libraries, and so the TCBs are stored in the process' heap. *Kernel-level threads*, on the other hand, are tracked by the OS. They're created via system calls, and TCBs are stored in the OS's heap. Although they cost more time, kernel threads keep a process busy and are the only way to split work across multiple processors.

{{< subtext >}}
    Threads are cheaper than processes ==(but still very costly)==. So too is the context switch between threads.
{{< /subtext >}}



# {{< heading "Synchronization" >}}
<!-- TODO: do "shared variable operations" include only reads? -->
Because there is shared data, it is possible for ==outcomes to be dependent on the order in which threads execute==. That is why it is important to be vigilant around **critical sections**—the shared variable operations. Race conditions can be identified with interleavings, permutations of statements between threads. 

A good defense against race conditions is one that is easy to follow, symmetric (scalable), no busy waiting, and the thread with a turn to access shared data must not be busy doing something unrelated. Therefore, the critical section must exhibit:
<!-- TODO: is bounded waiting about "no thread may jump over this one" -->
- Safety: *mutual exclusion* in the critical section
- Liveness: a thread wanting to enter the critical section must be guaranteed to eventually enter
- Bounded waiting: concrete number of threads that will enter the critical section before this thread
- *Failure atomicity*: no issue if a thread dies in the critical section

{{< subtext >}}
    *Mutual exclusion* describes an activity that only one process or thread may do at any given time.

    Failure atomicity is very difficult and often not present.
{{< /subtext >}}

The following proposals give the code something known as *logical atomicity*. This means a section of code will seem atomic—uninterruptible—by not allowing any other thread to touch the data and any related data to whatever this thread is operating on. It is important to have because there can be ==code that looks correct but can be messed up by out of order execution or compiler reordering==. Even a single assembly instruction cannot be trusted (e.g., `STR` is not atomic).

Disabling timer-interrupts (with system calls) is the most primitive way of achieving this. However, it doesn't disable them for every processor, requires that system calls not be used, and other interrupts (e.g., input) are also disabled. Another option are atomic read-modify-write instructions that read a bit while simultaneously unconditionally setting it (e.g., Test&Set). However, these instructions vary between architectures and are not to be relied upon for portability.

<!-- TODO: is low latency an advantage? -->
**Locks** are better than these options as they block a thread when another is already in the critical section. This is through `Lock::Acquire()`. `Lock::Release()` does the opposite operation, waking up any thread that's been blocked. A flaw of locks is priority inversion, meaning that higher priority threads can starve if the holder is lower priority.

{{< subtext >}}
    Locks actually use the primitive methods then adds onto it. Basically takes some of the work off the coder.

    Locks have two states: free and busy.
{{< /subtext >}}

**Semaphores** offer more versatility. `Semaphore::Down()` is analogous to `Lock::Acquire()`, as is `Semaphore::Up()` to `Lock::Release()`. Uniquely, a semaphore's state is measured with an integer that can represent the amount of a resource available. Binary semaphores can only be 0 or 1 and function like a lock, while counted semaphores can be a variety of numbers. This and other small differences make semaphores the preferred choice for the *rendezvous pattern*: synchronization in which the work of one thread is necessary to another thread.

<!-- TODO: monitors can suffer from deadlock, but how? why use a monitor over a semaphore? -->
However, semaphores have a lot of roles (mutual exclusion and rendezvous pattern), so they're typically only seen in the OS. For user code, there are **monitors**. Like Java classes, they encapsulate shared data and provide functions to operate on them. There will be one lock for the entire monitor, and this lock must be acquired at the start of every function and released at the end. For every synchronization point in the functions, there will be a new ==resource and condition variable pair==. 

A resource variable tracks the amount of a resource is available. *While* the resource variable is depleted, `CV::Wait()` should be called. This will release the common lock and block the thread. When the resource is replenished, the waiting thread wakes up, reacquires the lock, leaves the loop, and decrements the resource variable. On the supplier side, it will increment the resource variable before calling `CV::Signal()` to wake up a single thread, or `CV::Broadcast()` to wake up all. 

{{< subtext >}}
    When a waiting thread releases their lock, any invariants (anything it thinks its true at this point) must hold when it reacquires it.
{{< /subtext >}}



## {{< heading "Deadlock" >}}
Deadlock is when all threads never make progress as each thread prevents another from doing work. These following conditions are necessary and sufficient for deadlock:
- Mutual exclusion
- Hold & wait: one thread holds onto the resource while another waits for it
- No pre-emption: no external force is able to forcefully release a thread's resource
- Circular wait: for this thread waits on this other thread, which needs another thread, etc.

<!-- TODO: is lock ordering correct? resource partitioning? -->
Deadlock disappears if one of these conditions are broken. We commonly break circular wait with lock ordering. This requires that locks be acquired the same way across all functions. Alternatively, monitors break hold & wait because of condition variables releasing the lock.



# {{< heading "Serializability" >}}
*Coarse-grained locking* encloses the entire critical section in one big lock. It's simple but restricts the performance boost from multi-threading. *Fine-grained locking* partitions the critical section and puts a lock on each part. To defend against oversights (e.g., deadlocks), *conservative two-phase locking* is used. In order for a thread to do work, it must acquire all its necessary locks. When it finishes, it releases all its locks. This provides **serializability**—ability to associate a series of sequential actions to a specific thread.

<!-- TODO: atomic (at once), serializable (as a unit), and durable? -->
*Transactions* detail the actions of a thread. Values in this transaction must be idempotent (explicit). They must be durable, meaning they are saved. The transaction will be written to the *write-ahead log* on a special place in disk then committed. If something goes wrong in the middle of writing, the changes are lost forever. *Writing-behind* is handled by another thread, and it puts the changes in the correct places. There is no special rollback operation if this fails; the thread can just restart the write.

{{< subtext >}}
    Logs can also fail if a transaction is interrupted before the commit (e.g., releasing the lock too early allows another thread to write to the log with its own transaction).
{{< /subtext >}}