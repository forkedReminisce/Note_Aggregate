---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



While VMs have their common level at that hypervisor, containers have theirs at the host OS level. This means that containers have weaker isolation. However, the benefit is that containers are smaller in size and resource demands. There's also the added benefit of CPU scheduling and memory access functioning just like the container being just another process.

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
    The overlay file system is copy-on-write; the package layers are read-only.
{{< /subtext >}}

<!-- private layer alternatively called the mutable working directory -->
On top of the package layers is the *private layer*. If the container ever needs to write to a file system, it can write to the private layer. However, data written is not persistent; the private layer is thrown away when the container terminates. If some of the written data needs to be retained, write it to a persistent volume (i.e., some external storage). It is copy-on-write; if a file in a layer is written to, the file is copied-up to the mutable.

Finding files requires going through each layer top-to-bottom. In other words, if the file exists in multiple layers, the topmost is read. Remember, the layers are like git commits that evolve the file system one layer at a time.



# {{< heading "Container Orchestration" >}}
<!-- containers represent micro-services -->
With multiple containers that need to run as a unit and multiple machines, it is the **manager's** job to assign containers to machines. Its goal is to keep the workload balanced across the machines. To facilitate assignment, each machine has a *container management system*. The manager also directly pings the containers to make sure they are still up.

Kubernetes has an additional detail called *pods*. Containers in a pod must be instantiated at the same time ==on the same machine==. Additionally, the containers will be in the same namespace. This way, interprocess communication can skip the network and, therefore, be better.

For containers on the same machine to communicate between each other, a virtual ethernet interface (veth) is necessary. To communicate with containers (or anything for that matter) outside this machine, a bridge is necessary.



# {{< heading "Open Container Initiative" >}}
<!-- OCI commands are used by Kubernetes to create containers -->
Kata containers contains an OCI package. Kubernetes interacts with the Kata runtime to create containers and other things. The OCI is like an interface, and the Kata runtime implements the OCI. These OCI libraries implement interfaces like the container refine interface (CRI) and container network interface (CNI).



# {{< heading "gVisor" >}}
When the container makes a system call, it usually just goes to the host OS. For security, the container is usually wrapped in a VM. Alternatively, gVisor has the container sit on top of an alternate layer that is a user-space kernel called the gVisor Sentry. If the gVisor Sentry cannot handle it does the system call go to the host OS. 