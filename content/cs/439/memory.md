---
draft: true
title: 

params: 
    desc: 
    author: FREEZURN 
---



Memory is viewed in two different ways. The **physical address space** is for what is loaded into memory. However, not everything in the process thinks is in memory is actually in this physical address space. In actuality, each process has its own **logical/virtual address space**. Any data that is in the physical address space is in the process' *segment*. 



# {{< heading "Relocation" >}}
In uniprogramming (one program running at a time to its termination), the program is always loaded at physical address `0x0`. This makes mapping virtual addresses to physical addresses easy. When the process finishes, the segment is reclaimed for the next program to take. 

For multiprogramming, we will want *transparency*: illusion that this process is the only process in memory, and where in physical memory its segment is should not matter. It's important to have safety, ensuring processes do not corrupt each others' or the OS' segments.

{{< subtext >}}
    OS is reserved the highest physical addresses.
{{< /subtext >}}

**Relocation** will load processes where they fit. If there's no space, the OS will wait. Of the process' segment, the lowest physical address is the *base address*. This allows the program's compiler to assume that the program will be loaded at `0x0`. 

The loader can correct this with *static relocation*. As in, it will go into the binary and add the base address to every address. However, this makes moving a process in memory impossible. *Dynamic relocation* is an alternative strategy that makes this possible. It will require two new registers from hardware: the *base register* with the process' base address and the *limit register* to hold the largest address of the process' logical address space. Then hardware will, in parallel, add the base register and check if the original address is below the limit register. Moving processes and segment growth can now use this architecture, albeit, tediously.

To figure out the most appropriate free block for the process, a memory allocation policy is used. 
- **First-fit**: lowest addressed adequate free block 
- **Best-fit**: smallest, adequate free block. Free list sorted by size
- **Worst-fit**: largest, adequate free block. Free list sorted by size. Allocation faster than best-fit

Fragmentation will be a major concern; *external fragmentation* is leftover free space after cutting up a free block, while *internal fragmentation* is unused memory inside an allocated block.

*Compaction* is a strategy to eliminate external fragmentation. When an incoming process needs some noncontiguous free blocks to be merged, programs' segments are moved elsewhere in order to coalesce those free blocks. Alternatively, work around external fragmentation with *swapping*. Some process is suspended, its segment is moved to the swap partition, and the incoming process (especially a high priority one) now has a segment. 



# {{< heading "Paging" >}}
Relocation is not great for a number of reasons:
- All of the process must fit in a contiguous chunk of physical memory
- Computationally expensive to allocate and move
- External fragmentation
- Hard to share memory between processes
- Only supports primitive security

Overlays was the first step. Programmers would manually partition a process' virtual address space and only load the parts that were needed at this particular time. These ideas can be seen in the current solution of **pages**, but the partitions are of equal size. The physical address space is similarly partitioned into **frames**, which pages slot into. To keep track of the process' segment, the PCB stores a **page table**. 

{{< subtext >}}
    A process' virtual address space is now independent of the physical address space

    Every process' virtual address space size is defined by the OS. 
{{< /subtext >}}

Virtual addresses are a $(p, o)$ pair bit string. The high-order $p$ bits represent the page number, and the following $o$ bits are the page offset. Physical addresses have the synonymous $(f, o)$ bit string for frame number and frame offset respectively. The MMU (Memory Control Unit) relies on this system for address translation. The $p$ bits index into the page table, which spits out $f$ bits to be prepended to the $o$ bits. The page table supports memory sharing by sharing frame numbers between different page tables. 

{{< subtext >}}
    The pointer to the current page table is stored in the *page table base register*.

    A system call enables memory sharing.
{{< /subtext >}}

A page table entry contains some other bits. Permissions. Resident bit reveals whether the page is currently in physical memory, and then the next bits (including $f$) can either be the frame number or something else (e.g., swap slot). Reference bit tells whether the page has been used recently. Dirty bit determines eviction behavior.

