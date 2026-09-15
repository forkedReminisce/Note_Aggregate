---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



# {{< heading "Scheduling" >}}
The guest OS has virtual CPUs that it schedules (virtual) threads onto. The hypervisor schedules these virtual CPUs onto physical CPUs. However, since it doesn't schedule the threads, there is an information gap between the guest OS and hypervisor. Synchronization becomes cumbersome because the hypervisor can't tell a thread is holding a lock. Locality is hard as the hypervisor has no idea a thread was on a particular physical CPU and, therefore, was using its cache.

Scheduling strategies can help mitigate these shortfalls. The objectives are utilization, fairness, and progress. Utilization is work conservation: every virtual CPU gets the chance to run. 
- Gang scheduling: run strictly all of a VM's virtual CPUs at once 
- Independent scheduling: per virtual CPU
- Relaxed co-scheduling: try to balance each virtual CPUs' run time of a given VM, using gang scheduling if it gets too imbalanced

<!-- Para virtualization can optimize scheduling by minimizing the information gap. There is communication between the guest OSs and hypervisor about the state of threads and such. -->



# {{< heading "Memory" >}}
A VM has "physical" memory. This, though, doesn't get used besides keeping the guest OS happy. The guest OS does have page tables that map application virtual addresses to this fake physical memory, but, under shadow paging, the hypervisor maps these same application virtual addresses to actual physical memory. It's the hypervisor's page tables that's actually used.

{{< subtext >}}
    The VM does have its own `CR3` separate from the actual, and it iis stored in the VMCS. 
{{< /subtext >}}

<!-- is host OS initially in control, guest OS triggers write violation, and then hypervisor gets control? -->
On a memory access, the hypervisor is in control. If there is page fault, it gets injected to the guest OS. However, the hypervisor made the guest OS' page tables read-only, so when it tries to install the page table entry, a write violation trap is triggered. The hypervisor then retakes control, allocates memory for both the host OS and guest OS, and updates both page tables. The guest OS is then back in control, retries the instruction, and succeeds. This is incredibly slow.