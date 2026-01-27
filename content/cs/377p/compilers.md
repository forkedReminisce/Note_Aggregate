---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



A compiler converts high-level source code into assembly language. The assembler then turns that into machine language. 



# {{< heading "x-86" >}}
The x-86 instruction set has general purpose 32-bit registers `eax`, `ebx`, `ecx`, `edx`, `esi`, and `edi`. There are 16-bit and 8-bit takes on these. Like in AArch64, these general purpose registers are partitioned for the purposes of state management. In gcc, `eax`, `ecx`, and `edx` are the caller-saved registers, while `ebp`, `ebx`, `esi`, and `edi` are callee-saved. `eax` also serves as the return value register. Additionally, `esp` is the stack pointer, and `ebp` is the frame base pointer. The condition code register is analogous to `PSTATE` from AArch64.

An instruction is formatted like `Opcode a, b`. `a` and `b` are operands, but at most one can be a memory address. `b` does also serve as the destination.

On the topic of memory, more often than not, one will want to use an addressing mode to produce an effective address:
- Absolute: \\[`addr`\\]
- Register indirect: (%`reg`)
- Register offset: `num`(%`reg`) — `num` + `reg`
- disp(base, offset, scale): `num1`(%`reg1`, %`reg2`, `num2`) — `num1` + `reg1` + 4 \* `reg2`

There is a way to declare constants so that its address is known at compilation. This is through a label: `label: .type value`. To use it in an instruction, prefix the label with a `$` like so: `$label`. 

{{< subtext >}}
    Constants also use the `$` prefix.
{{< /subtext >}}