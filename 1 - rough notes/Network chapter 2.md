# Comprehensive Study Guide: Computer Networking — Chapter 2 (Application Layer)

This study guide provides an exhaustive synthesis of Chapter 2 ("Application Layer") from *Computer Networking: A Top-Down Approach* by James F. Kurose and Keith W. Ross. It covers the core conceptual, mathematical, and algorithmic foundations of network applications, protocol standards, directory services, content distribution architectures, and socket programming.

## 1. Architectural Principles of Network Applications

### 1.1 Application Architecture vs. Network Architecture

In network engineering, a fundamental distinction exists between the fixed network architecture and the developer-designed application architecture:

* **Network Architecture:** Refers to the underlying 5-layer Internet protocol stack (Application, Transport, Network, Data Link, Physical). It is fixed from the perspective of an application developer and provides a set of core communication services.
* **Application Architecture:** Designed by the application developer, dictating how the application software is structured across various end systems (hosts).

Modern network applications primarily utilize two predominant architectural paradigms:

1. **Client-Server Architecture:**
    * Features an always-on host called the server that services requests from many client hosts.
    * Clients do not communicate directly with each other (e.g., two Web browsers do not exchange data directly).
    * The server possesses a fixed, well-known IP address, allowing clients to initiate contact at any time.
    * To handle massive scaling, servers are frequently housed in data centers containing thousands to millions of physical servers.
2. **Peer-to-Peer (P2P) Architecture:**
    * Minimizes or eliminates reliance on dedicated infrastructure servers in data centers.
    * Exploits direct communication between pairs of intermittently connected hosts called peers.
    * Peers are user-controlled end systems (desktops, laptops, smartphones) residing in home, enterprise, or campus networks.
    * Demonstrates self-scalability: as new peers join the network to download content, they simultaneously contribute their upload bandwidth capacity to the system.

### 1.2 Process Communication and Sockets

In operating systems terminology, communication occurs between processes running within end systems. When processes run on different hosts, they communicate by exchanging messages across the network.

* **Client and Server Processes:** In any communication session between two processes:
  * The process that initiates the communication (contacts the other process first) is designated as the **client**.
  * The process that waits to be contacted to begin the session is designated as the **server**.
  * In P2P applications, a host can act as both a client and a server within the same application, but for any given message exchange pair, one process acts as the client and the other as the server.
* **The Socket Interface (API):**
  * A socket is a software interface on an end system through which an application process sends and receives messages from the network. It functions as the Application Programming Interface (API) between the application layer and the transport layer.
  * **House and Door Analogy:** A process is analogous to a house, and its socket is analogous to its door. When a process wants to send a message, it shoves it out the door (socket), relying on the underlying transport infrastructure to deliver the message to the destination socket.
  * The developer has complete control over everything on the application-layer side of the socket, but has minimal control over the transport-layer side (restricted to selecting the transport protocol and tuning a few parameters like buffer sizes).
* **Addressing Processes:** To direct messages to a specific process on a remote host, two pieces of information are required:
  1. **IP Address:** A 32-bit quantity (in IPv4) that uniquely identifies the target host machine.
  2. **Port Number:** An integer that identifies the specific receiving socket/process on the destination host (e.g., HTTP Web servers use port 80; SMTP mail servers use port 25).

### 1.3 Transport Services Required by Applications

Application developers select an underlying transport protocol based on four fundamental service dimensions:

| Service Dimension | Requirements & Characteristics | Application Examples |
| --- | --- | --- |
| Data Loss / Reliability | Reliable Data Transfer: Guarantees 100% accurate and complete byte delivery. Loss-tolerant applications can accept minor data degradation. | File transfer, E-mail, Web documents require no loss. Audio/video streaming are loss-tolerant. |
| Throughput | Bandwidth-sensitive applications require a guaranteed minimum throughput. Elastic applications consume as much or as little throughput as available. | Streaming video/audio require specific kbps/Mbps. E-mail, Web, and file transfer are elastic. |
| Timing / Latency | Low-delay guarantees (e.g., max hundreds of milliseconds) required for real-time interactivity. | Internet telephony (VoIP), interactive multiplayer gaming, teleconferencing. |
| Security | Confidentiality (encryption), data integrity, and end-point authentication. | Financial transactions, private e-mail, secure login. |

### 1.4 Transport Services Provided by the Internet

The Internet transport layer provides two main protocols to application developers:

