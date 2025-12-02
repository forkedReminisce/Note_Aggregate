---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



A system must be designed so to be safe, even if it is used improperly. Security is a policy, and protection is the mechanism that implements this policy.



# {{< heading "Design Principles" >}}
- Economy of Mechanism: a simple design that reuses well-tested mechanisms is secure enough
- Fail-Safe Defaults: too little is better than too much; there will be an alert
- Complete Mediation: secure the system as a whole
- Open Design: assume potential intruders already understand everything about the design
- Separation of Privileges (e.g., group, root)
- Least Privilege: initially give minimum access rights required to perform a task
- Psychological Acceptability: so simple that people don't care to disable it
- Physical Security (e.g., keys)
- Educate the humans: counteract social engineering



# {{< heading "Authentication" >}}
Authentication is a mechanism. It's the idea that a multi-user system needs to verifiably tell who is currently logged in. Most often, this is done with *passwords*. 

The system only stores encrypted passwords. Then to check the input, the input is encrypted and compared against what is stored. Encryption is performed with a one-way hash. That means that the encryption key can do but not undo. 

However, if two people use the same password, the same encryption value will be produced. Therefore, each user gets a unique salt that gets encrypted with the password or input. Of course, there's still the issue of sending the password over the network and humans in general. 



<!-- TODO: it's hard to detect and resecure after a breach -->
# {{< heading "Security Attacks" >}}
- Trojan House: a code fragment that is not what it looks like
- Trap Door or back door: programmer built in a way to circumvent normal security procedures
- Logic Bomb: program that will cause a security incident under specific circumstances (e.g., a particular time)
- Stack and Buffer Overflow: exploit a bug that will allow input code to execute
- Rootkit: set of programs or files that tries to hide itself

Sony once sold music CDs that came with a rootkit against file sharing. It did this by watching traffic to and from the I/O device. It consumed a lot of resources and crashed the computer all the time. The public was furious and Sony had to recall the CDs and distribute rootkit removal software (after some trouble).

Ken Thompson made a Unix back door by modifying the compiler of the compiler. This "top" compiler will always insert the back door code into the "bottom" compiler, which inserts it wherever necessary (i.e., `login.c`). The moral of the story is that we cannot trust the code that we write unless we trust every tool in the toolchain.  

{{< subtext >}}
    The back door exploits the fact that a program can output its source code, but a compiler may output incorrect machine code.
{{< /subtext >}}

Stack and Buffer Overflow happen because of copying data of unchecked size. The attackers can figure out the address space layout. When data is being copied to the stack, it's possible to strategically modify the return address to somewhere in the stack. That way, input code can be executed.

Viruses are code fragments embedded in a legitimate program. They're specific to architecture, OS, or a particular application. They're very difficult to remove from a system. For example, a boot sector virus would replace the boot sector (it's been copied elsewhere), made it so that it used unused memory (by lying about the available memory) and blocked any attempts to write to the boot sector.

Unlike viruses, worms are standalone. Robert Morris Jr. created the first worm that broke passwords and took down the internet for two weeks. Stuxnet was initially spread on a USB drive that then used private networks to infect other computers. It targeted Siemens' control software, leading to the destruction of centrifuges. 