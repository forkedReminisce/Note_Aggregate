---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



Confidentiality is that the VMs cannot see what other VMs are doing (functional security). Availability ensures the VMs can do what they want to do without unreasonable delay. Performance isolation. Trusted computing base (TCB) is how (the users of) VMs may trust other VMs and the hypervisor.   

Hacking containers can compromise the host system, but hacking a VM is stuck in said VM.

The prime and probe attack tries to get data from a VM on the cloud. Create an attacker VM and run it on the same physical machine as the target VM. Send traffic to the target VM, which will fill the shared processor cache (e.g., L3). Sensitive data might get stored in the cache. To make the cache only contain the sensitive data, the attacker VM will have a timing loop that will access some data in such a way that the only other data in the cache is it's own. The cache is a form of side channel.

<!-- is the hypervisor skipped or can the hypervisor just not be able to read it -->
<!-- sgx?? -->
Sometimes the hypervisor is not trusted. Therefore, trust the hardware. The guest application puts data to an enclave. The data gets encrypted there and sent to hardware. The guest OS needs to be modified to support this, but then the hypervisor can be skipped over.

Alternatively, the VM can be a trust domain. AMD SEV and Intel TeX. Whatever happens in the trust domain can only be interpreted by the VM that's in it. The Extended Page Table also doesn't work, so there has to be a reverse mapping table. Related paging.