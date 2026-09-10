---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- on host OS -->
The page table is pointed to by the `CR3` register. The `CR2` points to the instruction that's accessing a (virtual) memory address.

<!-- change host OS to hypervisor -->
In a VM, the guest OS has "physical" memory of its own. Without hardware support for only memory, there is shadow paging. The guest OS has a page table that maps its processes' virtual addresses to its physical address, and the host OS copies this page table but maps it to the actual physical addresses. The VMCS has a separate for each guest process `CR3`. 

Memory access is initially handled by the host OS, but if there is a page fault, a page fault gets injected to the guest OS (to be handled by its interrupt vector stored in the VMCS). However, the guest OS cannot install the page table entry because the hypervisor made its page tables is read only. This causes a write violation trap to allow the hypervisor to retake control. The hypervisor allocates memory for both the host OS and guest OS and updates both page tables. The guest OS regains control, retries the instruction, and hardware can actually find the mapping to host physical memory. ==This is incredibly slow==.