---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



While VMs have their common level at that hypervisor, containers have theirs at the host OS level. This means that containers have weaker isolation. However, the benefit is that containers are smaller in size and resource demands, and containers are treated as just another process, improving CPU scheduling and memory access. 

{{< subtext >}}
    More containers than VMs can be run on a physical machine by a factor of 10-100.
{{< /subtext >}}



# {{< heading "Isolation" >}}
Isolation between containers is achieved with three main technologies. These technologies have existed before the advent of containers.

**Namespaces** create a new "view" of a world. These worlds include mount and PID. There is always a default namespace for each world.

{{< subtext >}}
    The network namespace is not lightweight because the entire network stack needs to be allocated and copied over.
{{< /subtext >}}

One namespace worth touching on is the user namespace. With a new user namespace, it's possible for a process in a container to think it is root. However, the convention is that the root can do anything, which isn't appropriate for containers. Therefore, every privileged action is locked behind a *capability*.

**Control groups** facilitates resource management across its members.

<!-- is the shared image THE overlay file system? -->
An image can be thought of as a package that the container runs. It consists of layers representing each installation of a package (i.e., diffs). If all the containers' packages were installed, the resultant image can act as the **overlay file system**. What this means is that instead of each container having its own separate image, they can instead possess pointers to the corresponding diff in the "central" image for each of its packages. If the overlay file system doesn't have the package, the container can just make a new layer on the overlay file system.

{{< subtext >}}
    The package layers are read-only.
{{< /subtext >}}

On top of the package layers is the mutable working directory. If the container ever needs to write to a file system, it can write to this layer. As the overlay file system is copy-on-write, writing to a file copies-up to the mutable. However, the data written is not persistent. If the written data needs to be retained, write it to a persistent volume (i.e., some external storage). 

Finding files requires going through each layer top-to-bottom. In other words, if the file exists in multiple layers, the topmost is read. Remember, the layers are like git commits that evolve the file system one layer at a time.



# {{< heading "Container Orchestration" >}}
<!-- containers represent micro-services -->
With multiple containers that need to run as a unit and multiple machines, it is the **manager's** job to assign containers to machines. Its goal is to keep the workload balanced across the machines. To facilitate assignment, each machine has a *container management system*. The manager also directly pings the containers to make sure they are still up.

Kubernetes has an additional detail called *pods*. Containers in a pod must be instantiated at the same time ==on the same machine==. Additionally, the containers will be in the same namespace. This way, interprocess communication can skip the network and, therefore, be better.

For containers on the same machine to communicate between each other, a virtual ethernet interface (veth) is necessary. To communicate with containers (or anything for that matter) outside this machine, a bridge is necessary.



# {{< heading "Small Details" >}}
The Open Container Initiative (OCI) essentially provides a layer of abstraction between container management software and the actual implementation of creating containers and the like. These OCI commands are implemented by container runtimes, and a particular runtime may be used for a particular goal. Such runtimes will implement the commands in a particular way. However, every runtime can only differ so much because the OCI outlines the implementation of a number of interfaces.

<!-- whats the benefit -->
gVisor is an alternative to running the container on a VM. On a VM, when the container makes a system call, the guest OS will handle it until it can't anymore. With gVisor, the system call is sent to the gVisor Sentry, a user-space kernel. Only when necessary does the system call go to the host OS.