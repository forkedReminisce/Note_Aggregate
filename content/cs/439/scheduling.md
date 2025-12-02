---
draft: false
title: Scheduling Policies

params: 
    desc: Schedulers implement a scheduling policy to decide what process will get the CPU next.
    author: Andrew Nguyen 
---




The scheduler works to support the OS in its illusionist role. To pick a process, it will follow a **scheduling policy**. *Preemptive* schedulers will stop a process even if it isn't finished. *Non-preemptive* schedulers wait for the process to finish or get blocked before scheduling another process.

{{< subtext >}}
    The scheduler is not a process but actually a code segment that running processes will execute when its time.
{{< /subtext >}}



# {{< heading "Evaluating" >}}
Scheduling policies are evaluated on these criteria.
- CPU utilization: keep the CPU as busy as possible
- Throughput: number of processes that complete for a unit of time
- Turnaround time: total time it takes for a process to finish
- Response time: time between execution and getting a result
- Waiting time: total time a process sat in the ready queue

{{< subtext >}}
    Real CPU schedulers minimize response time by minimizing its variance. This is both predictable and fair.
{{< /subtext >}}

It must be noted that context switches are the #1 source of overhead for the scheduler. It has to store the executing process' register contents into the PCB, and take the next process' register values from its PCB.



# {{< heading "Policies" >}}
**First-In-First-Out** (FIFO) is a non-preemptive scheduler that executes jobs in the order it appears in the ready queue. **Round Robin** expands this by incorporating the timer-interrupt. When the executing process' time slice (or quantum) is met, the process is kicked off the CPU and sent to the end of the ready queue.

{{< subtext >}}
    A time slice should be long enough that the context switch is as long as 1% of it.
{{< /subtext >}}

**Shortest Job First** (SJF) will schedule the job with the least amount of work until it blocks or terminates. Can be non-preemptive or preemptive. Obvious questions are how to assess how much work a job is and would higher-demand jobs ever get the CPU.

{{< subtext >}}
    That latter flaw is a case of *starvation*: the process requests a resource (CPU) but never gets it.
{{< /subtext >}}

We will use the past to evaluate the future; a process generally behaves similarly to its previous runnings in its lifetime. **Multilevel Feedback Queues** uses this logic to assign processes priorities. There will now be multiple ready queues that serve each priority level. The scheduler will work through the highest priority ready queue until it runs out jobs, then it will go to the second-highest ready queue. When it's time to pre-empt the process, the scheduler will judge the process' behavior. If it was good, it gets sent back to high ready queues; bad behavior go to low ready queues. Time slices are different between queues, and they grow exponentially the lesser the priority is.