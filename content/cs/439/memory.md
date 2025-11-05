---
draft: false
title: Memory Management

params: 
    desc: The OS has to manage the memory of a process. A process has to manage its own heap. 
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


## {{< heading "Optimization" >}}
In general, accessing data requires going to memory twice: one to get to the page table and another to the data. The former can be less frequent with the use of the *TLB* (Translation Lookaside Buffer) cache, which resides in the MMU. It stores the most recently accessed page table entries. During address translation, both the TLB and page table will be queried *in parallel*. If there is a TLB hit, then the page table access is terminated. If there is a TLB miss, the page table entry is put into the TLB.

Space is another concern because page tables can be very big and are also per process. *Multi-level page tables* address this by first cutting the $p$ bits into parts. The highest-order part indexes into the first-level page table... The final-level page table has the frame numbers. Since most processes don't actually use all the virtual address space, some lower-level page tables can be pruned. This is made possible with a sort of lazy allocation strategy for the lower-level page tables.

However, navigating through the multi-level page table is bad for time. *Inverted page tables*, opposite to the forward-mapped page table, are an alternative that hashes frame numbers to index into. This means that there is only one kept throughout the entire system. An entry will contain the PID, page number, and flags. In address translation, hashing is used to figure out which frame the page is in. 

{{< subtext >}}
    Inverted page tables are so complicated that multi-level page tables are the convention.
{{< /subtext >}}


## {{< heading "Loading" >}}
There are two policies to loading pages of a process for the first time. Demand paging loads a page upon reference. Pre-paging will see the OS predict and load pages the process will likely immediately need. If the process tries to access an unmapped virtual address, the OS will execute the **page fault handler**. This handler will figure figure out why its being called: unallocated page will cause a memory exception, while a nonresident page means true page fault. The latter requires bringing the page into physical memory. 

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

Since it's cheaper to simply replace an unmodified page and not write it back to swap, **second chance** may be preferred. It adds the dirty bit, which is only checked if the reference bit is `0`. If the dirty bit is `0`, it replaces the page. If it's `1`, the OS could make an I/O request to write *in parallel* with it clearing the dirty bit and continuing to search for a `0`-`0`. Another option is to clear the dirty bit and continue searching, making sure to actually write back if the page does get evicted. Make sure to have a true dirty bit.

Page replacement is still relatively expensive, so it would be great if true page faults could be reduced. The **working set model** strives to do this by evicting pages that haven't been referenced in the last $T$ seconds, the window size. Eviction is handled by a daemon—background process or thread that perform system maintenance during a period of light workload.

Too small a window size can lead to **thrashing**, eviction of pages that are still in use. **Load control** can help ensure memory does not get over-committed by placing a limit on how many processes can be in physical memory at a time. If the system is already at the limit but a new process needs to run, some ready or blocked process is suspended.

{{< subtext >}}
    The window size is fairly large, enough to fit the amount of instructions that can be done in the time it takes for the page fault handler to finish.

    Load control is not used by current OSs. They're best effort operating systems, meaning they will always execute the program the user wants regardless of the performance hit.
{{< /subtext >}}



# {{< heading "Heap" >}}
Looking into the process itself, its heap is managed by its runtime system (e.g., Java Virtual Machine or `malloc()`'s handler). The entire virtual address space isn't initially unlocked, so the runtime system must request pages from the OS. The OS will then create the page table entries for these allocated pages. 

{{< subtext >}}
    Free blocks are typically coalesced before a request more pages.
{{< /subtext >}}

*Explicit memory managers* gives the programmer great discretion over the heap (think C). Programmer burden can be high but so is efficiency and exposes virtual addresses to programmers. The runtime system can allocate by using a binned free list or *bump-pointer*. The only benefit of the latter is that it is fast and has great spatial locality of contemporaneously allocated objects; it allocates the block starting from where it is currently pointing then fast-forwards itself past this new block. It can't even reuse blocks that are eventually freed.

{{< subtext >}}
    There are two binning strategies for the free list: exact fit has a bin for every size up to a point, where the final bin takes the rest of the blocks. Range binning has each bin take a range of sizes.

    With a free list, if a free block needs to be split, the later portion is allocated.
{{< /subtext >}}


## {{< heading "Automatic Memory Managers" >}}
*Automatic memory managers* manage the heap itself (think Java). To do this, explicit pointers are taken away from programmers, and instead *safe pointers* wrap them and given to the programmer.

Since there's no such `free()` function, these memory managers implement **garbage collectors**. This results in a cycle of ==allocation, identification (of what's not in use), and reclamation==. 

{{< subtext >}}
    Allocation efficiency also includes spatial locality of contemporaneously allocated objects
{{< /subtext >}}

Garbage is *unreachable objects*. One method of identification is *reference counting*, which includes from the register and heap. That latter source makes this method moot because of cyclical references (e.g., circular linked lists). Instead, *tracing* starts from the program roots (i.e., registers, stack, SDS) and marks objects reachable. 

{{< subtext >}}
    Dead objects will *certainly* never be referenced by the program ever again. They're impossible for the runtime system to identify.

    In tracing, performing of the transitive closure describes traversing further links from the program roots.
{{< /subtext >}}

<!-- TODO: does sweeping handle unreachable objects at once (i.e., after tracing) -->
Reclamation algorithms fall into either non-copying or copying categories. **Mark-Sweep** is a non-copying reclamation algorithm. Since it uses binned free lists, reclaimed blocks must be tediously coalesced and put into the free list. It uses tracing for identification. Unexpectedly, Mark-Sweep has ==good space efficiency since it has little external fragmentation==. However, it does have bad allocation efficiency. 

The remaining reclamation algorithms are of the copying type. **Mark-Compact** uses a bump-pointer and tracing. After tracing, every reachable object is copied to the start of the heap, and the bump-pointer is put after this chunk. This is space ==and time== efficient, but it does have poor reclamation efficiency. 

**Semi-Space** is like Mark-Compact, but an object is copied immediately after being traced. More crucially, the heap is divided into two parts: 'from space' and 'to space'. Which partition gets what name alternate, but data gets allocated into the 'from space'. When the 'from space' is full, it is traced and the reachable objects are copied into the 'to space'. Having a dedicated 'to space' makes it so that the memory manager does not have to check for any allocated data there. However, since one part must always be left alone, there is terrible space efficiency.

<!-- nursery is reaped? -->
The generational hypothesis states that young objects are likely to become obsolete before older ones. **Generational collectors** rely on this idea to improve space efficiency. The heap will be divided into at least three spaces: nursery, the generations older, and a 'to space' for the final generation. When the nursery is full, reachable objects are copied into the next generation, and the nursery is cleared. When an old generation is full, all generations up to this one, inclusive, are traced and copied to the next generation or the 'to space'.

{{< subtext >}}
    Modern day garbage collectors are generation collectors.
{{< /subtext >}}