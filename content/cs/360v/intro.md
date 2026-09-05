---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- TODO: verify final sentence -->
Popek and Goldberg defined a set of standards for virtualization. The virtual machine monitor, or hypervisor, is the interface to hardware. It must have resource control in order to fairly allocate resources. The "user" of the hypervisor is the virtual machine. It should be that each core runs a VM, and that each VM has the same properties as the physical machine it runs on.

A method of virtualization is hosted interpretation. When host architecture is emulating target architecture, every instruction had to go through a translation by the VMM. All data is stored in software, not physically. This was robust but greatly inflated the actual number of instructions. Trap and emulate is an optimization that only translates privileged instructions after triggering a trap. 

The x86 architecture had a category of instructions known as sensitive, which can modify physical resources without a trap. Therefore, binary translation would convert the sensitive instruction into a sequence of privileged instructions. Before execution, there is a preprocess stage that partitioned the code into translation blocks—sequences of only unprivileged, privileged, or sensitive instructions. In execution, if a privileged or sensitive block is entered, it is translated. If the block has been translated already, it's in the translation cache. This is actually faster than trap and emulate.

After hardware manufacturers recognized virtualization, hardware-assisted virtualization came. It allows for virtualization at the highest efficiency, even today. Intel came out with VT-X, extensions that made virtualization easier. Before VT-X, the guest OS would be in a ring above the hypervisor. VT-X introduced root mode for that bottom ring, allowing the guest OS to be in the same ring as the hypervisor without the privileges that come with it. The optimization is that the guest OS can read the CPL directly, which is performed frequently, and trap less frequently. Additionally, VT-X allows a developer to define what causes a trap.

<!-- TODO: is the VMCS exclusive to VT-X? idk about the control register -->
The VMM can be thought of as an OS in that it has to manage multiple virtual machines at once. Each virtual machine has a virtual machine control structure (VMCS). It consists of guest state, host state, and the control register. Hardware introduced a dedicated, separate set of physical "virtual" registers for virtualized machines; VMs are still not allowed to access root registers. These "virtual" registers can be modified without a trap. The context switch is now also performed by hardware, increasing efficiency.

Para virtualization is virtualization on top of a virtual machine.