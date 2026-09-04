---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- TODO: verify final sentence -->
Popek and Goldberg defined a set of standards for virtualization. The virtual machine monitor, or hypervisor, is the interface to hardware. It must have resource control in order to fairly allocate resources. The "user" of the hypervisor is the virtual machine. It should be that each core runs a VM, and that each VM has the same properties as the physical machine it runs on.

<!-- for emulation -->
A method of virtualization was hosted interpretation. When host architecture is emulating target architecture, every instruction had to go through a translation by the VMM. All data is stored in software, not physically. This was robust but greatly inflated the actual number of instructions. Trap and emulate is an optimization that only translate privileged instructions after falling into a trap. 

The x86 architecture had a category of instructions known as sensitive, which can modify physical resources without a trap. Therefore, binary translation would convert the sensitive instruction into a sequence of privileged instructions. In processing the program, translation blocks are made of separate sequences of straight unprivileged, privileged, or sensitive code. In execution, as a privileged or sensitive block is entered, it are translated. If the block has been translated already, it's in the translation cache. Traps are now no longer necessary; in fact, this is faster despite having to translate in realtime.

<!-- guest OS != host OS -->
After hardware manufacturers recognized virtualization, hardware-assisted virtualization came. It allows for virtualization at the highest efficiency. It is the gold standard for today. Intel came out with VT-X, extensions that made virtualization easier. Before VT-X, the guest OS would be placed a ring above the hypervisor but still below the guest application. VT-X allowed the guest OS to be in the same ring as the hypervisor, but not in root mode. The optimization is that the guest OS can read the CPL directly, which is performed frequently, and it will trap less frequently. Additionally, VT-X allows a developer to define the causes for a trap.

<!-- TODO: is the VMCS exclusive to VT-X? idk about the control register -->
The VMM can be thought of as an OS in that it has to manage multiple virtual machines at once, akin to processes. Each virtual machine has a virtual machine control structure (VMCS). It consists of guest state, host state, and control register. Guest state describes the contents of the virtual registers, host state are the physical registers, and the control register defines what to trap on and what to exit the virtual machine on. Hardware has dedicated, separate set of physical "virtual" registers for virtualized machines; VMs cannot assess the root registers. These can be modified without a trap since they're isolated from the registers that matter (root). The context switch is also performed by hardware.

Para virtualization is virtualization on top of a virtual machine.