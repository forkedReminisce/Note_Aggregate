---
draft: false
title: Improving The Pipeline

params: 
    desc: CS 429 introduced the pipeline, but it is possible to improve it with parallelism and better branch prediction strategies.
    author: Andrew Nguyen 
---



Pipelining features two main types of **dependencies**: data and control. Data dependencies can further be categorized:
- Flow dependence: read after write (RAW)
- Anti dependence: write after read (WAR)
- Output dependence: write after write (WAW)

Dependencies indicated an execution order that needed to be followed. Furthermore, **precise exceptions** are important in debugging situations. This states that when an exception is thrown, the state must appear as though no instructions after the exception-throwing one was executed.



# {{heading "Out-of-order Execution"}}
**Superscalar** introduces the idea of multiple parallel pipelines. However, dependencies present a challenge in that if one pipeline gets halted, all pipelines halt. Therefore, **out-of-order execution** (or in-order completion \\[commit/retire\\]) is required. 

In order to maintain precise exceptions, an architectural and speculative state must be maintained. Instructions will be fetched and decoded in-order, but will then execute in the **reorder buffer** (ROB) out-of-order in the speculative state. Instructions will then retire from the ROB in order, updating the architectural state. 

There are also two sets of registers: architected and physical. Architected registers are those visible to the ISA and programmer, while physical registers comprise many more registers that support architected registers. **Register renaming** uses these to increase parallelism. Dataflow execution only retires and updates registers in-order, which can *unnecessarily* keep consumers and other writers waiting. Register renaming also eliminates WAR and WAW. 

All together, when an instruction is let into the ROB, it will:
1. Check the architected register file, indexing into each index it will read
   - If it is valid, no instruction in flight is writing to it, so the value can be read
   - If it isn't valid, follow the stored index into the architected register file
     - If that is valid, read the value
     - If it isn't valid, join the consumers list
2. Obtain a free physical register number
3. On the architected register file, set the valid bit to false and update the physical register number
4. Index into the physical register file and set the valid bit to false
5. Execute
6. Write the value and set the valid bit to true on the physical register file and notify all consumers
7. Retire: on the architected register file, update the value and, if the physical register number is the same, the valid bit



# {{< heading "Branch Prediction" >}}
Although ROBs can fit around 200 instructions, taking the wrong branch too often makes this moot. This is especially true when analysis found that branches make up about 1 in every 5 instructions.

The IF stage contains the BPU to perform the predicting. The \(n\)-bit saturating counter (or bimodal predictor) is a state machine with \(2^n\) states. This set is divided into two such that one half corresponds to predicting branch not taken and the other half otherwise. As branches are evaluated, the current states moves one step closer or farther away from the transition point. This is not the only branch predictor, though.