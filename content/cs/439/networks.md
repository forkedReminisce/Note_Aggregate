---
draft: false
title: Networks

params: 
    desc: Networks rely on various components for the purposes of efficiency, correctness, and robustness. 
    author: Andrew Nguyen 
---



People are most familiar with symbolic names (e.g., `bbc.co.uk`). In the Global IP Internet (hereafter referred to as "Internet"), has a distributed database called **Domain Naming System** (DNS). This is how symbolic names are mapped to IP addresses. 
1. Resolver queries its local DNS server with a symbolic name through a *system call*
2. Local DNS server queries the root DNS with the final token of the symbolic name (e.g., `uk`).
3. Root DNS returns the IP address of the DNS server of the token
4. Local DNS server queries this server with the previous token (e.g., `co`).
5. Repeat steps 3 and 4 until the IP address for the first token is reached

{{< subtext >}}
    The resolver and local DNS server do cache responses.

    The first numbers of an IP address is used to geographically group IP addresses. This speeds up data routing.
{{< /subtext >}}

Networks do have their issues. 
- Latency: time it takes for one byte to go from one place to physically make it to the other. 
- Bandwidth: capacity that can be out in network at a time. 
- Overhead: Time for machines to prepare or retrieve data 



# {{< heading "Seven Layers" >}}
Layer 1 is hardware. The Network Interface Controller (NIC) is the device that handles network communication. It shares the same bus as many other devices, like HDDs. 

There are multiple different, incompatible networks (e.g., ethernet and wireless). These are physically connected with routers. Internet protocols are responsible for getting a message to the destination machine, which may be on a different network. This is the essence of internet.

Layer 3 is network. When data enters the internet layer, it gets a packet header attached to it. This is metadata about the transmission. Here, the data is known as a *packet*. 

{{< subtext >}}
    Each network defines a Maximum Transmission Unit for the maximum size of a packet.
{{< /subtext >}}

Layer 2 is data link. After getting a packet header attached, the data then gets a frame header of the network. When it passes through a router, the frame header gets replaced with one appropriate for the other network. The data is called a *frame*.


## {{< heading "Transport" >}}
Layer 4 is transport. This is how processes send and receive data. The *User Datagram Protocol* (UDP) provides unreliable delivery. This puts the responsibility on the sender if a datagram gets lost or corrupted.

**Transmission Control Protocol** (TCP) approaches things differently by handling unretrieved segments itself. First, a session must be set up between the client and server through a *Three-Step Handshake*.
1. Client sends an `SYN` message for a synchronous connection
2. Server responds with `SYN`/`ACK` to acknowledge and accept the request
3. Client returns an `ACK`

{{< subtext >}}
    Datagrams and segments are the results of attaching another header.
{{< /subtext >}}

<!-- TODO: whats the point of NACK if the sender can just resend segments there's not an ACK for -->
The sender will now send a segment and start a timer. When the receiver retrieves the segment from the TCP buffer, it will send back an `ACK`. If the sender doesn't get this `ACK` before the timer goes off, it resends the segment. When the sender has multiple outstanding segments, an `ACK` for segment $i$ will serve for segments $i$ and below (cumulative ack). `NACK` allows the receiver to specify which segments in that range was not received, which allows the sender to resend only those segments. 

{{< subtext >}}
    Delayed acks use application response as implicit `ACK`.

    Segments can be received out of order.
{{< /subtext >}}

<!-- TODO: is the TCP buffer per process -->
*TCP Flow Control* is about sending most amount of segments without overwhelming the receiver (i.e., overflowing its TCP buffer). The sender already knows the size of the buffer thanks to the handshake. It also knows how much of the receiver's TCP buffer is filled based on how many segments were sent versus how many have been `ACK`'d. 

The *TCP Congestion Window* sets the maximum number of bytes that can be sent without an `ACK`. This is to avoid congestive collapse, where the network is already congested but data keeps getting sent through it, bringing the network down further and potentially leading to packet loss. The Congestion Control algorithm tries to optimize the size of the window based on:
- Additive increase, multiplicative decrease: window will grow by 1 every cumulative ack but halves at loss event 
- Slow start: window starts 1, and for each `ACK` the window doubles, until the first loss event
- Reaction to timeout events: when the timer runs out, the window is set to 1 and goes into slow start until the window reaches some threshold
- Round trip variance estimation: initially set timer to estimated round trip time plus quadruple the average deviation of round trip time
- Exponential retransmit timer backoff: if the timer runs out, it is doubled

TCP is implemented by the OS. Metadata is stored in a Protocol Control Block (PCB). There is one for each connection, so a process may hold multiple PCBs.

# {{< heading "Sockets" >}}
In a particular system on the network, a process is identified by a *port*, a 16-bit integer. Well-known ports are reserved for common services, while ephemeral ports are assigned automatically. The endpoints of a connection are called **sockets**. Its address is `IPAddress:Port`. To an application, sockets behave like file descriptors.

When the client wants to connect to a server, it must `getaddrinfo()` the server. It will then create a socket descriptor with `socket()`. Then it will connect to the server, which is blocking (until `connect()` returns).

<!-- TODO: is listen() blocking -->
If it doesn't already know itself, the server will `getaddrinfo()`. It will then create a socket descriptor and call `bind()` to make the socket discoverable over the network. `listen()` will make the running thread block until a connection request comes in. At which, `accept()` makes a new thread and *connected descriptor*, which is the socket the server will use to communicate with this client. The older socket is the *listening descriptor*. 

When the client disconnects, it signals the server to do the same.