1. **Transmission Control Protocol (TCP):**
    * **Connection-Oriented:** Requires a 3-way handshaking procedure between client and server to initialize state variables and buffers before application data flows.
    * **Reliable Data Transfer:** Guarantees that all data delivered through the socket arrives without error, loss, duplication, or misordering.
    * **Congestion Control:** Throttles sending rates when the network core experiences congestion, balancing traffic for the general welfare of the Internet.
    * **Flow Control:** Prevents a fast sender from overflowing a slow receiver's buffer.
2. **User Datagram Protocol (UDP):**
    * **Connectionless:** No preliminary handshaking; sends packets immediately.
    * **Unreliable Data Transfer:** Does not guarantee packet arrival, delivery order, or duplicate prevention.
    * **No Congestion Control:** Pumps data into the network layer at any rate the application dictates.
    * **Minimal Overhead:** Contains no connection setup latency and features an lightweight 8-byte header (compared to TCP's 20-byte header).

> [!warning] Security Note
> Neither TCP nor UDP provides built-in encryption. Security is added at the application layer by enhancing TCP with Transport Layer Security (TLS), which encrypts data passed into the socket before handing it down to TCP.

## 2. The Web and Hypertext Transfer Protocol (HTTP)

### 2.1 Overview of HTTP

The HyperText Transfer Protocol (HTTP) is the Web's application-layer protocol, defined in RFC 1945 (HTTP/1.0), RFC 7230 (HTTP/1.1), and RFC 7540 (HTTP/2).

* **Client-Server Model:** Web browsers implement client-side HTTP; Web servers (e.g., Apache, Nginx, Microsoft IIS) implement server-side HTTP.
* **Underlying Transport:** HTTP uses TCP as its transport protocol (default port 80). The client initiates a TCP connection to the server, and messages are transferred reliably across socket interfaces.
* **Stateless Protocol:** An HTTP server maintains no state information about past client requests. If a client requests the exact same object twice in a few seconds, the server resends it completely without remembering the previous transaction.

### 2.2 Non-Persistent vs. Persistent Connections

#### Non-Persistent Connections (HTTP/1.0)

* Each TCP connection is closed immediately after the server transmits a single requested object.
* Fetching an HTML page with 10 embedded JPEG images requires 11 separate TCP connections.
* **RTT Calculation:** The Round-Trip Time (RTT) is the time taken for a small packet to travel from client to server and back.

  $$\text{Total Response Time per Object} = 2 \times \text{RTT} + \text{Transmission Time of File}$$

  * First RTT: Initiates 3-way TCP handshake (SYN, SYN-ACK).
  * Second RTT: Sends HTTP request combined with ACK, and receives initial response data from the server.

#### Persistent Connections (HTTP/1.1 Default)

* The server keeps the TCP connection open after sending a response. Subsequent requests and responses between the same client and server flow over the single established connection.
* **Pipelining:** Clients can issue back-to-back requests for embedded objects without waiting for replies to pending requests, reducing delivery delay for dependent objects to approximately $1 \times \text{RTT}$ total after connection setup.

### 2.3 HTTP Message Formats

HTTP messages are written in plain ASCII text.

* **Request Structure:** Contains a **Request Line** (method such as GET, POST, HEAD, PUT, DELETE; URL path; HTTP version) followed by **Header Lines** (Host, Connection, User-Agent, Accept-Language) and an optional **Entity Body**.
* **Response Structure:** Contains a **Status Line** (HTTP version, status code, reason phrase) followed by **Header Lines** (Date, Server, Last-Modified, Content-Length, Content-Type) and the **Entity Body** containing the requested file data.
* **Status Codes:**
  * `200 OK`: Request succeeded.
  * `301 Moved Permanently`: Requested object permanently relocated.
  * `304 Not Modified`: Cached copy is fresh (conditional GET).
  * `400 Bad Request`: Generic syntax error in request message.
  * `404 Not Found`: Requested document does not exist on server.
  * `505 HTTP Version Not Supported`: Server lacks support for requested protocol version.

### 2.4 User-Server Interaction: Cookies

Because HTTP is stateless, Web sites use **cookies** (RFC 6265) to maintain state and track user sessions across transactions. The server responds to an initial request with a `Set-Cookie` header containing a unique user identification number. The browser stores this identifier in its internal cookie database and automatically includes a `Cookie` header on all subsequent requests to that domain.

### 2.5 Web Caching and Conditional GET

A **Web Cache (Proxy Server)** is a network entity with local storage that satisfies HTTP requests on behalf of origin Web servers.

* **Benefits:** Reduces response time for client requests and cuts bandwidth consumption on an institution's access link.
* **Traffic Intensity Analysis:** If access link capacity is $15\text{ Mbps}$, request rate is $15\text{ req/sec}$, and object size is $1\text{ Mb}$, traffic intensity is 1.0. Installing a cache with a $40\%$ hit rate reduces link traffic intensity to 0.6, eliminating severe queuing delays.
* **Conditional GET:** Proxy caches send an `If-Modified-Since` header containing the cached object's timestamp to the origin server. If unchanged, the origin server returns status `304 Not Modified` without an entity body.

### 2.6 Evolution of HTTP: HTTP/2 and HTTP/3

* **HTTP/1.1 Head-of-Line (HOL) Blocking:** A large object at the head of a persistent TCP connection blocks all subsequent small objects behind it.
* **HTTP/2:** Introduces binary framing and stream interleaving over a single TCP connection, multiplexing multiple requests and responses concurrently without HOL blocking.
* **HTTP/3:** Implements application-layer transport protocol QUIC over UDP, eliminating transport-level TCP HOL blocking across independent streams and supporting zero-RTT connection establishment.

## 3. Electronic Mail Infrastructure

Internet e-mail consists of three core structural components: **User Agents**, **Mail Servers**, and the **Simple Mail Transfer Protocol (SMTP)**.

### 3.1 Simple Mail Transfer Protocol (SMTP)

SMTP (RFC 5321) transfers mail messages from the sender's mail server to the recipient's mail server over TCP (port 25).

* **Direct Transfer:** SMTP does not use intermediate relay servers. The sender's mail server opens a direct TCP connection to the recipient's mail server. If the destination server is down, the message remains queued at the sender's server for retry.
* **7-Bit ASCII Constraint:** Requires all mail message bodies to be encoded in 7-bit ASCII text.
* **Handshaking Sequence:** Setup connection (TCP port 25), `HELO` greeting, `MAIL FROM:`, `RCPT TO:`, `DATA` command (ended by a line with a single period `.`), and `QUIT`.

### 3.2 Mail Message Formats vs. SMTP Headers

* **Mail Header Fields:** Defined in RFC 5322 (includes From:, To:, Subject:, Date:). These headers are contained within the text body of the mail message itself during the DATA phase.
* **SMTP Protocol Commands:** `MAIL FROM:` and `RCPT TO:` are envelope protocol commands exchanged during the handshaking phase and are separate from the message body headers.

### 3.3 Mail Access Protocols

Because a recipient's user agent cannot remain permanently online to receive SMTP connections, messages are deposited in the recipient's always-on mail server. The user agent retrieves mail using a **Mail Access Protocol**:

1. **POP3 (Post Office Protocol 3 - RFC 1939):** Operates over TCP port 110 across Authorization, Transaction (LIST, RETR, DELE), and Update (QUIT) phases.
2. **IMAP (Internet Message Access Protocol - RFC 3501):** Maintains state across client sessions by storing every message in hierarchical folders on the central server, allowing partial message inspection.
3. **HTTP Webmail:** Browsers act as user agents interacting with the mail server via HTTP, while the mail server uses SMTP internally to relay messages to remote servers.

## 4. Domain Name System (DNS): The Internet's Directory Service

### 4.1 Services Provided by DNS

The Domain Name System (DNS) is an application-layer directory service running over UDP port 53 that translates hostnames to 32-bit IP addresses. It also provides **Host Aliasing**, **Mail Server Aliasing**, and **Server Load Distribution**.

### 4.2 Hierarchical Architecture of DNS Servers

DNS uses a distributed, hierarchical database to prevent single-point failures:

1. **Root DNS Servers:** 13 root server IP addresses (replicated globally via anycast) provide pointers to Top-Level Domain (TLD) servers.
2. **Top-Level Domain (TLD) Servers:** Responsible for generic domains (.com, .org, .net) and country-code domains (.uk, .fr).
3. **Authoritative DNS Servers:** Maintained by organizations or service providers, housing public DNS records mapping hostnames to IP addresses.
4. **Local DNS Server:** Provided by ISPs or institutions to act as a proxy cache forwarding queries on behalf of local hosts.

### 4.3 Query Resolution: Recursive vs. Iterative

* **Iterative Querying (Standard):** Local DNS server sequentially queries Root DNS $\rightarrow$ TLD DNS $\rightarrow$ Authoritative DNS, receiving referrals at each step until obtaining the final mapping.
* **Recursive Querying:** Each contacted DNS server forwards the query up the hierarchy to the next server and returns the final answer back down the chain.

### 4.4 Resource Records (RRs)

DNS servers store Resource Records (RRs) as 4-tuples: `(Name, Value, Type, TTL)`.

* **Type A:** Name is a hostname, Value is an IPv4 address.
* **Type NS:** Name is a domain, Value is the hostname of an Authoritative DNS server.
* **Type CNAME:** Name is an alias hostname, Value is the canonical hostname.
* **Type MX:** Name is a domain, Value is the canonical name of the mail server.

## 5. Peer-to-Peer (P2P) Systems and File Distribution

### 5.1 Mathematical Model of File Distribution Scalability

Distributing a file of size $F$ bits to $N$ peers ($u_s$ = server upload rate, $u_i$ = peer $i$ upload rate, $d_{\min}$ = minimum peer download rate):

#### Client-Server Distribution Time ($D_{CS}$)

$$D_{CS} \ge \max \left\{ \frac{N \cdot F}{u_s}, \frac{F}{d_{\min}} \right\}$$

As $N \rightarrow \infty$, distribution time grows linearly without bound.

#### P2P Distribution Time ($D_{P2P}$)

$$D_{P2P} \ge \max \left\{ \frac{F}{u_s}, \frac{F}{d_{\min}}, \frac{N \cdot F}{u_s + \sum_{i=1}^N u_i} \right\}$$

As $N \rightarrow \infty$, P2P distribution time approaches a constant baseline $\frac{F}{u}$, demonstrating self-scalability.

### 5.2 BitTorrent Protocol Mechanics

BitTorrent partitions files into equal $256\text{ KB}$ chunks.

* **Torrent & Tracker:** A swarm of peers distributing a file is a **torrent**; an infrastructure node called a **tracker** maintains the peer list.
* **Rarest-First Selection:** Peers request rarest chunks first to equalize chunk distribution across the swarm.
* **Tit-for-Tat Unchoking Algorithm:** Peers measure arrival rates from neighbors every 10 seconds, unchoking the top 4 uploaders. Every 30 seconds, one additional peer is optimistically unchoked to discover better uploaders.

## 6. Video Streaming and Content Distribution Networks (CDNs)

### 6.1 Dynamic Adaptive Streaming over HTTP (DASH)

Video is encoded into multiple bit rates, divided into 2–10 second chunks. A **Manifest File** lists available URLs and bit rates. The client dynamically evaluates bandwidth and buffer levels, requesting appropriate bit-rate chunks.

### 6.2 Content Distribution Networks (CDNs)

CDNs deploy geographically distributed server clusters using two strategies:

1. **Enter Deep:** Small server clusters inside hundreds of access ISP networks worldwide.
2. **Bring Home:** Large server clusters in major IXP locations.

#### CDN DNS Redirection Workflow

1. Client requests video URL $\rightarrow$ Local DNS (LDNS) queries Content Provider DNS.
2. Content Provider DNS returns CNAME pointing to CDN domain.
3. LDNS queries CDN Authoritative DNS.
4. CDN DNS selects optimal edge node based on LDNS IP address and returns its IP.
5. Client establishes TCP connection to CDN edge node to stream video.

## 7. Socket Programming API: Building Network Applications

### 7.1 UDP Socket Programming (Python 3)

#### UDP Client (UDPClient.py)

```python
from socket import *

serverName = 'hostname'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_DGRAM)
message = input('Input lowercase sentence: ')
clientSocket.sendto(message.encode(), (serverName, serverPort))

modifiedMessage, serverAddress = clientSocket.recvfrom(2048)
print('From Server:', modifiedMessage.decode())
clientSocket.close()
```

#### UDP Server (UDPServer.py)

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_DGRAM)
serverSocket.bind(('', serverPort))

