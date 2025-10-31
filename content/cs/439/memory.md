---
draft: true
title: 

params: 
    desc: 
    author: FREEZURN 
---



The physical address space is defined by the RAM (determines the size). Logical or virtual address space is the collection of addresses that a process can access. Segment is the logical address space but in the scope of the physical address space.

In uniprogramming (one program running at a time to its termination), the program is always loaded at address 0 to ease the mapping from physical to virtual memory. When it finishes, the segment is reclaimed for the next program to take. For multiprogramming, we want something known as *transparency*: multiple processes, illusion that the process is the only process is memory, and where in physical memory they reside should not affect anything. Safety is ensuring processes cannot corrupt each other's or the OS' address space. Efficiency.

{{< subtext >}}
    OS sits in the highest memory.
{{< /subtext >}}

**Relocation** is how we support multiprogramming. Processes will be loaded wherever it fits, and if it cannot, the OS will wait for a space to open. Of the process' segment, the lowest memory address is known as the *base address*. The program's compiler will behave as though the program was loaded at 0. The loader fixes this in *static relocation*, where it adds the base address to all addresses in the binary. However, moving a process in memory is impossible.

<!-- TODO: how can there be invalid addresses? how does a process tell the OS how much memory it needs? -->
To circumvent this restriction, *dynamic relocation* is used. First, two new registers: base register stores the base address of the process and limit register with the largest address in the process' logical address space. Then hardware will, in parallel, check if the address is below the limit register and add the base register. Moving processes just reuses this architecture, albeit, tediously. Process' address space can also grow.
<!-- disadvantages: sharing memory is hard, all of the process must fit in memory (cannot load a portion), expensive since checking ever address, only supports primitive security -->

The goal of a memory allocation policy is to minimize wasted space, or *fragmentation*. External fragmentation is the cutting up of free blocks, and internal fragmentation is unused memory inside an allocated block. 
- First-fit: allocate the lowest addressed free block, but a lot of external fragmentation
- Best-fit: choose the smallest, adequate free block. Free list sorted by size. External fragmentation unusable and slow deallocation.
- Worst-fit: choose the largest, adequate free block. Free list sorted by size. Allocate is fast. External fragmentation and slow deallocation

*Compaction* is a strategy to eliminate fragmentation. Copy programs' address spaces to another address in order to coalesce the free blocks. *Swapping* allows a high priority process to run despite no appropriate free blocks. Some process is *suspended* (at the OS' discretion to be put back on the ready queue). Its address space is moved to disk, specifically the swap partition. Then this high priority process can take the address space.



# {{< heading "Virtual Memory" >}}
Ours goals is to:
- Allow processes to use more memory than that which is physically available
- Reduce or eliminate external fragmentation
- Easily allocate and deallocate memory (and grow processes)
- Share memory between processes
- Fine-grained protection

Process' virtual address space is partitioned into *pages* of size *minimum unit*. Similarly, the physical address is partitioned into *frames*. Pages are slotted into frames. To keep track of the process' virtual address space, the *page table* maps a page number to a frame number (number is the lowest address divided by the minimum unit). Pages may be put into swap if the process doesn't access it for a period of time, allowing other pages of other or the same process to be inserted.

{{< subtext >}}
    A process' virtual address space can be larger than the physical address space.

    Virtual address space size is universal. 
{{< /subtext >}}

Virtual addresses can be thought of as a $(p, o)$ pair bit string. The high-order $p$ bits represent the page number. The following $o$ bits are the page offset. Physical addresses have the synonymous $(f, o)$ bit string for frame number and frame offset respectively.

The page table is like an array. Using the $p$ bits to index, the OS can get the $f$ bits and address translate using those bits and the offset bits. The page table lives in the PCB. The page table base register holds the pointer to the page table.

Sharing memory is achieved by putting in multiple page tables the same $f$ bits. The OS knows to do this for processes via a system call. A page table entry contains some other bits. Permissions. Resident bit reveals whether the page is currently in physical memory, and then the next bits (including $f$) can either be the frame number or something else (e.g., swap slot). Reference bit tells whether the page has been used recently. Dirty bit determines eviction behavior.

There are two policies to loading pages of a process for the first time. Demand paging loads a page when it is referenced. Pre-paging will see the OS load pages the process will likely immediately need.

{{< subtext >}}
    Demand paging is more popular due to the principle of locality.
{{< /subtext >}}

Pages of the code section are typically not in memory or even swap. They live in the file, and whatever code segments are being run are put into memory. Data and stack pages are in memory or the swap partition. If the process tries to access a non-mapped page, the OS will execute the page fault handler. The handler will figure out why the page is not in physical memory; unallocated data is treated as an exception, while a resident bit of 0 means true page fault. A true page fault is a mistake on the OS' part, and it fits the page into a frame.

The goals of page replacement algorithms are to reduce the number of page faults and efficient. The ideal solution is to replace the page that gets referenced the latest. This is predicting the future, so we use least recently used (LRU). 

LRU is hard to implement efficiently. Therefore, **clock** is used. There will be a circular linked list of all the pages in memory. The pointer to this list will be to the oldest page. When a page needs to be evicted, it will look for a page with a reference bit of `0`, meaning the page hasn't been accessed since the last time it was checked. If it's `1`, the reference bit is re-set to `0`. Either way, the hand is advanced.

{{< subtext >}}
    When a page is replaced, the new page will get a reference bit of `1`.

    Hardware sets the reference bit to `1`.
{{< /subtext >}}

It's actually cheaper to replace an unmodified page since it doesn't need to be written to disk. This is **second chance**, and it will also check the dirty bit if the reference bit is `0`. If the dirty bit is `0`, it replaces the page. If it's `1`, one option the OS could make is an I/O request to write *in parallel* with clearing the dirty bit and searching. Alternatively, clear the dirty bit and move on, while making sure to actually write if the page gets evicted.

The working set model is a result of trying to use the principle of locality to evict a bunch of pages, create free frames, and avoid having to do page replacement. It consists of pages the process has referenced in the last $T$ seconds (window size). Evictions of pages that exceed the window size are handled by deamons—background processes or threads that run from the idle loop that perform system maintenance.

Thrashing is when pages are evicted while they are still in use. This can be due to a small window size. Since page faults are so slow, the window size ends up being fairly large, enough to fit the number of instructions that can be done in the time of a page fault.

Load control is the maximum number of processes in memory at a single time. This is to ensure memory does not get over-committed and avoid thrashing. If the number will be exceeded, a ready or blocked process is chosen to be suspended. However, current operating systems do not use load control.

Small pages reduce internal fragmentation. Larger pages reduce the number of page table entries and page faults. They also reduce I/O time. 



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



# {{< heading "Managing The Heap" >}}
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