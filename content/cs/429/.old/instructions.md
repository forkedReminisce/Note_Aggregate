Instruction Set Architecture is the interface software interacts with to run on hardware %% behavior of machine language is defined by ISA. it is bit vectors encoding instructions??? %% An instruction takes the machine state to another. 

## ARM
ARM is a family of ISAs. It is a *load/store architecture*, meaning data from memory must be put in the register in order to perform data-processing operations. Instructions must be word-aligned (4-byte boundary). In the A64 ISA, instructions are 32 bits and little-endian.
-# A register is essentially **quick-access** memory on the CPU.

The AArch64 machine state supports A64. It has 64-bit registers; the following are the components:
- General-purpose registers from 0 to 30
- Stack pointer register `SP` contains the address of the newest activation record (merge)
- Program counter `PC` that holds the address of the current instruction 
- Process state `PSTATE` contains condition flags 
- Data memory
- Architectural status denotes any errors (refer to the pipeline)

## Instruction Processing
Instructions consist of source operands, operation code, destination, and any side-effects. There are three types of operands, but they are all R-values; destination is a register or somewhere in memory. The following is the procedure to processing an instruction.

[TODO: handle the extra decode stuff]: #
1. Fetch instruction from `PC`
2. Decode the instruction and read operands
    1. Extract the `opcode`. This resides in the most significant bit positions. These are varying sizes, but the largest is 11 bits. Start with 11 bits, reference a lookup table, then move down a bit.
    2. Extract the remaining fields based on the format specified by the `opcode`.
3. Execute necessary operation (update `PSTATE` if needed)
4. Access memory (if needed)
5. Write to destinations (if needed)
6. Control transfer and architectural status

*Immediate* operands are constants. These are written into the bits of the instruction. *Register* operands refer to the contents of a register. *Memory* operands are for accessing memory locations %% how is this encoded %%.

## Branching
Branching is the nickname given to *non-sequential control transfer*. This is commonly used to achieve if-else statements and loops. There are two types of branching: conditional and unconditional. Unconditional always branches, while conditional depends on the truth values of its conditions. This is where `PSTATE` becomes useful. Regardless, the `PC` is updated perhaps not to `PC + 4`. In a branch instruction, a signed-word-scaled 26 bit value is embedded (up to ±128 MiB or ±2^27 bytes). This represents the *signed* distance from the `PC` to the instruction to branch to.

Setting `PSTATE` from addition is based on $C_out$s and the sum bit string. 
- `N` is equal to the most significant bit of the sum
- `Z`
- `C` is based on the final, "lost" $C_out$
- `V` becomes `1` if and only if the final $C_out$ and $C_in$ differ

-# To go more in-depth into `V`, it is true under two scenarios: when both numbers are non-negative and the result is negative, or the inverse. Knowing that negativity is based on the most significant bit, we can deduce such a conclusion.

## Procedures
Think functions and methods. At runtime, procedures are managed on the **run-time stack**. Inside are **activation records**—the technical term for stack frames %%where is the stack relative to the heap?%%. Several important details are stored in a record, and this will be explained in the coming sentences. For now, know that the newest record has the lowest memory address, records can be of varying sizes, and are 16-byte aligned.
When a call to another procedure is made, the local %% program ??? %% state of the caller needs to be isolated from the callee. This is known as *state management*. To accomplish this, AArch64 splits the set of general-purpose registers into disjoint subsets. One subset is caller-saved and another is callee-saved; look at the table at the end of this section for details %% how are R0-R7 saved? %%.
To support parameters and returned data, registers 0 to 7 handle *data transfer*. Any additional parameters are wedged between the caller's and callee's activation records. The result is then returned in register 0.
Two general-purpose registers serve *control transfer*. These should be treated as caller-saved registers. The **link register** `LR` resides in 30. It points to the top of the link stack, a stack of return addresses to the caller. The return address is the instruction after the branch. Register 29 is the **frame pointer** `FP`. This serves as a linked list of frame records, which reside inside activation records *somewhere*. On that note, this structure in a way constructs a linked list of activation records. A frame record contains the caller's frame record and the caller's return address *in that order from lowest memory address*. `FP` points to the newest frame record %%it actually points to the frame record of the caller%%.
%% AArch64 register type table here %%