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



# {{< heading "I/O" >}}
Memory mapped I/O (MMIO) are ranges of physical addresses that are not backed by RAM but device registers. The device sends an interrupt when it finishes with an operation, and DMA skips some intermediate steps. All these components need to work not only on the host machine but also on the VM.

Device emulation is one approach on the VM. For the guest OS, the MMIO region is read/write protected. This means every operation triggers a trap to the hypervisor. The hypervisor must then spoof the operation and DMA. This is obviously expensive, but interrupts not so much; the hypervisor just changes the VMCS state and let the guest OS handle the interrupt. 

Alternatively, under device passthrough, the host OS hands off control of the device exclusively to one VM. This eliminates trapping, but the device can no longer be shared with other VMs. Interrupts are forwarded by the hypervisor. DMA writes to host physical addresses mapped from guest physical addresses. The IOMMU holds the translations in a data structure, and there is an IO TLB. 

<!-- so a device has memory on the device itself on top of the MMIO on the hardware? -->
<!-- interrupt demapping? remapping? -->
These solutions are without specific hardware support. Single-Root I/O Virtualization (SR-IOV) basically unlocks sharing for device passthrough. A device manufacturer allows their device to partition its resources as necessary to create virtual functions. Its these virtual functions that the VM receives. The virtual function may also be able to short-circuit—send the interrupt directly to the guest OS.


## {{< heading "Xen Paravirtualization" >}}
Xen Paravirtualization brings many optimizations to I/O. First, there is the hypercall that the OS uses to invoke the hypervisor. Although the OS needs to be modified to support hypercalls, they do allow for batching. The main benefit of batching is that it reduces the number of traps.

<!-- Xen sends the signal through the event channel, not backend? -->
Each domain has the simple frontends for device drivers. Domain 0 specially contains the only backend that actually interacts with the hardware. Grant tables allow for pages to be shared between domains. These shared pages are used to transfer data between the frontend and backend. Since the backend receives the interrupt, event channels allows the frontend to also receive the interrupt. Based on the signal, the frontend calls a particular upcall handler.

Virtio device drivers install into guest OSs knowing they're in a VM. They serve a purpose that helps the overall host system. The frontend lives in the VM and it interacts with the virtqueue. This virtqueue is handled by the backend.

The hypervisor is kept as small as possible. Since domain 0 as to make hypercalls, the hypervisor still gets the final say. So if domain 0 gets compromised, it's not the end of the world.



## {{< heading "GPU" >}}
Direct assignment is just like device passthrough. When the GPU uses DMA or sends an interrupt, the Virtual Function I/O (VFIO) reroutes it to the correct VM. The VFIO is free to modify the IOMMU, interrupts, and page table. 

Mediated passthrough is when the hypervisor handles the IOMMU and interrupts. Additionally, only a fraction of the cores are made available to the VM and it's time-sliced.

Something akin to SR-IOV is static spatial sharing of the GPU. Under NVIDIA Multi-GPU (MIG), the GPU is divided up into GPU regions that each can be allocated to VMs. However, it's not possible to change the division configuration at a fine level. That is not to say that SR-IOV is possible, though, and the VFIO will be necessary.



# {{< heading "File Systems" >}}
When the VM accesses a file, the address it generates needs to be translated for the host machine. 

Modifying metadata and/or data requires atomicity and, therefore, journaling. However, a case of I/O amplification can occur through something known as double journaling. This is when the VM journals its changes, commits, then actually makes the changes. The hypervisor then journals the journal then writes the VM's journal and its modifications.

<!-- files copied from host VM? -->
Most VMs have copy-on-write file systems. When a file is copied, only the metadata (e.g., inodes) is copied. When a data block is written to, a copy of the block is made and the respective pointer in the copied metadata is changed. 

Snapshots saves the file state for the purposes of restoring it at a later time. This relies on copy-on-write. The copied metadata replaces the original metadata, and the original metadata serves as the snapshot. 

Taking a snapshot of the entire VM requires copying CPU state, memory, device, and storage state. It is desirable for the VM to be frozen during a snapshot, so the EPT is write-protected. If the VM tries to access a page, the VM is frozen. Snapshots of the VM is great for migration.

Virtual storage is backed by the host machine's storage. The hypervisor can provision the entire amount the VM wants, but that's slow. Instead, thin provisioning allocates a fraction of the whole amount. The VM's file system must support holes—unallocated gaps between allocated space.