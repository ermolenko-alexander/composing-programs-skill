# Section 4.6: Distributed Computing

## Core Idea
A distributed computing application is multiple interconnected but independent computers coordinating a joint computation; they share no memory, so all coordination happens through messages governed by agreed protocols.

## Frameworks Introduced
- **Message protocols**: a set of rules for encoding and interpreting messages (byte sequences). Protocols are not programs or libraries — any program in any language that conforms can participate in the system.
- **TCP/IP protocols**: IP transfers packets (max 65,535 bytes) best-effort across unreliable, dynamic networks with no central tracking — packets may arrive out of order, be lost, or be duplicated. TCP is an abstraction over IP providing reliable, ordered transmission of arbitrarily large byte streams (ordering, dedup, retransmission), paying for reliability with latency. Connections start with a three-step handshake (request to a port + reply port, response + wait for ack, acknowledgment).
- **Client/server architecture**: a server dispenses a service from a central source; many interchangeable clients issue requests and consume responses. Clients need not know how the service is computed; the server need not know how responses are used.
  - When to use: service-oriented situations (the Web, OS device drivers serving input to programs, CPU as client of the GPU).
- **HTTP**: a protocol implemented over TCP governing the Web; `GET` requests a location on port 80; responses start with fixed response codes (200 OK, 404 Not Found) — fixed code sets reduce transmission size and standardize semantics.
- **Peer-to-peer systems**: labor divided among all components; every computer sends/receives data and contributes processing and memory, so capacity grows with system size. Many are not pure P2P: scaffolding components (e.g. Skype supernodes) maintain network structure, locations, and membership.
  - When to use: data transfer and data storage — data too large or too valuable for one machine, replicated across peers and restored on failure.

## Key Concepts
- **Message**: information transferred between computers over a network; computers do not share memory.
- **Packet**: IP's unit of best-effort data transfer, carrying a header with the destination IP address.
- **TCP segment**: a portion of a data stream plus header with sequence/state info; some segments only establish or terminate connections.
- **Port / port number**: the addressable endpoint on a machine to which requests and responses are sent.
- **DNS**: a server mapping domain names to IP addresses (`socket.gethostbyname('www.nytimes.com')`).
- **Response code**: fixed protocol constants (200 success, 404 not found).
- **Single point of failure**: the server is the only component able to dispense the service.
- **Supernode**: a scaffolding component in a non-pure P2P network handling login/logout, locations, and structure changes.

## Mental Models
- Think of a protocol as shared semantics, not shared code — heterogeneous systems interoperate purely by conforming.
- Think of TCP as the reliability layer purchased with latency on top of IP's "best effort, no promises."
- Use client/server when there is one service and many consumers; use peer-to-peer when you want capacity to scale with participants.
- Client/server is a role, not a machine: driver↔program and CPU↔GPU are client/server on one box.

## Anti-patterns
- **Assuming the network is reliable**: IP loses, reorders, and duplicates packets by design; build on TCP or handle it yourself.
- **Scaling by adding clients**: clients increase demand without contributing resources; the server becomes scarce and remains a single point of failure.
- **Calling a scaffolded network "pure P2P"**: specialized components with different functions (supernodes) make it a hybrid.

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
- **What it demonstrates**: the two client requests behind viewing a web page — DNS lookup, then HTTP GET to the web server.

## Worked Example
Requesting a web page. The browser (client) first asks a Domain Name Server to map www.nytimes.com to an IP address; then it sends an HTTP GET over TCP to that address and receives a response headed `HTTP/1.1 200 OK` with date, server, and content-type lines, followed by an HTML document; the HTML contains addresses of images and other components, which trigger further requests. A mistyped address yields `HTTP/1.1 404 Not Found`. Underneath, TCP's three-way handshake established the connection, and TCP ordered and retransmitted the IP packets carrying the page.

## Key Takeaways
1. No shared memory across machines: everything is messages plus protocols.
2. Protocols are language-agnostic rules; conformance, not shared software, enables interoperation.
3. TCP = reliable ordered streams over unreliable IP, at a latency cost.
4. Client/server gives powerful modularity but a single point of failure and non-scaling capacity.
5. Peer-to-peer scales capacity with membership and tolerates failure via replication, at the cost of maintaining network structure.
6. Fixed response codes are a general protocol-design technique.

## Connects To
- **Section 4.7**: MapReduce coordinates distributed machines for data processing atop these mechanisms.
- **Section 4.8**: message passing reappears as a synchronization approach within one machine.
