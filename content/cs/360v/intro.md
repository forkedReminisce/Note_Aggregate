---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



<!-- TODO: verify final sentence -->
Popek and Goldberg defined a set of standards for virtualization. The virtual machine monitor, or hypervisor, is the interface to hardware. It must have resource control in order to fairly allocate resources. The "user" of the hypervisor is the virtual machine. It should be that each core runs a VM, and that each VM has the same properties as the physical machine it runs on.

<!-- TODO: is this only for emulation??? -->
A method of virtualization was hosted interpretation. When host architecture is emulating target architecture, every instruction had to go through a translation. This was robust but greatly inflated the actual number of instructions. An optimization would be to only translate privileged instructions, but the x86 architecture had a category of instructions known as sensitive, which behave like privileged instructions but without a trap—the trigger for intervention. Therefore, binary translation would convert the sensitive instruction into a sequence of privileged instructions. 

<!-- TODO: -->
Hardware-assisted virtualization allows for virtualization at the highest efficiency. Paravirtualization is virtualization on top of a virtual machine.