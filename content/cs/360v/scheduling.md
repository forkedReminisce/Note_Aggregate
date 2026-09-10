---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



There may be more virtual CPUs at play than physical CPUs. Therefore, the hypervisor schedules by vCPU instead of threads. The (virtual) threads are scheduled by the guest OS onto the vCPU. 

VMs struggle with synchronization and locality because of the information gap from the hypervisor. It does not know what threads are holding locks so that they don't get preempted. It does not know which physical CPU a thread was on in order to continue keep the cache.

<!-- utilization, work conservation: every vCPU is able to get the chance to run -->
The goals of scheduling is utilization, fairness, and progress. Gang scheduling schedules a VM's vCPUs all together at the same time. However, if there are more total vCPUs than physical CPUs, a VM might starve. Independent scheduling schedules per vCPU. Relaxed co-scheduling tries to make sure each vCPU on a VM run for the same amount of time. If there's a great imbalance, gang scheduling is employed for that VM.

Para virtualization can optimize scheduling by minimizing the information gap. There is communication between the guest OSs and hypervisor about the state of threads and such.