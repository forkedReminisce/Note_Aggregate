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

<!-- the actual TLB is guest virtual address to host physical address -->
<!-- TODO: hardware is traversing the (shadow) page table. hypervisor gets page fault then injects page fault to guest OS. this inject is necessary for the hypervisor to learn the guest physical address, which it can then back with a host physical address. -->
On a memory access, the hypervisor is in control. If there is page fault, it gets injected to the guest OS. However, the hypervisor made the guest OS' page tables read-only, so when it tries to install the page table entry, a write violation trap is triggered. The hypervisor then retakes control, allocates memory for both the host OS and guest OS, and updates both page tables (to keep them in sync). The guest OS is then back in control, retries the instruction, and succeeds. Shadow paging is slow, but only during a page fault; otherwise, it is as fast as without virtualization. Double the memory is consumed, though.

What if, instead of mapping guest virtual addresses, the host page tables map guest physical addresses to host physical addresses? Let `CR3` point to the guest's page table and create an `EPT` (extended page table) field in the VMCS that points to the host's page table. The `TLB` is unchanged, mapping guest virtual address to host physical address. If hardware can't find a mapping, it sends the page fault directly to the guest OS. The guest OS will install the page table entry without triggering a write violation. Hardware will regain control but there is still no page table entry for the guest physical address, so hardware sends an EPT violation to the hypervisor to resolve it. Compared to shadow paging, EPT is slower on TLB misses but faster on page faults. It also uses less memory.

Especially when there are multiple VMs running at once, total virtual memory capacity will exceed physical memory capacity. Overcommitment is when the hypervisor allows this to happen, and it often gets away with it because every VM wouldn't possibly demand all their memory at the same time. Even if they do, the SWAP is a reliable fallback. 

<!-- does the balloon tell the hypervisor what pages the guest OS evicts? or is it a way of communicating to the guest OS that the hypervisor is taking away resources? -->
There is a semantic gap between the hypervisor evicting random pages but the guest OS using a strategy to select a page to swap out. This can lead to events like double swapping. By installing the balloon kernel device driver onto the VM, the hypervisor can shrink this gap. When the hypervisor finds it necessary, it can "inflate" the balloon, meaning request more memory. Since it's higher priority as a device driver, the guest OS has to initiate memory acclamation from other processes. Additionally, the balloon itself is not backed by host physical memory, which allows the hypervisor to reclaim memory from the VM. When the situation cools down, the hypervisor will "deflate" the balloon and return the memory to the VM. The downside of the balloon is that it is not portable, meaning it needs to be developed for each OS.

{{< subtext >}}
    The guest OS doesn't actually evict pages.
{{< /subtext >}}

Copy-on-write will map to the same host physical page for multiple VMs if its content is the same. When one VM wants to write to it, however, it gets remapped to a copy of the page. Comparing each page is expensive, though. Therefore, a shared pages data structure is used: a hash function takes a page's data and returns an index and checks if the index is already in the data structure.