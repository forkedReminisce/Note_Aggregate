---
draft: false
title: What Is An OS?

params: 
    desc: An operating system (OS) serves three roles—referee, illusionist, and glue. To evaluate an OS, there are some criteria one may check.
    author: FREEZURN 
---



An operating system manages the computer's resources. It makes life better for both the developer and regular user.

The OS plays three big roles. As the **referee**, it coordinates between many applications to give each fair access to resources while the computer stays efficient. Isolating programs from each other prevents a crash from disrupting the entire system. When necessary, it will facilitate communication between applications.

**Illusionist** describes the use of virtualization to create the illusion of infinite resources to the user. This is why it may seem that the processor is doing many things at once when it really can only do one-at-a-time.

The **glue** simplifies application design by providing standard interfaces to the hardware. 



# Evaluation
**Reliability** concerns whether the OS is predictable; it does what it is designed to do. This is measured by the percentage of time the system is usable versus when it is experiencing a bug and troubleshooting. 

**Security** is about the access of users' data. This can come under threat to an attacker or even just careless applications. Policies are put in place to ensure security, and enforcement mechanisms describe how that policy is upheld. 

<!-- also helps with keeping complexity low and provide protection between applications -->
As a way to look after the future, **portability** consists of three interfaces. The *Abstract Machine Interface* (AMI) has the set of legally executable instructions, the memory access model, and the API. The *Application Programming Interface* (API) consists of system calls so that applications can use OS services. These OS services do not need to change as hardware changes and advances. The *Hardware Abstraction Layer* (HAL) has hardware-specific software to allow devices to communicate with the system. As the name implies, these change according to hardware. Drivers and such implement functions with a standard signature to the benefit of the AMI or API. Portability is also concerned about applications themselves. 

**Performance** is measured by many criteria. *Efficiency* is how much overhead the OS takes up (the OS is overhead itself in the grand scheme of things). *Fairness*, *throughput*, and *predictability*. *Response time* is about how long it takes until the user gets a response. Some criteria may not be relevant to some OS's.