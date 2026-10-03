---
draft: false
title: An Aside On Security

params: 
    desc: VMs run on trust, or lack thereof. Thankfully, there are hardware solutions.
    author: Andrew Nguyen 
---



<!-- is it the user or OS developer that sets TCB? what is TCB really? -->
An important aspect for virtual machines is confidentiality. When VMs cannot see what other VMs are doing, functional security is achieved. There is the notion of the Trusted Computing Base to note trust of other VMs and even the hypervisor.

Cloud computing relies on VMs. Therefore, a client may be targeted with the prime and probe attack. The attacker can request an (attacker) VM and get it to run on the same physical machine as the client's (target) VM. Since it is the same machine, there is a common cache. By sending traffic to the target VM, its sensitive data can make it onto this cache. With a timing loop, the only data in the cache will now be the attacker's and said sensitive data.

{{< subtext >}}
    The cache is a form of side channel.
{{< /subtext >}}

If even the hypervisor cannot be trusted, it can be kept out of observing operations with the use of enclaves. These are regions of the physical address space where the data is encrypted. The data gets decrypted at the last possible moment, preventing the hypervisor from examining it. The guest OS does need to be modified to support this.

<!-- EPT doesn't work, so reverse mapping table and related paging. whatever that means -->
Alternatively, the VM can be a trust domain. Whatever happens in the trust domain can only be interpreted by the VM that's in it. 