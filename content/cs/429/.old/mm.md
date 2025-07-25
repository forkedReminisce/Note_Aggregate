When an object of unknown size at program execution time is put into memory, it is in dynamic memory. This object will continue to exist even after its identifier (e.g., variable) is out of scope. 

## System
The mutator (i.e., application) makes calls to the allocator for memory; the allocator manages the arena provided by the OS. This arena is fragmented into allocated and free blocks. Free blocks are organized in one or more free lists.

%% what is the difference between heap and pool/arena %%
When the mutator makes a request, the allocator returns an allocated block larger than what was asked. However, this is abstracted from the mutator. The main portion is the payload, while the extra bytes are overhead for storing useful information to the allocator.

## Free List Management
Format is the way blocks are structured. Along with an initial bit (`1` for allocated, `0` for free) indicating the state, there is a header holding metadata such as the *size of the block* (not payload). In addition to the payload, an allocated block may have padding because the returned memory address must be a multiple of 16. A free block will contain pointers to other free blocks.

-# A canary value—special bit vector—may be placed in the header. If, during freeing, the value is different, there is corruption.

Organization concerns the free list. 
- **Explicit** only contains free blocks; **implicit** has all
- **Binned** are multiple different free lists based on a metric; or **single**
- General linked list considerations
- Order

-# Free lists are not linked lists.

Selection is the policy for picking the appropriate free block to satisfy the request. The most common policies are: best-fit, worst-fit, first-fit, and next-fit. On next-fit, when a free block is selected, the position at or around is saved and picked up again next time. %% this usually necessitates circular %%

Splitting. There is choosing the side within the free block to allocate, and when the split: never, always, threshold. Threshold is a fixed size or percentage the leftover portion must meet to warrant a split.

Coalescing concerns returned allocated blocks. Freeing may be immediate, deferred, or none. Deferred is only when the current request cannot be fulfilled. Then there are boundary tags; the header is duplicated at the end of the block (i.e., footer). 

%% internal fragmentation is the dividing of a block; external fragmentation is the dividing of the arena %%
%% global variables are initialized at execution time; other variables in the stack (or heap???) during runtime as appropriate %%

## Alignment
A bus acts as a medium between memory and the CPU for data. It can only hold 8 bytes at a time. It is important for data to be aligned to make this process easier.
Basic type objects must have a memory address that is a multiple of its size. Derived data types are aligned when:
- Sub-object rule: every sub-object is aligned
- Array rule: An array of the object is aligned.

An easy way to tell array rule is met is when the size is a multiple of the largest field/variant. Padding may be added to satisfy the rules; size will include this padding. 