---
draft: false
title: Handling Files

params: 
    desc: There's a lot of data that needs to be durable. Therefore, they're put into stable storage. A file system facilitates working with these files in a simpler way.
    author: Andrew Nguyen 
---



Each I/O device consists of:
- Bus (typically shared by multiple devices)
- Device port: status, control (command to perform), data-in (sent to CPU), data-out (back from CPU)
- Controller: packages data-in

<!-- TODO: is access control right? -->
The OS provides a generalized interface to these devices, HAL. Not only does the OS perform the operations, it also allocates and releases devices from processes. Other things the OS handles is installing device drivers and access control (synchronization around the device).

# {{< heading "Stable Storage" >}}
Stable storage is persistent and cheap. **Magnetic disks** are one such stable storage. Operations are handled by a head, of which there are many connected with an arm assembly. They move in unison and ==do not support concurrent operations==. Considering this, a cylinder is the track of each platter that can be vertically stacked. Because the OS tends to do operations on a larger amount of data than the size of a sector, it does operations in units of blocks. 

{{< subtext >}}
    A typical sector size is 512 bytes. Meanwhile, block size is typically the same as page size.

    The OS operates on the same block size for every device.
{{< /subtext >}}

==Disk is slow, though==. Seek time describes moving the head to the correct track. It has an additional Head Switch Time for switching heads across the cylinder. These are the most dominant cost. Rotational time is waiting for the sector to come around. Transfer time is further split: surface transfer time for performing the operation on the sector and host transfer time for transferring data between the head and disk buffer.

{{< subtext >}}
    Outer tracks spin faster.
{{< /subtext >}}

**Solid State Drives** (SSDs) ==have no moving parts==, eliminating seek time and rotation time. Additionally, they can have multiple independent data paths, allowing concurrency. SSDs also use less power and are more resistant to physical damage. Operations are performed in units of pages; however, the entire block (different meaning) must be erased to perform a write. Since writing is orders of magnitude slower than reading, *remapping* will load and modify the block in memory and write the entire block to a free block.

{{< subtext >}}
    Pages make up blocks which make up planes.
{{< /subtext >}}

Since SSDs are not as resilient as disk, it has to use some tricks. It marks blocks defective, wear-level (spreading updates to frequently used pages across erasure blocks), and reserving space to sustain these tricks.

<!-- TODO: does DMA write the entire data to pages or just a page size? -->
The CPU and the stable storage device need to communicate. **Polling** is when the OS checks whether the stable storage device is finished with the operation. If not, the OS will reschedule it to the ready queue. **Interrupts** will let the CPU do other work until it is alerted by the device for completion. ==Both of these operate in a word at a time==. **Direct Memory Access** expands on interrupts by introducing DMA controllers that are able to write to memory. Not only does this decrease the burden on the CPU, but the CPU is also only interrupted when the ==entire transfer is complete==. 



# {{< heading "File Systems" >}}
File systems are the medium the user uses to interact with data (files) in stable storage. A system has multiple file systems. On disk, each one gets its own *partition*, a collection of neighboring cylinders, to optimize seek time. Considering this, file systems have metadata, including its type, number of blocks, and permissions, stored in its superblock. 

The goals when designing a file system are:
- Efficiency
- Usability: easy interface for programmers and users
- Reliability: writes can be interrupted and protection against corruption

Free space is kept track with a free list or bitmap (of each block; `1` means allocated).


## {{< heading "Efficiency" >}}
When it comes to allocation strategies, it's also important to keep in mind file growth and random access. **Contiguous allocation** will find a contiguous chunk to give to file data. The *file header*, which is typically stored elsewhere, needs to keep the pointer and size. Unsurprisingly, this cannot handle file growth. 

{{< subtext >}}
    A file header also stores the owner ID, permissions, and last modified time.
{{< /subtext >}}

<!-- TODO: FAT has size cap on file system (not only files themselves) and basically replaces the header (no metadata). -->
**Linked allocation** stores file data like a linked list of blocks. Since this will sometimes require many read requests, the *File Allocation Table* (FAT) has a Master File Table (MFT) that can be brought into the *buffer cache*. This way, ==traversal happens in the MFT== and only the desired block is retrieved. Random access is still bad and file size is limited along with some other limitations.

