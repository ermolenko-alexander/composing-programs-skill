# Section 4.6: Distributed Computing

## Core Idea
Large-scale data processing applications often coordinate effort among multiple computers. A **distributed computing application** is one in which multiple interconnected but independent computers coordinate to perform a joint computation. The computers are independent in that they **do not directly share memory**; they communicate using **messages** — information transferred from one computer to another over a network — governed by agreed protocols.

## Formal Protocols

### Message protocols
Messages are sequences of bytes; they can request data, send data, or instruct another computer to evaluate a procedure call. The sender must encode information so the receiver can decode and correctly interpret it. A **message protocol** is a set of rules for encoding and interpreting messages; both sides must agree on the semantics. Two common designs: fixed formats where certain bits at fixed positions indicate fixed conditions, and special delimiter bytes "much as punctuation delimits sub-expressions in the syntax of a programming language." Crucially, **message protocols are not particular programs or software libraries** — they are rules applicable by a variety of programs, even in different languages; computers with vastly different software systems participate in the same distributed system simply by conforming.

### IP (Internet Protocol)
Specifies how to transfer **packets** of data among different networks for global communication. Designed under the assumptions that networks are **inherently unreliable at any point and dynamic in structure**, with **no central tracking or monitoring**. Each packet carries a header with the destination IP address; packets are forwarded by simple routing rules on a **best-effort basis**. Constraints: packets (IPv4/IPv6) have a **maximum size of 65,535 bytes** (larger values split across packets); IP does **not** guarantee order; packets may be **lost** or **transmitted multiple times**.

### TCP (Transmission Control Protocol)
An abstraction defined in terms of IP providing **reliable, ordered transmission of arbitrarily large byte streams**, achieved by correctly ordering packets, removing duplicates, and requesting retransmission of lost ones. The improved reliability "comes at the expense of latency." TCP breaks the stream into **TCP segments**: a portion of data preceded by a header with sequence and state information; some segments carry no data and only establish or terminate a connection.

**Three-step handshake** between computers A and B:
1. A sends a request to a **port** of B to establish a TCP connection, providing a **port number** to which to send the response.
2. B sends a response to the port specified by A and waits for its response to be acknowledged.
3. A sends an acknowledgment response, verifying that data can be transferred in both directions.

After the handshake, data flows both ways. Termination is a sequence of steps in which both client and server request and acknowledge the end of the connection.

### HTTP
The Hypertext Transfer Protocol is implemented using TCP and governs communication for the WWW, assuming a client/server architecture between browser and web server; it specifies the format of all request and response messages. The most common request type is **GET**, which specifies a location: typing `http://en.wikipedia.org/wiki/UC_Berkeley` issues an HTTP GET to **port 80** of the web server at `en.wikipedia.org` for the contents at location `/wiki/UC_Berkeley`. Response:
```
HTTP/1.1 200 OK
Date: Mon, 23 May 2011 22:38:34 GMT
Server: Apache/1.3.3.7 (Unix) (Red-Hat/Linux)
Last-Modified: Wed, 08 Jan 2011 23:11:55 GMT
Content-Type: text/html; charset=UTF-8

... web page content ...
```
A wrong address yields a header starting `HTTP/1.1 404 Not Found`. 200 and 404 are **HTTP response codes**; a fixed set of response codes is a common feature of a message protocol — designers anticipate common messages and assign fixed codes "to reduce transmission size and establish a common message semantics."

## Architectures

### Client/Server
A way to dispense a service from a central source: a **server** provides a service; multiple **clients** communicate with it to consume the service. Roles differ: the server responds to requests; a client issues requests and uses responses to perform some task.

**Modularity.** Client and server are powerful abstractions: a server may serve many clients simultaneously; clients need not know how the service is provided or how data is stored/calculated; the server need not know how its responses will be used. The roles are not tied to separate machines: programs are clients of the OS's device drivers (servers turning physical input signals into usable data), and the CPU is often a client of the GPU as a server of images.

**Drawbacks.**
- The server is a **single point of failure** — the only component able to dispense the service; clients are interchangeable and can come and go.
- Resources become **scarce with too many clients**: clients increase demand without contributing any computing resources.

### Peer-to-Peer
For computational goals better served by an equal division of labor. In a P2P system labor is divided among **all** components: every computer sends and receives data and contributes processing power and memory — so as the system grows, its capacity grows. Division of labor among all participants is **the identifying characteristic** of P2P. Peers must communicate reliably, so P2P systems need an organized network structure; components cooperate to maintain enough information about the locations of others to deliver messages.

Some systems delegate network health to specialized components — then they are **not pure P2P** (different component types, different functions). These components act like **scaffolding**: keep the network connected, maintain location information, help newcomers take their place in their neighborhood.

