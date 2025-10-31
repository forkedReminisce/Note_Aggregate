---
draft: false
title: History of OS's

params: 
    desc: Throughout time, OS designers have been innovating ways to maximize utilization of the computer.
    author: FREEZURN 
---



Decades ago, applications were physical and had to be inserted into the computer to run, and only that application would execute at a time. ==Utilization was poor== as there was this human element and also I/O (reading and printing) took a while. Therefore, **batch processing** streamlined the process. Separate machines handle the I/O, and input, specifically, would coalesce a list of tasks so the main machine can execute each one directly one after another.

When a program requests I/O, **multiprogramming** allows the another program to run. This will happen when the OS follows the request with the *I/O interrupt*. The OS will request the I/O device and wait, all the while another program is executing. When the I/O device sends back an interrupt, the OS will call the scheduler to pick either the original job or newer job according to the scheduling policy.

<!-- TODO: is timesharing related to multiple users and timer interrupt multiple applications -->
A minicomputer allow for multiple users at once. To create the *illusion* that a user feels that they are the sole user, **timesharing** was conceived. After a set period of time, a *timer interrupt* will occur: the scheduler is called and a context switch is performed.

{{< subtext >}}
    Context switching is handled by the OS; the program does not need its own code to handle it.
{{< /subtext >}}