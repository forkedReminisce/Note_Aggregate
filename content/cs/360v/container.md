---
draft: true
title: 

params: 
    desc: 
    author: Andrew Nguyen 
---



VMs have a stronger level of isolation since the shared level is the hypervisor. However, each VM needs its own guest OS. Containers tone down the isolation to the shared OS. Not only will the container be smaller, but it will also be less demanding. There is isolation between processes, but containers extends on that (e.g., PID isolation).

{{< subtext >}}
    Containers came about from developers wanting to ship their app with the exact correct dependencies without having to attach a large OS.

    More containers can be run on a physical machine than VMs by a factor of 10-100.
{{< /subtext >}}

Memory and scheduling have to go to the container layer, and the host OS just sees it as the container, not the processes inside the container. 

<!-- VISIBILITY --> 
Namespaces is how isolation is achieved with the host machine and even between containers. There are different types of namespaces for each kind of isolation. Creation and membership is separate; it is possible to create a namespace and add processes to it later. That is how the default, unnamed namespaces exist.  

The network namespaces is not lightweight; the network stack needs to be copied and allocated. Each namespace might have its own root user. However, the convention is that the root can do anything, which isn't compatible with containers. Capabilities are required tokens to do certain privileged actions. Without the capability, even if the user is root, it can't do the desired action.

<!-- RESOURCE MANAGEMENT -->
Control groups can restrict how much of a resource (e.g., memory) across its members can use. Other kinds of resource management can be achieved like which core to run the members on. This concept was invented separately from containers, but they are often joined together.

<!-- is the shared image THE overlay file system? -->
Every container has an image. It consists of the base layer—the OS—followed by layers of the OS plus *one* additional application (think committing after each step). Instead of shipping the final product, it is the base plus the following diffs. This is the essence of the overlay file system. That way, there can be a single shared image with containers having their layers point to layers of the shared image. Mismatched ordering of layers or a deviation of one layer does not disregard the shared image after; the container creates a new layer and continues trying to match layers. 

For writes, there is a private layer on top. This layer does not persist when the container terminates. If the data needs to be retained, it should be written to an external database.




# {{< heading "Container Orchestration" >}}
Container orchestration is about running a number of containers at once as a unit. There are multiple host machines and the containers needs to be distributed across them. The manager handles the assignment. Its goal is to keep the workload balanced across the machines. To facilitate this, each machine has a container management system. The manager pings the containers to make sure the micro-service it represents is still up; if it doesn't get a ping back, it assumes the container is dead. 

Kubernetes has an additional detail called the pod. The pod contains containers, and these containers must be instantiated at the same time on the same machine. The containers share the same namespace in the pod. This way, interprocess communication is better by skipping the network.