**Common applications**: data transfer and data storage. For transfer, each computer helps forward data through its neighborhood. For storage, the data set may be too large to fit on a single computer or too valuable to store on just one; each computer stores a small portion, with multiple copies spread across machines — when a computer fails, its data is restored from other copies when a replacement arrives.

**Skype example**: a data-transfer P2P application. Each computer knows a few neighbors; a packet travels neighbor-to-neighbor to its destination. Skype is not pure P2P: a scaffolding network of **supernodes** handles logging users in and out, maintaining location information, and modifying the network structure as users enter and exit.

## Code Examples
```python
>>> from socket import gethostbyname
>>> gethostbyname('www.nytimes.com')
'170.149.172.130'
>>> from urllib.request import urlopen
>>> response = urlopen('http://www.nytimes.com').read()
>>> response[:15]
b'<!DOCTYPE html>'
```
The two client requests behind viewing a web page: a DNS lookup, then an HTTP GET to the web server.

## Worked Example: Requesting a web page
1. The browser (client) requests the IP address for www.nytimes.com from a **Domain Name Server (DNS)** — a server whose service is mapping domain names to IP addresses (numerical machine identifiers).
2. It then requests the page contents from the web server at that address. The response is an HTML document containing the day's headlines and excerpts plus layout expressions.
3. Upon receiving the response, the browser issues **additional requests** for images, videos, and other auxiliary components, because the HTML contains their addresses and how they embed in the page.
4. Underneath: a TCP three-way handshake established each connection, and TCP ordered, deduplicated, and retransmitted the IP packets carrying the data.
5. A mistyped address produces `HTTP/1.1 404 Not Found` rather than `200 OK`.

So one page view involves at least two different servers (DNS + web), several protocol layers (IP → TCP → HTTP), and a cascade of client requests.

## Reference Table: Client/Server vs Peer-to-Peer

| | Client/Server | Peer-to-Peer |
|---|---|---|
| Division of labor | asymmetric roles | labor divided among all components |
| Capacity as system grows | demand grows, resources don't (clients contribute nothing) | capacity of computational resources increases |
| Failure | server is a single point of failure | data restored from replicas on peer failure |
| Structure maintenance | trivial (one known server) | organized network structure; possibly scaffolding/supernodes |
| Best for | service-oriented situations (Web, drivers, CPU/GPU) | data transfer and data storage at scale |

## Key Concepts
- **Message**: byte sequence transferred over a network; the only inter-machine communication (no shared memory).
- **Message protocol**: encoding/interpretation rules; shared semantics, not shared code.
- **Packet**: IP's best-effort unit, header with destination address, ≤ 65,535 bytes.
- **TCP segment**: data portion + sequence/state header; some segments only open/close connections.
- **Port / port number**: addressable endpoint for requests and responses.
- **DNS**: the domain-name → IP-address service.
- **GET request / response code**: HTTP's page request; fixed codes (200 success, 404 not found).
- **Single point of failure**: the unique dispenser of a service.
- **Supernode / scaffolding**: specialized components maintaining a non-pure P2P network.

## Mental Models
- A protocol is an interface contract between machines — conformance, not shared software, enables interoperation of heterogeneous systems.
- TCP is reliability purchased with latency on top of IP's "best effort, no promises."
- Client/server is a role, not a machine: driver↔program and CPU↔GPU are client/server on one box.
- Choose client/server for one-service-many-consumers; choose P2P when capacity should scale with participation.

## Anti-patterns / Common Errors
- **Assuming the network is reliable**: IP loses, reorders, and duplicates packets by design; build on TCP or handle it yourself.
- **Scaling by adding clients**: clients add demand without resources; the server stays scarce and remains a single point of failure.
- **Calling a scaffolded network "pure P2P"**: specialized components with different functions (supernodes) make it hybrid.
- **Baking protocol into one implementation**: a protocol tied to a particular program or library defeats its purpose — different software systems must be able to conform.

## Key Takeaways
1. No shared memory across machines: everything is messages plus protocols.
2. Protocols are language-agnostic rules; conformance enables interoperation.
3. IP: best-effort packets (loss, reorder, duplication, 65,535-byte cap). TCP: reliable ordered streams over IP, at a latency cost, via the three-step handshake and sequence headers.
4. Client/server gives powerful modularity (mutual ignorance of details) but a single point of failure and non-scaling capacity.
5. Peer-to-peer scales capacity with membership and tolerates failure via replication, at the cost of maintaining network structure.
6. Fixed response codes are a general protocol-design technique.

## Connects To
- **Section 4.7**: MapReduce coordinates distributed machines for data processing atop these mechanisms.
- **Section 4.8**: message passing reappears as a synchronization approach among processes on one machine.
- **Chapter 2 (OOP message passing)**: the message metaphor scales from objects to machines.
