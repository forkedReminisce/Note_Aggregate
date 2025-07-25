bistable device: a physical object with two stable—cannot randomly change—states
  using semiconductors (e.g., silicon) in electronics makes making switches easy
    MOSFET = metal oxide semiconductor field effect transistor
      transistors act as a switch by applying physical force
      MOSFET applies voltage to gates to switch between states; source and drain are electrically connected
        cut off: basically no current; gate-to-source voltage is below threshold
        saturation region: current has hit max; drain-to-source voltage is too high
        most basic building block of electronic devices
  ferromagnetic material is another way of achieving a bistable electronic device
    usually seen in hard drive discs
    based on direction of magnetization (stable saturation M-states of the magnetic hysteresis curve)
math definition of modulus
  a = qd + r
    given a and d
  	where 0 < d
  	0 <= r < |d|
  	r = a mod d
  normal for positive
  negative a is: find a multiple of d that is less than or equal to a then find the difference for r

## Word Size
There is a power of 2 number of cells in memory indexed from 0 to 2^w - 1. Each ell holds one byte of data. Memory accesses are always a whole number of bytes. This is called byte-addressed memory. w is called the word size of the machine. w is the number of bits needed to represent the address of a memory cell.
- AArch64 (and x86-64) W = 64
- AArch32 (and IA32) W = 32

## Arithmetic On Representations
For floating point, make sure the power on the 2 are the same. This can be done by shifting the significand (including the hidden bit) then performing the opposite (multiply for right shift, divide for left shift) on the power on the 2. Then add and right shift the sum once while incrementing the power on the 2.
%% converting decimals from base-10 to base-2. multiply the decimal representation by 2. when there is a whole 1, make it 0 and continue. when the product is 1.0, stop and go back to the top and record in order the whole part. if the product is a repeat, go back to the original step and indicate repetition. %%

## ISA
System ISA has higher privilege and may only be used by the OS. User ISA by everyone else. The **Application Binary Interface** includes user ISA and system calls (e.g., `sbrk()`). **Application Programming Interface** includes user ISA and high-level language library calls (e.g., `malloc()`).

The von Neumann Architecture has:
- Processing unit
- Control unit: ensure the instructions are being performed and program counter (what is the next instruction)
- Memory

The von Neumann bottleneck is when only an instruction or data can go between the process and memory. This is because there's only one connection. There is also no distinction between instruction and data memory. This comes with the risk of self-modifying code.

The Harvard Architecture fixes these problems by having separate memories of instructions and data and separate connections.

## Buses
A bus is a broadcast mechanism. It is an organizational idea that contains data, address, and control signals that individually have different destinations. Think of a bus as a highway. Take a memory write for example: the CPU sends the memory address and indicates that it is a write operation. Data is either sent together or delayed. Then memory sends a first response signal indicating that it has recieved the data (so the CPU can take off data from the lines), then later a final acknowledge stating the write was successful. This is known as a *request acknowledge protocol*. 

Recall that accessing memory and storage takes magnitudes of time longer. This is known as I/O latency, and it typically "blocks" the pipeline from continuing. Therefore, a strategy known as context switching is used. *Memory-Mapped I/O* refer to a range of addresses (I/O ports) reserved for communicating with I/O devices. When the processor makes its request through the I/O port, it freezes its current process and works through the next in queue. The I/O device then performs the operation on its own (*Direct Memory Access*), then it sends an interrupt to the CPU telling it to put the process back in queue.