There are two policies to loading pages of a process for the first time. Demand paging loads a page upon reference. Pre-paging will see the OS predict and load pages the process will likely immediately need. If the process tries to access an unmapped virtual address, the OS will execute the **page fault handler**. This handler will figure figure out why its being called: unallocated data will cause an exception, while a nonresident page means true page fault. The latter requires bringing the page into physical memory. 

{{< subtext >}}
    Demand paging is more popular due to the principle of locality.
{{< /subtext >}}

However, there may not be a free frame. Therefore, page replacement algorithms are in order. The goals of such algorithms are to reduce the number of page faults and efficiency. The ideal solution is to replace the page that hasn't been referenced for the longest time. However, this is predicting the future. Least Recently Used (LRU) is another option, but its hard to implement efficiently. 

**Clock** is a viable compromise. There will be a circular linked list of all the pages in memory. On the first eviction, the pointer to this list start on the oldest page. It will then look for a page with a reference bit of `0`, meaning the page hasn't been accessed since the last time it was checked. If it's `1`, the reference bit is re-set to `0`. ==Either way, the hand is advanced.== The page that is replaced is typically put into swap, and the new page is typically retrieved from swap.

{{< subtext >}}
    When a page is replaced, the new page will get a reference bit of `1`.

    Hardware sets the reference bit to `1`.

    Pages of the code section are typically not seen in swap. They're pulled from the file system and, on eviction, its data in frame is simply discarded.
{{< /subtext >}}

Since it's cheaper to simply replace an unmodified page and not write it back to swap, **second chance** may be preferred. It adds the dirty bit, which is only checked if the reference bit is `0`. If the dirty bit is `0`, it replaces the page. If it's `1`, the OS could make an I/O request to write *in parallel* with it clearing the dirty bit and continuing to search for a `0`-`0`. Another option is to clear the dirty bit and continue searching, making sure to actually write back if the page does get evicted.

Page replacement is still relatively expensive, so it would be great if true page faults could be reduced. The **working set model** strives to do this by evicting pages that haven't been referenced in the last $T$ seconds, the window size. Eviction is handled by a daemon—background process or thread that perform system maintenance during a period of light workload.

Too small a window size can lead to **thrashing**, eviction of pages that are still in use. **Load control** can help ensure memory does not get over-committed by placing a limit on how many processes can be in physical memory at a time. If the system is already at the limit but a new process needs to run, some ready or blocked process is suspended.

{{< subtext >}}
    The window size is fairly large, enough to fit the amount of instructions that can be done in the time it takes for the page fault handler to finish.

    Load control is not used by current OSs. They're best effort operating systems, meaning they will always execute the program the user wants regardless of the performance hit.
{{< /subtext >}}



## {{< heading "Optimization" >}}
However, we end up going to memory twice: to access the page table then the data. The TLB (translation lookaside buffer) is a cache that does reduce the amount of page table accesses. It stores the most recently accessed page table entries. During address translation, both the TLB and page table will be queried. If there a TLB hit, then the page table access is terminated. If there is a TLB miss, the entry from the page table is put into the TLB.

{{< subtext >}}
    The TLB is part of the MMU (memory management unit). The MMU as a whole handles address translation.

    The TLB takes advantage of the principle of locality.

    A virtually addressed cache holds the recently accessed virtual addresses and maps to the corresponding data. Likewise, the physically addressed cache with physical addresses. The TLB gets a physical address, so the next step would be here.
{{< /subtext >}}

<!-- TODO: how does the OS determine to omit entries. regular page table is called forward-mapped -->
Space is another concern because page tables can be very big and is also per process. Also internal fragmentation. Multi-level page tables seek to solve the first problem. The $p$ bits are cut into parts, and the highest order bits are used to index into the first-level page table, the next highest order bits for the second-level page table, and so on. The final-level page table has the frame numbers. Space is saved for most processes since they don't use all the virtual address space. Therefore, entries can be pruned.

However, navigating through the multi-level page table hurts time. Inverted page tables use frame numbers as the index. This means that there is only one kept throughout the entire system. The entry will contain the PID, page number, and flags. In address translation, hashing is used to figure out which frame is for the page. However, this is so complicated that multi-level page tables is the convention.



