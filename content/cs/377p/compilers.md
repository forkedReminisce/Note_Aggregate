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
- disp(base, offset, scale): `num1`(%`reg1`, %`reg2`, `num2`) — `num1` + `reg1` + `num2` \* `reg2`

There is a way to declare constants so that its address is known at compilation. This is through a label: `label: .type value`. To use it in an instruction, prefix the label with a `$` like so: `$label`. 

{{< subtext >}}
    Constants also use the `$` prefix.
{{< /subtext >}}



<!-- TODO: new -->
# {{< heading "Compiler Optimizations" >}}
The front-end eventually produces an abstract syntax tree. The optimization phase has a high intermediate representation (IR) (trees) and low IR (closer to assembly) phases. The code is cycled through each phase multiple times.

Space or time optimizations must be safe. But even then, the compiler must figure out the right balance between space and time in the right context. 
- Constant propagation: if a variable acts like a constant, replace it with the constant
- Constant folding: evaluate an expression if operands are known at compile time
- Algebraic simplification: constant folding with chaining (if present)
- Copy propagation: if `x = y`, replace uses of `x` with `y`
- Common subexpression elimination: if the same expression appears multiple times, reuse the result
- Unreachable code elimination: omit unreachable code
- Dead code elimination: if the effect of a statement is never observed, omit it
- **Loop-invariant code motion**: hoist the computation of an expression out of a loop if it's the same every iteration
- **Strength reduction**: generally the act of replacing expensive operations with cheap ones
<!-- - Induction variable elimination: if there are multiple induction variables, eliminate the ones used only in the test condition (to continue looping) and rewrite it with the remaining induction variables -->
- **Loop unrolling**: modify loop to perform the body multiple times in a single iteration (of potentially multiple)
- Function inlining: replace a function call with that function's body
- Function cloning: create specialized versions of functions that reflects the particular call site's arguments

{{< subtext >}}
    Induction variables are variables in loops whose value depends on the iteration number.

    Loop unrolling eliminates a lot of condition branches, which isn't really relevant nowadays because branch predictors are very good.
{{< /subtext >}}