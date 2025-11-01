---
draft: true
title: 

params: 
    desc: 
    author: FREEZURN 
---



# {{< heading "Stable Storage" >}}
Each I/O device consists of:
- Bus (typically shared by multiple devices)
- Device port: data of status, control (command to perform), data-in (sent to CPU), data-out (back from CPU)
- Controller: packages the data
- Actual device

A device can be described with:
- Transfer unit: character or block
- Timing: synchronous (CPU waits on the device) or asynchronous
- Shared or dedicated (e.g., keyboard)
- Speed
- Operations: input, output, or both
- *Access method*: sequential or random

To the application programmer, the OS provides an overall interface to devices. Not only does the OS perform the operations, it also allocates and releases devices from processes. It automatically installs device drivers with the device-specific behaviors. It provides access control (synchronization). Buffering, caching, and spooling provide efficient communication with the device. 

Stable storage are persistent and cheap. Magnetic disks consists of multiple heads connected with an arm assembly (or comb). They move in unison, but data can only be read from one head at a time. An entire sector must be read and write to. Blocks are the smallest unit the OS chooses to operate on, and it is a number of continuous sectors. The block size is shared between every stable storage device. 

{{< subtext >}}
    A typical sector size is 512 bytes.
    
    Block size is typically the same as page size.

    Outer tracks are faster.
{{< /subtext >}}

Seek time is the most dominant time cost. Head Switch Time is the time it takes to switch from one head to another. Transfer time is split into two measures: surface transfer time is writing the data the head reads to the disk buffer, and the host transfer time is transferring the disk buffer to host memory.

<!-- TODO: reexplain remapping. it uses flash translation with the controller? -->
Solid State Drives (SSDs) has no moving parts, removing seek time and rotation time. They also use less power and more resistant to physical damage. Operations are performed in units of pages. However, the entire block must be erased to perform a write. Since writing is orders of magnitude more lengthy than reading, remapping will mark the page as old and puts the new data into another block with free space. Flash devices can have multiple independent data paths, which allows concurrent requests. 

{{< subtext >}}
    Pages make up blocks which make up planes.
{{< /subtext >}}

Flash drives is not as reliable as hard drives. So the SSD will mark blocks defective, wear-level the drive by spreading updates to frequently used pages across many physical pages, and reserving space. 

The CPU and the stable storage device need to communicate. Polling is when the OS checks whether the stable storage device is finished with the operation. If it's not ready, the CPU will do other work until the OS gets the task again. Interrupt is when the CPU does other work until it gets an alert from the device. Direct Memory Access expands on interrupt by including DMA controllers on the device that write to memory instead of having the CPU do it. The DMA writes a block instead of a word at a time.

The file system is the medium the user uses to interact with data (files) in stable storage. Each file system focuses on a partition, a collection of cylinders. A file system has metadata: file system type, number of blocks in the file system, file system permissions, etc. All this metadata is stored in the superblock. Free space is kept track with a free list or bitmap (of each block; `1` means allocated). File systems design goals are:
- Efficiency: space and time
- Usability: Easy interface
- Reliability: Writes can be interrupted at any time and protection against corruption

## {{< heading "Design" >}}
<!-- TODO: we also want to be able to support file growth -->
Most files are small, but most of the space is taken up by large files. 

Contiguous allocation is when file data is given a continuous chunk of free blocks when the file is created. It uses the fit algorithms. The file metadata, which is typically disconnected, has the start location and size of the data.  

<!-- retrieving the file header then traversing through the linked list is many accesses. no support for transactional updates. TODO: FAT has size cap -->
Linked allocation stores file data blocks like a linked list. The file header doesn't store size this time. This is bad for random access. The File Allocation Table (FAT) seeks to solve this with a Master File Table (MFT) that lists the links of all the blocks of a file. A FAT entry contains this data in some number of bits. Since this is small, it can be brought into the buffer cache and stay there. Alternatively, direct allocation just puts all the pointers in the file header. However, this array can get too large for the header to contain in a single block. Indexed allocation has the header block point to index blocks, which points to a subset of data blocks. There are linked index blocks that go sequentially or multilevel index blocks (although, this puts a cap on the file size). If we use an asymmetric tree, multilevel index blocks can be very powerful as not every level has to be allocated. File headers in this case would be called inodes.

Each file header also has:
- Owner ID
- Size
- Permissions
- Last modified time

All of them are stored in a location the OS knows.

<!-- TODO: is this the Fast FileSystem for Linux. there is an inode array -->
Mechanical sympathy is the necessity to understand hardware to do effective design. A partition's tracks are divvied up into block groups. This exploits locality of files inside a particular directory. The inodes of files in this directory are also stored in this block group, albeit still far from the actual files (on the outer track for the rotational velocity). Blocks are allocated on a first-basis. The available space is shrunk 10% for effective locality. The buffer cache will return success even when the write has yet to be performed. 

NTFS is Windows' file system. It uses flexible tres and an MFT for file records (headers). File records have a lot of metadata (e.g., permissions and name) and "data". If the data is small enough, it will be stuffed into the record (resident). Otherwise, it goes into varying length, contiguous extents and the record stores pointers (nonresident). A record can also have attributes that point to continued file records. Metadata (e.g., MFT and free space bitmap) is stored in particular files. Uses best-fit and caches a small section of the free space bitmap (writes that occur together in time get clustered together).  

{{< subtext >}}
    File records are much bigger that the entire MFT isn't cached but rather a window of it.
{{< /subtext >}}

<!-- TODO: -->
Extents are typically used with SSDs because metadata. Copy-on-Write file systems updates data and metadata to a new location. This is because small writes are expensive. 



# {{< heading "Usability" >}}
Directories create a name space for files. This is why the same file name can be used in different directories. File names are actually mapped to file numbers, and this mapping is known as a directory entry. The file number indicates the location of the file header. The collection of mappings are stored in a file, and this file is represents the directory. Directories can only be modified in kernel mode. 

There was first the Make-It-Work Strategy, which offered only one name space for the entire system (two users couldn't have a file with the same name). The Simple User-Based Strategy has a separate single directory for each user. Multi-level Directories is used today, and they consist of a root directory and user directories that many contain subdirectories. 

To find the blocks for a file, get the file number of the root directory, which is known. Then do traversals to directories and finally the file.