# {{< heading "Heap" >}}
The heap is managed by the process itself. Therefore, there are many techniques for managing it happening at once. The runtime system (e.g., Java Virtual Machine or the handler for `malloc()`) manages the heap. When the heap doesn't have enough space (e.g., at the start), the runtime system will request pages from the OS. The OS allocates it these pages, flips the allocation bit, and makes more virtual addresses available. The pages are reclaimed when the process ends. The runtime system has several requirements to meet:
- Handle arbitrary request sequences
- Immediately respond to requests
- Metadata structures must reside in the heap
- Blocks must be aligned

Explicit memory managers (runtime system) makes the programmer actively manage the heap (think C). Programmer burden can be high, but efficiency can be very high and exposes addresses to programmers. Automatic memory managers manage the heap (think Java). However, explicit pointers are prohibited so that the runtime system can determine what data is no longer in use. Safe pointers are the substitute, and it's just an abstraction layer for the programmer.

One allocation technique that can used by the manager is the bump-pointer. It's basically first-fit; a pointer starts at the start of the heap, and it is fast-forwarded after an allocated block. Reusing freed blocks is not considered.

Another allocation technique is another free list with one of the fit algorithms (typically best-fit). A free block is divided into: pointer to the next free block (embedded free list), size of itself, and the usable space for the user.

{{< subtext >}}
    When splitting, allocate the later portion.

    Free list is the only option for explicit memory managers.
{{< /subtext >}}

Since searching for the best free block is slow, binning based on size is used. Exact fit has a bin for every size and a final bin for too-big blocks, and range binning has a bin for each range of size. Each utilizes an array of free lists.

Automatic memory managers implement garbage collection to handle deallocation. It consists of three parts: allocation, identification (of what's not in use), and reclamation. Garbage collection is evaluated on:
- Space efficiency
- Allocator efficiency: time to allocate and spatial locality of contemporaneously allocated objects
- Reclamation efficiency

Garbage are unreachable objects. One method of determination is reference counting, including from the register and heap. That latter point makes this method moot because of, think, circular linked lists. Tracing marks objects that are not reachable starting from the program roots (i.e., registers, stack, globals). 

{{< subtext >}}
    Dead objects are objects the program will certainly never reference again. Impossible for the runtime system to identify.

    In tracing, noting reachable objects by references from other objects is called performing of the transitive closure.
{{< /subtext >}}

Reclamation algorithms can be either non-copying or copying. A non-copying garbage collector is like deallocation of an explicit memory manager. Copying divides the heap into two spaces: 'to space' and 'from space'. Data is allocated in the 'from space' via bump-allocator until it is full. The garbage collector will then copy the reachable objects to the 'to space' and coalesce the entire 'from space'. 

Mark-Sweep is one reclamation algorithm. It uses free lists with binning. When there's not enough space for a request, a collection is triggered. The mark phase is through tracing. The sweep phase reclaims unreachable objects in a non-copying manner. This algorithm has good space efficiency (i.e., no external fragmentation), slower allocation time, poor locality of contemporaneously allocated objects. 

Mark-Compact uses a bump-pointer and tracing. After tracing, every reachable object is copied to the front of the heap. Then the bump-pointer is placed after the allocated objects. It has great locality because objects are allocated next to each other. It's also space efficient. However, Mark-Compact has poor reclamation efficiency because there's one traversal for tracing and another for copying.

Semi-Space is like Mark-Compact, but an object is copied immediately after being traced. There's also two spaces. Fast allocation and superb locality but terrible space efficiency.  

To deal with wasted space, we refer to the generational hypothesis: young objects are likely to die before older ones. To use this, the heap is further divided into young (nursery) and old. When the nursery is full, reachable objects are copied into the old space and the nursery is cleared. When the old space is full, both generations are traced and copied to the next space (either the next generation or the 'to space');

{{< subtext >}}
    Modern day garbage collectors are generation collectors.
{{< /subtext >}}