print('The UDP server is ready to receive')
while True:
    message, clientAddress = serverSocket.recvfrom(2048)
    capitalizedSentence = message.decode().upper()
    serverSocket.sendto(capitalizedSentence.encode(), clientAddress)
```

### 7.2 TCP Socket Programming (Python 3)

#### TCP Client (TCPClient.py)

```python
from socket import *

serverName = 'hostname'
serverPort = 12000

clientSocket = socket(AF_INET, SOCK_STREAM)
clientSocket.connect((serverName, serverPort))

sentence = input('Input lowercase sentence: ')
clientSocket.send(sentence.encode())

modifiedSentence = clientSocket.recv(1024)
print('From Server:', modifiedSentence.decode())
clientSocket.close()
```

#### TCP Server (TCPServer.py)

```python
from socket import *

serverPort = 12000
serverSocket = socket(AF_INET, SOCK_STREAM)
serverSocket.bind(('', serverPort))
serverSocket.listen(1)

print('The TCP server is ready to receive')
while True:
    connectionSocket, addr = serverSocket.accept()
    sentence = connectionSocket.recv(1024).decode()
    capitalizedSentence = sentence.upper()
    connectionSocket.send(capitalizedSentence.encode())
    connectionSocket.close()
```

## 8. Practice Examination & Review Problems

### Short-Answer Practice Quiz

1. **Question 1:** Explain why HTTP, SMTP, and DNS protocols run on top of TCP or UDP as indicated in standard Internet architecture. Why does DNS use UDP while HTTP uses TCP?
2. **Question 2:** Calculate the total response time to fetch a base HTML document and 4 embedded images over non-persistent HTTP without parallel connections. Assume round-trip time is $\text{RTT} = 50\text{ ms}$, transmission time per object is $\text{RTT}_{\text{trans}} = 10\text{ ms}$, and initial IP resolution is cached.
3. **Question 3:** What is the difference between an iterative DNS query and a recursive DNS query?
4. **Question 4:** Describe how conditional GET messages prevent bandwidth waste in Web proxy caches.
5. **Question 5:** What is Head-of-Line (HOL) blocking in HTTP/1.1, and how does HTTP/2 solve it?
6. **Question 6:** Explain the difference between `MAIL FROM:` in SMTP and the `From:` header line in an e-mail message body.
7. **Question 7:** In P2P file distribution, why does BitTorrent use a rarest-first chunk selection strategy?
8. **Question 8:** What are the four tuple values that uniquely identify a TCP socket on a server host?
9. **Question 9:** In Python socket programming, why does a TCP server require two distinct sockets (`serverSocket` and `connectionSocket`), whereas a UDP server requires only one (`serverSocket`)?
10. **Question 10:** Explain how DNS CNAME records and DNS redirection enable Content Distribution Networks (CDNs) to load-balance Web video streams.

### Answer Key

1. **Answer 1:** HTTP requires reliable, in-order byte delivery because corrupted or missing HTML/text data renders documents unusable; hence it runs over TCP. DNS relies on small, single-request single-response exchanges; using UDP avoids the overhead and latency of TCP's 3-way connection handshake.
2. **Answer 2:** Fetching the base HTML file requires $2 \times \text{RTT} + \text{RTT}_{\text{trans}} = 2(50) + 10 = 110\text{ ms}$. Each of the 4 embedded images requires an independent non-persistent TCP connection taking $2 \times \text{RTT} + \text{RTT}_{\text{trans}} = 110\text{ ms}$. Total time = $110 + 4(110) = 550\text{ ms}$.
3. **Answer 3:** In an iterative query, the contacted DNS server replies directly to the local DNS server with a referral to the next-hop DNS server in the hierarchy. In a recursive query, the contacted server assumes responsibility for obtaining the final IP mapping by querying subsequent servers itself down the chain.
4. **Answer 4:** A proxy cache includes an `If-Modified-Since:` header with the last modification timestamp of its cached copy. If the origin server's document has not changed, it returns an HTTP `304 Not Modified` status code with no entity body, saving link bandwidth.
5. **Answer 5:** In HTTP/1.1 over a single persistent connection, large objects at the head of the queue delay all subsequent small objects behind them. HTTP/2 solves this by breaking messages into binary frames and interleaving them concurrently over a single TCP stream.
6. **Answer 6:** `MAIL FROM:` is an SMTP envelope protocol command transmitted during handshaking to identify the sender to the mail server. `From:` is a header line written inside the text payload of the mail message itself, rendered by the user agent.
7. **Answer 7:** Requesting rarest chunks first ensures that chunks held by the fewest peers are copied quickly into the swarm, reducing the risk of a chunk disappearing if peers holding it disconnect.
8. **Answer 8:** Source IP Address, Source Port Number, Destination IP Address, Destination Port Number.
9. **Answer 9:** TCP is connection-oriented; `serverSocket` acts as an always-open welcoming door to accept initial connection requests, spawning a dedicated `connectionSocket` for each distinct client connection. UDP is connectionless; packets carry explicit destination address tags, allowing a single `serverSocket` to handle incoming datagrams from all clients.
10. **Answer 10:** When a client queries a content provider's domain, authoritative DNS returns a CNAME pointing to the CDN's DNS domain. The CDN's authoritative DNS server evaluates the requester's LDNS IP address and dynamically returns the IP address of the geographically optimal, least-loaded CDN edge node.

## 9. Synthesis & Discussion Questions

1. **Architectural Evaluation of Protocol Layering:**
    * **Prompt:** Evaluate the architectural trade-offs of implementing QUIC and HTTP/3 at the application layer over UDP versus making structural modifications to the kernel-level TCP protocol at the transport layer. Why are modern Web protocol innovations increasingly occurring at the application layer?
2. **Infrastructure Scalability Analysis:**
    * **Prompt:** Compare client-server data center deployments with P2P file-sharing networks across bandwidth cost, security vulnerabilities, copyright management, and fault-tolerance metrics. Under what network conditions does P2P fail to demonstrate self-scalability?
3. **Privacy and State Management in Stateless Protocols:**
    * **Prompt:** Examine how HTTP cookies, Web beacons, and DNS query logging impact end-user privacy. How do modern browser security standards balance persistent session state management with anti-tracking requirements?
4. **CDN Edge Server Distribution Strategies:**
    * **Prompt:** Contrast Akamai's "Enter Deep" CDN placement strategy with Google/Netflix's "Bring Home" IXP strategy. What financial, topological, and operational constraints dictate an enterprise's choice between private and third-party CDNs?
5. **Socket API Abstraction & Real-Time Performance:**
    * **Prompt:** Analyze how operating system buffer allocations, TCP slow-start algorithms, and Nagle's algorithm interact with application-layer socket calls (`send` and `recv`). How do these lower-layer mechanisms impact real-time multiplayer gaming protocols?

## 10. Glossary of Core Concepts

* **Application Architecture:** The structural design dictating how an application's processes are distributed across end systems (e.g., Client-Server, P2P).
* **Socket API:** The software interface between an application process and the transport-layer protocol within a host machine.
* **Stateless Protocol:** A protocol whose server retains no state information about past client requests (e.g., HTTP).
* **Round-Trip Time (RTT):** The time required for a small packet to travel from a client to a remote server and back.
* **Persistent Connection:** An HTTP design where a single underlying TCP connection remains open for multiple sequential request/response exchanges.
* **Head-of-Line (HOL) Blocking:** A queuing delay phenomenon where a large packet/request at the front of a line prevents subsequent small packets/requests from proceeding.
* **Web Cache (Proxy Server):** An intermediary network entity that stores copies of recently requested Web objects to satisfy client requests locally.
* **Conditional GET:** An HTTP request containing an `If-Modified-Since` header, used by caches to verify object freshness without re-downloading unmodified data.
* **SMTP (Simple Mail Transfer Protocol):** The principal application-layer protocol for transferring e-mail messages directly between mail servers over TCP port 25.
* **IMAP (Internet Message Access Protocol):** A mail access protocol that permits user agents to manipulate and synchronize hierarchical mail folders maintained on a central mail server.
* **DNS (Domain Name System):** A distributed, hierarchical application-layer directory service that translates human-mnemonic hostnames into IP addresses over UDP port 53.
* **Resource Record (RR):** A 4-tuple database entry `(Name, Value, Type, TTL)` stored in DNS servers to resolve hostnames, name servers, and mail exchanges.
* **Self-Scalability:** The property of P2P networks where newly added client nodes simultaneously contribute upload bandwidth to the system, offsetting their own resource demands.
* **Torrent:** A community of P2P nodes actively distributing and downloading chunks of a specific file.
* **DASH (Dynamic Adaptive Streaming over HTTP):** A streaming technology where video content is chunked at multiple bit rates, allowing clients to dynamically request chunks based on real-time bandwidth.
* **CDN (Content Distribution Network):** A geographically distributed network of server clusters designed to deliver Web content and video streams to users from nearby edge nodes.