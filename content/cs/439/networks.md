---
draft: true
title: 

params: 
    desc: 
    author: FREEZURN 

# TODO: indexes MUST specify weight. the later the section, the higher the weight
---



Networks are a system of lines that interconnect. It's a hierarchical system organized by geographical proximity. It requires complex software because efficiency, correctness, and robustness matter. 

The OS treats networks like another device. The Network Interface Controller (NIC) is on the bus. 


# {{< heading "Communication" >}}
A symbolic name are Internet domain names. They are mapped to IP addresses, which the computer understands. Of the symbolic name, the first string (delimited by `.`) is the most specific part (i.e., the remote computer). It maps to the final number of the IP address. 

Each continent has its IP addresses isolated from the others. Despite this, though, continents are running out of 32-bit addresses, IPv4 (e.g., 128.83.139.82). IPv6 is replacing it (e.g., fe80::46a8::42ff:fe44:a726), but most traffic still occurs on IPv4. 

The Internet maintains the mappings in a huge worldwide distributed database called Domain Naming System (DNS). It's organized as a trie from the first-level domain names (`com`). Our computers are the resolver, and they send symbolic links it doesn't recognize through a system call that queries the local DNS server. This local DNS server will make requests to each level of domain names, starting at the root DNS, to construct more precise IP addresses. ==Like the resolver, the local DNS server caches responses==.

Information is transmitted through Gateways. Which Gateways are determined by the destination IP address, starting from the left number. Transmission is through units called *packets*, the size of which (Maximum Translation Units) determined by the networks.

<!-- layers 2 and 3. layer 1 is hardware (wires or the machines themselves) -->
Because there's multiple different, incompatible networks (e.g., ethernet and wireless), internet protocols serve to get to the right network of the destination machine. When data enters the internet layer, it gets a header and becomes whatever package the transport protocol uses. Local internet protocols are specific to the network and get to the destination machine itself. When the packet is here, it gets a frame header tacked on and becomes a frame. Routers connect networks to each other, and they replace frame headers. This is the essence of internet. 

{{< subtext >}}
    Global IP Internet is the most famous example of an internet and what we typically think of (IP addresses).
{{< /subtext >}}

Networks can be classified as:
- System Area Network (SAN): for connecting a machine room; fast (fibre)
- Local Area Network (LAN): computers in a single building; reliable (ethernet)
- Wide Area Network (WAN): computers across state, country, or planet

There are a number of time costs associated with networks. Latency is the time it takes for one byte to go from one place to another physically. Throughput is how many bytes can be sent per second. There's also overhead: the time it takes for the source and destination machines to prepare or pull data from the packet.


<!-- layer 4 -->
## {{< heading "Transport" >}}
The TCP/IP Protocol Family concerns reliable transport. User Datagram Protocol (UDP) provides unreliable delivery, meaning if a packet gets lost (e.g., full buffer) or corrupted, UDP doesn't care and makes the application detect it. It sends datagrams. Transmission Control Protocol (TCP) tries to mask unreliability of the network. It sends segments. To do what it does, it will set up a session between the client and server through the Three-Step Handshake.
1. Client sends an `SYN` message for a synchronous connection
2. Server responds with `SYN`/`ACK` to acknowledge and accept the request
3. Client sends back an `ACK`

The sender will now send a segment and start a timer. The receiver will receive the segment and return an `ACK`. If the sender doesn't get this `ACK` before the timer goes off, it resends the data. Senders and receivers are processes, and `ACK`s are not handled until the process gets the data that resides in the TCP's buffer. When the sender has multiple outstanding segments, an `ACK` for segment $i$ will serve for segments $i$ and below. `NACK` allows the receiver to specify which segments in that range was not received, which allows the sender to resend only those segments. This is cumulative acks.

{{< subtext >}}
    Delayed acts use application response as implicit `ACK`.
{{< /subtext >}}

*TCP Flow Control* is about sending most amount of segments without overwhelming the receiver. Overwhelming as in overflowing its buffer (RcvBuffer). The sender knows the size of the buffer (RcvWindow) thanks to the handshake. It also knows how much data it sent and how much of it was `ACK`'ed, allowing the TCP protocol to optimize the number of segments to send. 

The *TCP Congestion Window* sets the maximum number of bytes that can be sent without an `ACK`. This is to avoid congestive collapse, where the network is already congested but data keeps getting sent through it, bringing the network down further. The Congestion Control algorithm tries to figure out the size of the window based on:
- Additive increase, multiplicative decrease: window will grow by 1 every cumulative ack but halves at loss event (no `ACK` or `NACK`)
- Slow start: window starts 1, and for each `ACK` the window doubles, until the first loss event
- Reaction to timeout events: if the timer runs out without an `ACK`, the window is set to 1 and slow mode mode until the window reaches some threshold
- Round trip variance estimation: initially set timer to estimated round trip time plus quadruple the average deviation of round trip time
- Exponential retransmit timer backoff: if the timer runs out, it is doubled

TCP is implemented by the OS. It manages its metadata in a Protocol Control Block (PCB). There is one for each connection, so a process may hold multiple PCBs. It does have to decide when to start sending/returning data (i.e., if it's small, should it wait for more).

<!-- sockets are per connection -->
Processes are identified with a port, a 16-bit integer. Well-known ports are for common services (e.g., web), while ephemeral ports are assigned automatically and what a process would typically have. The endpoints of a connection are called sockets. Its address is `IPAddress:Port`. To an application, sockets are just somewhere to read or write to regarding networks.

When the client wants to communicate with a server, it must `getaddrinfo()` the server to query the DNS. It will then create a socket descriptor. On the kernel side, a data structure is created that includes the pointer to the protocol control block. Then it will connect to the server, which is blocking (until `connect()` returns).

The server will also `getaddrinfo()` on the client and create a socket descriptor. However, it must then associate this socket descriptor with a socket address. `listen()` will then make this socket serve the purpose of taking in connection requests from clients. Finally, it will call `accept()` to block until a connection request is made. This function will create a new thread and returns to it a connected descriptor, which is what the client and server will use to communicate; the listening descriptor continues to handle new connection requests. When the client disconnects, it signals the server to do the same.