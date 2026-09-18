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
On a memory access, the hypervisor is in control. If there is page fault, it gets injected to the guest OS. However, the hypervisor made the guest OS' page tables read-only, so when it tries to install the page table entry, a write violation trap is triggered. The hypervisor then retakes control, allocates memory for both the host OS and guest OS, and updates both page tables (to keep them in sync). The guest OS is then back in control, retries the instruction, and succeeds. This is incredibly slow, but it only affects the page fault; hits in the cache or page table is the same as a host application. However, double the memory gets used.

<!-- EPT: extended page table -->
What if we map guest physical addresses to host physical addresses? Now host page tables are per VM rather than per guest process. Let `CR3` point to the guest's page table, and create an `EPT` field in the VMCS that points to the host's page table. The `TLB` is still guest virtual address to host physical address. If hardware doesn't have a mapping, it sends the page fault directly to the guest OS. The guest OS can do its stuff without triggering a write violation. Now that the guest virtual address has been mapped to a guest physical address, the hardware regains control. Since there is no page table entry for the guest physical address, hardware sends an EPT violation to the hypervisor for it to install the mapping to host physical address. However, the cost of a TLB miss is higher than shadow paging, and, therefore, without virtualization.

What if the sum of expected memory of every VM exceeds the physical memory capacity? That's overcommitment, and this often happens because it's expected all VMs will not use all their memory at the same time. But we have SWAP so it's fine. There is a semantic gap in that the guest OS knows which pages are OK to swap out, but the hypervisor does not. In any case, a lack of communication leads to independent swapping without coordination. The guest OS doesn't actually swap, but it thinks it does nonetheless. The hypervisor swaps random pages. This can lead to double swapping. 

<!-- is double swapping avoided because the guest OS will not be trying to free, so the hypervisor is now free to do whatever it wants? -->
Install the Balloon kernel Device Driver onto the VM. This driver uses the io channel to reach the hypervisor directly. This driver will request for a substantial amount of memory from the guest OS when the VM is using too much memory. This requires the guest OS to initiate memory acclamation because its a device driver—it's higher priority. The OS will be swapping pages it doesn't see as valuable. This is communicated to the hypervisor. The balloon inflates. The guest physical addresses for the balloon is not backed by host physical memory. This reclaims memory from the VM for the hypervisor. Then the balloon deflates and returns memory to the VM. However, the driver is not portable since it needs to be developed for each OS and update for each OS update. 

Copy-on-write will map the same host physical page between multiple VMs if their pages are the exact same. However, this only applies for read. For write, a new page is allocated for the VM performing the write. However, comparing pages between VMs is extensive. Therefore, a shared pages data structure is used. A hash function takes a page's data, which returns the index. Check if the index is in the data structure.