{{< subtext >}}
    The buffer cache lives on RAM. This is not part of the process' state, so it isn't saved.  
{{< /subtext >}}

**Direct allocation** has the file header store all the pointers to the disjointed blocks. However, this array can get too large for a single header. Therefore, **indexed allocation** has the header point to index blocks, which point to a subset of data blocks. These index blocks could be linked or multilevel. Like multi-level page tables, multilevel index blocks supports pruning, but they also cap the file size.

<!-- TODO: the buffer cache holds the data to write. it is buffered by the buffer cache. the cache handles write. space is shrunk because scattered free space -->
The **Fast File System** (FFS) of UNIX uses multilevel index blocks. A file header, inode, also stores pointers to a small amount of data blocks. *Mechanical sympathy* is the idea that good design molds itself around hardware. FFS exhibits this in a number of ways. Firstly, the partition's tracks are divvied up into block groups. By dedicating a set of directories to a group and storing the respective inodes on the group's outer track, seek time is reduced even further. Next, first-fit is used. Available space is shrunk by 10% for effective locality. The buffer cache will return success even when the write has yet to be performed. 

<!-- TODO: inflates metadata size of small files -->
NTFS is Windows' file system. Headers, file records, are stored in an MFT. This MFT and other metadata are stored in files. Data is stored in extents—varying length contiguous blocks. All the extents can be found in a flexible tree, meaning that the depth depends on how large every extent ends up becoming. Depth comes in the form of indirection, in a multilevel way, to more file records. If the data is small enough, it might just be resident in the record. NTFS uses best-fit. Because it caches a section of the free space bitmap, new files will be clustered together. 

{{< subtext >}}
    File records are so big that the only a part of the MFT can be cached.
{{< /subtext >}}

Most file systems use extents with SSDs. This is because there's less overhead associated with it compared to hard drives (i.e., less need for multilevel index blocks). 



# {{< heading "Usability" >}}
**Directories** are files that map file names to file numbers. This is a directory entry. For example, `.` and `..` are mapped to the current directory's file number and the previous directory's file number respectively. The **file number** indexes into the data structure of file header pointers.

{{< subtext >}}
    Each directory creates a name space.

    CWD is stored in the PCB.

    There's some overhead in that anything that is read can only be processed once it's made its way into memory.
{{< /subtext >}}

The Make-It-Work Strategy only had a single directory (not necessarily folder) for the entire system. The Simple User-Based Strategy then gave each user their own directory. Multi-level Directories, which is what is used today, has the root directory and directories for basically every folder.

When given an *absolute path*, go to the root directory, whose file number is saved. After reading its header and then data, the subdirectory's file number can be retrieved.



# {{< heading "Reliability" >}}
**Consistency** means that there is no discrepancy between the metadata structures (e.g., free space bitmap and header). There's no action against a mismatch with data blocks. 

<!-- TODO: what is ad hoc. why synchronous vs asynchronous -->
UNIX updates structures, including data, in a particular order. For example, data might be updated first to maintain consistency as long as possible. If there's a crash, `fsck` will scan all of storage for inconsistencies. If the inode has the block, the data bitmap is updated; if the inode doesn't have the block, the bitmap is flipped. Data is written with asynchronous write-back, meaning that it'll be written at some point (by 30 seconds). Metadata is handled with synchronous write-through because it's important to keep it consistent. In general, though, this is slow.

<!-- no log means the changes made it to disk -->
If a set of files need to be modified as a unit, atomicity is necessary. Therefore, *journaling file systems* use write-ahead logging for metadata changes. This happens before actually modifying metadata. This makes `fsck` a liability. Because of **disk head scheduling**, `Commit` can be prematurely written before the entire transaction is written. Therefore, `Barrier` is placed before `Commit` to say that the previous data must be written before writing the rest.
<!-- TODO: there's a disk queue -->
- FIFO: a queue of requests
- Shortest Seek Time: handle the nearest track
- Elevator: handle the requests in one direction, turn around at the end, handle requests in the other direction
- LOOK: optimizes Elevator by turning around at the track of the final request in one direction
- C-SCAN: handle requests in one direction, and at the end, jump to the other edge and don't change direction
- C-LOOK: jump at the final track of a request