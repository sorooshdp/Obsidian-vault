# Computer Networking: Chapter 3 Study Notes (Transport Layer)

The transport layer occupies a central position within the Internet protocol stack, residing directly between the application layer and the network layer. Its primary role is to provide logical communication services directly to application processes running on distinct end systems. While the network layer provides host-to-host delivery across a complex mesh of routers and links, the transport layer transforms this broad host-to-host channel into precise, process-to-process delivery channels equipped with varying degrees of reliability, security, flow control, and congestion control.

## 1. Transport-Layer Services and Multiplexing/Demultiplexing

### 1.1 Relationship Between Transport and Network Layers

A fundamental distinction in network architecture is the division of labor between the transport layer and the network layer:

* **Network-Layer Service:** Provides logical communication between hosts. The Internet Protocol (IP) operates at this layer, moving IP datagrams across intermediate routers from a source host IP address to a destination host IP address. IP provides a best-effort, unreliable service model—it does not guarantee packet delivery, in-order arrival, or bit integrity.
* **Transport-Layer Service:** Provides logical communication between processes. Transport protocols execute entirely within the end systems (hosts), encapsulating application messages into transport-layer segments without participating in core router operations.

To conceptualize this relationship, consider the classic household mailing analogy:

* Two extended families live in separate houses located in distant cities (Host A and Host B). Each house contains 12 children (application processes).
* Ann in House A and Bill in House B act as household mail delegates. Their job is to collect letters written by their siblings, walk them to the corner mailbox, receive incoming mail from the postal carrier, and distribute the incoming letters directly to the specific child addressed.
* The national postal service represents the network layer (host-to-host delivery).
* Ann and Bill represent the transport layer protocol (process-to-process delivery within the end systems).
* The individual children represent the application-layer processes.

Just as Ann and Bill cannot speed up the postal service's delivery trucks or prevent postal trucks from losing mail, a transport protocol's capabilities are constrained by the underlying network-layer service model. However, a transport protocol can provide critical services—such as reliable data transfer—even when the underlying network layer is inherently unreliable.

### 1.2 Multiplexing and Demultiplexing Mechanics

Extending host-to-host delivery to process-to-process delivery requires two reciprocal operations:

1. **Multiplexing (at Sender Host):** Gathering data chunks from multiple application sockets, encapsulating each chunk with header fields (including source and destination port numbers), and passing the resulting segments down to the network layer.
2. **Demultiplexing (at Receiver Host):** Inspecting incoming transport-layer segment headers to identify the designated destination socket and delivering the payload to that specific socket.

```text
+-------------------------------------------------------------------+
|                        Application Layer                          |
|   Process P1 (Port 19157)                 Process P2 (Port 46428) |
|         |                                       ^                 |
|         v (Socket)                              | (Socket)        |
+---------|---------------------------------------|-----------------+
|         | Multiplexing                          | Demultiplexing  |
|         v                                       |                 |
|   Transport Layer (UDP / TCP Header added)      |                 |
|   [ Src: 19157 | Dst: 46428 | Payload... ] ---->|                 |
+-------------------------------------------------------------------+
|                        Network Layer (IP)                         |
+-------------------------------------------------------------------+
```

#### Port Numbers and Socket Identifiers

Sockets act as the software doors between application processes and the transport layer. Each socket in an end system is assigned a 16-bit port number, ranging from 0 to 65535:

* **Well-Known Ports (0 to 1023):** Restricted and reserved for standard application protocols by IANA (RFC 3232). Examples include HTTP (Port 80), HTTPS (Port 443), FTP (Port 21), SSH (Port 22), and DNS (Port 53).
* **Ephemeral / Dynamic Ports (1024 to 65535):** Automatically assigned by the client host's operating system when a client socket is created.

#### Connectionless Demultiplexing (UDP)

A UDP socket is identified uniquely by a 2-tuple:

$$\text{UDP Socket Identifier} = (\text{Destination IP Address}, \text{Destination Port Number})$$

When a UDP segment arrives at a host:

1. The transport layer checks the segment's Destination Port Number.
2. The transport layer routes the segment directly to the socket matching that port number.
3. If two UDP segments originate from different source IP addresses or different source port numbers, but carry the same destination IP address and destination port number, both segments will be demultiplexed to the same destination socket.

#### Connection-Oriented Demultiplexing (TCP)

A TCP socket is identified by a 4-tuple:

$$\text{TCP Socket Identifier} = (\text{Source IP Address}, \text{Source Port Number}, \text{Destination IP Address}, \text{Destination Port Number})$$

When a TCP segment arrives at a host:

1. The host examines all four header/IP fields.
2. Unless the segment is an initial connection request (SYN), the segment is directed to the specific connection socket matching the full 4-tuple.
3. Two arriving TCP segments with different source IP addresses or different source port numbers will be demultiplexed to two distinct sockets, even if they target the same destination port (e.g., Port 80 on a Web server).

In server applications (such as a Web server):

* A **Welcoming Socket** listens for incoming connection requests on a well-known port (e.g., Port 80).
* Upon accepting a connection via `accept()`, the operating system instantiates a new **Connection Socket** bound to that specific client's 4-tuple.
* High-performance servers spawn separate threads or processes for each connection socket to handle client streams independently.

## 2. Connectionless Transport: User Datagram Protocol (UDP)

Defined in RFC 768, UDP is a lightweight, "no-frills" transport protocol that adds minimal functionality beyond network-layer IP delivery. It operates without establishing a pre-connection handshake, earning its designation as connectionless.

### 2.1 Technical Motivations for UDP

Developers select UDP over TCP for specific performance and operational advantages:

1. **No Connection Establishment Delay:** UDP eliminates the 1-RTT delay required by TCP's three-way handshake. DNS relies on UDP primarily for this reason; running DNS over TCP would double domain lookup latencies.
2. **No Connection State:** UDP hosts do not maintain receive/send buffers, sequence numbers, congestion windows, or acknowledgment parameters. A server running over UDP can support significantly more active clients because it avoids connection-state memory overhead.
3. **Small Header Overhead:** UDP adds only 8 bytes of header overhead per segment, compared to TCP's baseline 20 bytes.
4. **Unregulated Sending Rate:** TCP's congestion control throttles the sender during network congestion. UDP pushes data into the network layer at whatever rate the application generates it, making it historically popular for real-time streaming and interactive voice applications that tolerate minor packet loss but require minimal latency.

| Feature / Metric | User Datagram Protocol (UDP) | Transmission Control Protocol (TCP) |
| --- | --- | --- |
| Connection Setup | None (Connectionless) | 3-Way Handshake Required |
| Header Size | 8 Bytes | 20–60 Bytes |
| Reliability | Unreliable (No ACKs or Retransmissions) | Guaranteed In-Order Reliable Delivery |
| Flow & Congestion Control | None (Unregulated) | Dynamic Windowing (rwnd & cwnd) |
| Demultiplexing Identifier | 2-Tuple (Dst IP, Dst Port) | 4-Tuple (Src IP, Src Port, Dst IP, Dst Port) |

### 2.2 UDP Segment Structure

The UDP header consists of four 16-bit fields (totaling 8 bytes):

```text
 0                   16                  31 Bits
+-------------------+-------------------+
| Source Port (16)  | Dst Port (16)     |
+-------------------+-------------------+
| Length (16)       | Checksum (16)     |
+-------------------+-------------------+
|                                       |
|      Application Data (Payload)       |
|                                       |
+---------------------------------------+
```

1. **Source Port Number (16 bits):** Port of the sending process (used by the receiver to reply).
2. **Destination Port Number (16 bits):** Port of the receiving process.
3. **Length (16 bits):** Total length of the UDP segment (header plus application data) in bytes. Minimum value is 8.
4. **Checksum (16 bits):** Error-detection field covering the pseudo-header, UDP header, and payload data.

### 2.3 UDP Checksum Calculation and the End-to-End Principle

The UDP checksum provides error detection against bit flips introduced during transmission across links or while residing in router memory.

#### Computation Algorithm (Sender)

1. Treat all 16-bit words in the segment (including a 12-byte IP pseudo-header containing source IP, destination IP, protocol number, and UDP length) as unsigned integers.
2. Sum all 16-bit words using 1s complement arithmetic (any overflow carry-out bit beyond 16 bits is added back to the least significant bit).
3. Take the 1s complement (flip all 0s to 1s and 1s to 0s) of the final sum.
4. Store the resulting 16-bit value in the Checksum field.

#### Verification Algorithm (Receiver)

1. Add all 16-bit words of the received segment, including the Checksum field itself.
2. If no bit errors occurred, the sum of the words and the checksum will equal $1111111111111111$ ($0\text{xFFFF}$). If any bit is 0, an error occurred.

#### Worked Numerical Example

Consider three 8-bit words (simplified for illustration): 01010011, 01100110, and 01110100.

```text
\begin{array}{r@{\quad}l} 01010011 & \

* 01100110 & \ \hline 10111001 & \text{(Sum of first two words)} \

* 01110100 & \ \hline 100101101 & \text{(Sum produces 9-bit result with carry-out 1)} \ \end{array}
```

Wrap around the carry bit:

```text
\begin{array}{r@{\quad}l} 00101101 & \

   1 & \\


\hline 00101110 & \text{(Sum after wraparound)} \ \end{array}
```

Take the 1s complement: $\text{Checksum} = \sim(00101110) = \mathbf{11010001}$

#### The End-to-End Principle

Why does UDP provide a checksum when link-layer protocols (like Ethernet) also implement error checking (CRC)?

* Intermediate links might use link protocols without error checking.
* Bit errors can occur in router memory or switches while stored in buffer memory between links.
* **The End-to-End Principle** (Saltzer, Reed, and Clark, 1984) states that functions that must be implemented on an end-to-end basis (such as integrity checking) cannot be entirely delegated to lower network layers, making lower-layer mechanisms redundant or incomplete.

## 3. Principles of Reliable Data Transfer (rdt)

Developing a reliable data transfer protocol requires constructing control mechanisms to overcome the imperfections of underlying physical channels, which can corrupt, drop, delay, or reorder packets.

```text
       Sending Side                                  Receiving Side
+-----------------------+                      +-----------------------+
|   Application Layer   |                      |   Application Layer   |
+-----------------------+                      +-----------------------+
     | rdt_send(data)                               ^ deliver_data(data)
     v                                              |
+-----------------------+    Unreliable        +-----------------------+
|  Transport Protocol   |====== Channel ======>|  Transport Protocol   |
+-----------------------+   udt_send(packet)   +-----------------------+
                                                     ^ rdt_rcv(packet)
```

### 3.1 Step-by-Step Protocol Evolution

#### rdt1.0: Reliable Transfer over a Perfectly Reliable Channel

* **Channel Assumptions:** No bit errors, no lost packets.
* **Mechanisms:** Sender creates packet and transmits (`udt_send()`). Receiver extracts data and delivers to upper layer (`deliver_data()`). Separate single-state FSMs for sender and receiver.

#### rdt2.0: Channel with Bit Errors (Stop-and-Wait)

* **Channel Assumptions:** Packets may experience bit corruption, but no packet loss.
* **Mechanisms Added:**
  1. **Error Detection:** Checksum field added to packets.
  2. **Receiver Feedback:** Positive Acknowledgments (ACK) and Negative Acknowledgments (NAK).
  3. **Retransmission:** Sender retransmits the packet upon receiving a NAK.
* **Protocol Classification:** Stop-and-Wait ARQ (Automatic Repeat reQuest) protocol.
* **The Fatal Flaw:** If an ACK or NAK packet itself becomes corrupted, the sender cannot determine whether the receiver successfully received the data. If the sender blindly retransmits, the receiver cannot distinguish between a new packet and a duplicate retransmission.

#### rdt2.1: Handling Corrupted Feedback with Sequence Numbers

* **Mechanisms Added:** 1-bit sequence numbers (0 and 1) attached to data packets.
* **Sender Behavior:** Includes current sequence number in data packet. Waits for an uncorrupted ACK/NAK. If feedback is corrupted or NAKed, retransmits current packet.
* **Receiver Behavior:** Checks sequence number of arriving packet. If a duplicate packet arrives (matching sequence number of previously delivered packet), the receiver discards the payload but re-sends an ACK. FSM expands to 4 states for both sender and receiver.

#### rdt2.2: A NAK-Free Protocol

* **Mechanisms Added:** Eliminates NAK packets entirely by requiring ACK packets to carry the sequence number of the packet being acknowledged (ACK 0 or ACK 1).
* **Receiver Behavior:** Upon receiving a corrupted packet or out-of-order packet, re-sends an ACK for the last correctly received packet.
* **Sender Behavior:** Receiving two consecutive ACKs for the same packet (duplicate ACK) signals that the subsequent packet was corrupted, triggering a retransmission.

#### rdt3.0: Reliable Transfer over a Lossy Channel (Alternating-Bit Protocol)

* **Channel Assumptions:** Underlying channel can corrupt bits AND completely drop packets.
* **Mechanisms Added:** Countdown Timer at the sender.
* **Loss Handling:** If an ACK is not received within a specified timeout interval, the sender assumes the packet (or its ACK) was lost and retransmits the packet.
* **Premature Timeout Handling:** If an ACK is merely delayed (not lost), the timer expires, causing a retransmission. Protocol rdt3.0 relies on sequence numbers to detect and silently discard the resulting duplicate packet at the receiver.

### 3.2 Pipelined Reliable Protocols

While rdt3.0 is functionally correct, its performance on high-speed links is severely degraded by its stop-and-wait operational model.

#### Stop-and-Wait Utilization Bottleneck

Consider a $1\text{ Gbps}$ link ($R = 10^9\text{ bps}$) across the United States with a round-trip time ($\text{RTT}$) of $30\text{ ms}$, sending $1{,}000\text{-byte}$ packets ($L = 8{,}000\text{ bits}$):

$$\text{Transmission Delay } d_{\text{trans}} = \frac{L}{R} = \frac{8{,}000\text{ bits}}{10^9\text{ bits/sec}} = 0.008\text{ ms} = 8\text{ }\mu\text{s}$$

Sender utilization ($U_{\text{sender}}$) is the fraction of time the sender is actively transmitting data bits:

$$U_{\text{sender}} = \frac{d_{\text{trans}}}{\text{RTT} + d_{\text{trans}}} = \frac{0.008\text{ ms}}{30\text{ ms} + 0.008\text{ ms}} = \frac{0.008}{30.008} \approx \mathbf{0.000267} \quad (0.0267\%)$$

Effective throughput is only $267\text{ kbps}$ on a $1\text{ Gbps}$ link!

#### Pipelining Solution

Pipelining allows the sender to transmit multiple packets without waiting for individual acknowledgments, keeping up to N unacknowledged packets in transit ("in the pipeline").

```text
Stop-and-Wait:  [Pkt 0]-------------------->(ACK 0) [Pkt 1]------------------->
Pipelined:      [Pkt 0][Pkt 1][Pkt 2]-----> (ACK 0)(ACK 1)(ACK 2)------------->
```

If N = 3, utilization triples. To support pipelining:

* The sequence number range must be expanded beyond 0 and 1.
* Sender and receiver entities must maintain multi-packet buffers.

### 3.3 Go-Back-N (GBN) vs. Selective Repeat (SR)

#### Go-Back-N (GBN) Protocol

GBN restricts the sender to a maximum sliding window size of N unacknowledged packets.

```text
                  sender's sequence number space:
[0 ... base-1]     [base ... nextseqnum-1]     [nextseqnum ... base+N-1]     [base+N ...]
Already ACKed      Sent, unACKed               Usable, not yet sent          Cannot be used
```

* **Cumulative Acknowledgments:** An ACK with sequence number n indicates that all packets up to and including n have been correctly received.
* **Single Timer:** The sender maintains a single timer corresponding strictly to the oldest unacknowledged packet (base). On timeout, the sender retransmits all N packets in the window (base through nextseqnum - 1).
* **Receiver Design:** The receiver maintains no out-of-order buffer. If packet n is expected but packet n+1 arrives, packet n+1 is discarded. The receiver re-ACKs the highest in-order packet (expectedseqnum - 1).

#### Selective Repeat (SR) Protocol

SR avoids unnecessary retransmissions by requiring the receiver to acknowledge packets individually.

* **Individual Acknowledgments:** Each packet in the pipeline is acknowledged independently.
* **Per-Packet Timers:** The sender maintains logical timers for every individual packet. On timeout for packet x, only packet x is retransmitted.
* **Receiver Buffering:** The receiver maintains an explicit receive window of size N, buffering out-of-order packets falling within `[rcv_base, rcv_base + N - 1]`. When contiguous packets starting at `rcv_base` are filled, they are delivered en masse to the application layer, and the receive window slides forward.

```text
Selective Repeat Receiver Window:
[rcv_base - N ... rcv_base - 1]   [rcv_base ... rcv_base + N - 1]   [rcv_base + N ...]
Already ACKed (re-ACK if rcvd)     Acceptable window range           Cannot receive yet
```

#### The SR Window Size Dilemma

In Selective Repeat protocols, the sequence number space size k must satisfy:

$$\text{Window Size } N \le \frac{\text{Sequence Number Space}}{2} = 2^{k-1}$$

If $N > 2^{k-1}$, the receiver cannot distinguish whether an arriving packet with sequence number x is a brand-new packet or a retransmission of an old packet from the previous sequence window cycle.

## 4. Connection-Oriented Transport: TCP

Transmission Control Protocol (TCP), defined across RFC 793, 1122, 2018, 5681, and 7323, is the Internet's dominant connection-oriented transport protocol.

### 4.1 Fundamental TCP Properties

* **Point-to-Point:** Connection exists strictly between two single end-system hosts. Multicasting or broadcasting is not supported.
* **Reliable, In-Order Byte Stream:** Data is treated as an unstructured, continuous stream of bytes without record markers.
* **Full-Duplex Service:** Data can flow simultaneously in both directions between Host A and Host B.
* **Connection-Oriented:** Requires a formal 3-way handshake to allocate buffers and state variables prior to data transmission.
* **Maximum Segment Size (MSS):** Limits the maximum amount of application payload data in a single TCP segment. MSS is typically set to $1{,}460\text{ bytes}$ (derived from Ethernet's $1{,}500\text{-byte}$ Maximum Transmission Unit [MTU] minus $20\text{ bytes}$ IP header and $20\text{ bytes}$ TCP header).

### 4.2 TCP Segment Structure

```text
 0                   15 16                   31 Bits
+----------------------+----------------------+
| Source Port Number   | Destination Port Num |
+----------------------+----------------------+
|            Sequence Number (32)             |
+----------------------+----------------------+
|         Acknowledgment Number (32)          |
+-------+--------+-----+----------------------+
|DataOff| Reserved|Flags| Receive Window (rwnd)|
+-------+--------+-----+----------------------+
|  Checksum (16)       | Urgent Pointer (16)  |
+----------------------+----------------------+
|            Options (if any)                 |
+---------------------------------------------+
|               Payload Data                  |
+---------------------------------------------+
```

* **Sequence Number (32 bits):** The byte-stream number of the first byte of data carried in the segment.
* **Acknowledgment Number (32 bits):** The sequence number of the next byte of data expected from the other side (cumulative acknowledgment).
* **Header Length / Data Offset (4 bits):** Specifies TCP header length in 32-bit words (typically 5 words = 20 bytes).
* **Flag Bits (6 bits + 2 ECN bits):**
  * **CWR** (Congestion Window Reduced) & **ECE** (ECN Echo): Explicit congestion signaling.
  * **URG:** Indicates urgent data pointer field is valid.
  * **ACK:** Indicates Acknowledgment number field is valid.
  * **PSH:** Requests immediate push of data to application.
  * **RST:** Resets and tears down an abnormal connection.
  * **SYN:** Initiates a connection (synchronize sequence numbers).
  * **FIN:** Closes a connection gracefully.
* **Receive Window / rwnd (16 bits):** Number of bytes the receiver is currently willing to accept (flow control).

### 4.3 RTT Estimation and Timeout Calculation

TCP sets its retransmission timeout (TimeoutInterval) dynamically based on moving estimates of Round-Trip Time ($\text{RTT}$).

#### Measuring SampleRTT

$\text{SampleRTT}$ is the measured time from segment transmission to ACK receipt. Per **Karn's Algorithm**, TCP never measures $\text{SampleRTT}$ for retransmitted segments, avoiding ambiguity as to whether an ACK corresponds to the original or retransmitted segment.

#### Exponential Weighted Moving Average (EWMA)

To smooth out network jitter, TCP updates $\text{EstimatedRTT}$ using an EWMA:

$$\text{EstimatedRTT} = (1 - \alpha) \cdot \text{EstimatedRTT} + \alpha \cdot \text{SampleRTT}$$

(Standard recommended value: $\alpha = 0.125$ or 1/8)

TCP computes RTT variability ($\text{DevRTT}$):

$$\text{DevRTT} = (1 - \beta) \cdot \text{DevRTT} + \beta \cdot |\text{SampleRTT} - \text{EstimatedRTT}|$$

(Standard recommended value: $\beta = 0.25$ or 1/4)

#### Setting TimeoutInterval

$$\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT}$$

When a timeout occurs, TCP doubles the TimeoutInterval for subsequent retransmissions (exponential backoff) to prevent compounding router queue overload.

### 4.4 TCP Reliable Data Transfer Mechanisms

TCP combines cumulative ACKs, single-timer management, and fast retransmission into a hybrid reliable data transfer model.

```text
Sender State                          Receiver Action
   |                                       |
   |---- Segment (Seq=100, Len=20) ------->|  (Receives Bytes 100-119)
   |                                       |  Sends ACK 120
   |<--- ACK 120 --------------------------|
   |                                       |
   |---- Segment (Seq=120, Lost X) --------|  (Packet Lost!)
   |---- Segment (Seq=140) --------------->|  (Out of order! Sends Dup ACK)
   |<--- ACK 120 (Dup ACK 1) --------------|
   |---- Segment (Seq=160) --------------->|  Sends Dup ACK
   |<--- ACK 120 (Dup ACK 2) --------------|
   |---- Segment (Seq=180) --------------->|  Sends Dup ACK
   |<--- ACK 120 (Dup ACK 3) --------------|
   |                                       |
   |===> FAST RETRANSMIT: Resends Seq 120 =>|
```

#### Fast Retransmit

Because timers can be long, TCP uses 3 duplicate ACKs (4 identical ACKs total) as an implicit signal that the segment immediately following the ACK number has been lost. Receiving 3 duplicate ACKs triggers **Fast Retransmit**—resending the missing segment immediately before its timer expires.

### 4.5 TCP Flow Control

Flow control is a speed-matching service that prevents a fast TCP sender from overflowing a slow receiver's application buffer.

```text
<---------------------------- RcvBuffer ---------------------------->
[   Delivered Data   |   Buffered Data   |    Spare Room (rwnd)    ]
                     ^                   ^                         ^
              LastByteRead         LastByteRcvd
```

The receiver advertises its available buffer space (rwnd) in every outgoing segment:

$$\text{rwnd} = \text{RcvBuffer} - (\text{LastByteRcvd} - \text{LastByteRead})$$

The sender enforces:

$$\text{LastByteSent} - \text{LastByteAcked} \le \text{rwnd}$$

#### Zero-Window Probing

If $\text{rwnd} = 0$, the sender is forced to stop transmitting. To prevent a deadlock where the receiver clears buffer space but has no data to transmit to notify the sender, the sender periodically sends 1-byte probe segments. The receiver responds with an ACK containing the updated non-zero $\text{rwnd}$.

### 4.6 Connection Lifecycle Management

#### Three-Way Handshake

1. **Client $\to$ Server (SYN):** Client sends a TCP segment with $\text{SYN} = 1$, $\text{ACK} = 0$, and a randomly generated Initial Sequence Number ($\text{client\_isn}$).
2. **Server $\to$ Client (SYNACK):** Server allocates connection buffers/variables, sends $\text{SYN} = 1$, $\text{ACK} = 1$, $\text{ack} = \text{client\_isn} + 1$, and its own random Initial Sequence Number ($\text{server\_isn}$).
3. **Client $\to$ Server (ACK):** Client allocates buffers, sends $\text{SYN} = 0$, $\text{ACK} = 1$, $\text{seq} = \text{client\_isn} + 1$, $\text{ack} = \text{server\_isn} + 1$. Payload data may be attached.

```text
Client (CLOSED)                                      Server (LISTEN)
   |                                                       |
   |--- SYN (seq=client_isn) ----------------------------->| (Enters SYN_RCVD)
   | (Enters SYN_SENT)                                     |
   |<-- SYNACK (seq=server_isn, ack=client_isn+1) ---------|
   | (Enters ESTABLISHED)                                  |
   |--- ACK (seq=client_isn+1, ack=server_isn+1) --------->| (Enters ESTABLISHED)
```

#### Connection Termination (Four-Wave Close)

1. Host A sends a segment with $\text{FIN} = 1$ and enters state FIN_WAIT_1.
2. Host B returns an ACK, transitioning Host A to FIN_WAIT_2. Host B enters CLOSE_WAIT.
3. Host B sends its own shutdown segment with $\text{FIN} = 1$ and enters LAST_ACK.
4. Host A returns an ACK and enters TIME_WAIT (holding resources for 2 Maximum Segment Lifetimes, $2\text{MSL} \approx 30\text{--}120\text{ seconds}$, to ensure the final ACK arrived), then closes. Host B receives the ACK and transitions to CLOSED.

#### Security: SYN Flood Attacks and SYN Cookies

In a **SYN Flood Attack**, attackers flood a target server with bogus SYN segments, causing the server to allocate half-open connection buffers until memory is exhausted, blocking legitimate users.

**SYN Cookies** defend against SYN floods without maintaining half-open connection state:

1. Upon receiving a SYN, the server does not allocate buffer space.
2. The server constructs an initial sequence number (cookie) that is a cryptographic hash of the source/destination IPs, source/destination ports, a secret key, and a timestamp.
3. The server sends this cookie in the SYNACK sequence number field.
4. When a legitimate client returns an ACK with $\text{ack} = \text{cookie} + 1$, the server recomputes the hash. If valid, the server creates the socket and allocates resources.

## 5. Principles and Mechanics of Congestion Control

Congestion occurs when too many sources attempt to send data through a network link faster than the link or intermediate routers can process it.

### 5.1 Causes and Costs of Network Congestion

#### Scenario 1: Infinite Buffers (Two Hosts, One Router)

* **Setup:** Two senders share a router with infinite buffer space and link capacity R.
* **Cost:** As sending rate $\lambda_{\text{in}}$ approaches R/2, queuing delay approaches infinity.

#### Scenario 2: Finite Buffers and Retransmissions

* **Setup:** Router has finite buffers; dropped packets are retransmitted ($\lambda_{\text{in}}' > \lambda_{\text{in}}$).
* **Costs:**
  1. Sender must retransmit dropped packets, consuming link capacity.
  2. Unneeded retransmissions caused by premature timeouts waste link throughput on packets already delivered.

#### Scenario 3: Multi-Hop Paths and Shared Links

* **Setup:** Four hosts send traffic over multi-hop paths sharing routers.
* **Cost:** When a packet is dropped at a downstream router after traversing several upstream links, all transmission capacity spent on that packet on upstream links is completely wasted.

### 5.2 Classic TCP Congestion Control (AIMD)

Classic TCP employs end-to-end congestion control, inferring network congestion from packet loss without explicit router assistance. The sender constrains its unacknowledged data via a Congestion Window (cwnd):

$$\text{LastByteSent} - \text{LastByteAcked} \le \min(\text{cwnd}, \text{rwnd})$$

#### The AIMD Algorithm

* **Additive-Increase:** cwnd increases linearly by $1\text{ MSS}$ per RTT in the absence of loss.
* **Multiplicative-Decrease:** cwnd is halved upon detecting a loss event (via 3 duplicate ACKs).

```text
  cwnd
   ^
W  |         /|        /|        /|
   |        / |       / |       / |
W/2|  /\   /  |  /\  /  |  /\  /  |   "Sawtooth" Pattern
   | /  \ /   | /  \/   | /  \/   |
   +-----------------------------------> Time
```

#### The Three Main Congestion Control Phases

```text
                       Start: cwnd = 1 MSS
                                |
                                v
                      +-------------------+
                      |    SLOW START     |<---------------+
                      | (Exponential x2)  |                |
                      +-------------------+                |
                        |                 |                |
             cwnd >= ssthresh             | Timeout        | Timeout
                        |                 v                |
                        v           +-----------+          |
              +--------------------+|  TIMEOUT  |----------+
              | CONGESTION AVOID.  || (cwnd=1)  |
              | (Linear +1 MSS)    |+-----------+
              +--------------------+      ^
                        |                 | Timeout
            3 Dup ACKs  |                 |
                        v                 |
              +--------------------+      |
              |   FAST RECOVERY    |------+
              | (Halve cwnd/ssth)  |
              +--------------------+
```

1. **Slow Start:**
    * Initial state: $\text{cwnd} = 1\text{ MSS}$.
    * Exponential growth: cwnd increases by $1\text{ MSS}$ for every received ACK (doubling cwnd every RTT).
    * Ends when $\text{cwnd} \ge \text{ssthresh}$ (transitions to Congestion Avoidance), when 3 duplicate ACKs arrive (enters Fast Recovery), or when a timeout occurs ($\text{ssthresh} = \text{cwnd}/2$, $\text{cwnd} = 1\text{ MSS}$, restarts Slow Start).
2. **Congestion Avoidance:**
    * Linear growth: cwnd increases by $1\text{ MSS}$ per RTT (specifically, $\text{cwnd} \leftarrow \text{cwnd} + \text{MSS} \cdot (\text{MSS}/\text{cwnd})$ per ACK).
    * On 3 duplicate ACKs: Sets $\text{ssthresh} = \text{cwnd} / 2$, $\text{cwnd} = \text{ssthresh} + 3\text{ MSS}$, and enters Fast Recovery.
3. **Fast Recovery (TCP Reno):**
    * Increments cwnd by $1\text{ MSS}$ for each additional duplicate ACK received.
    * When an ACK for missing data arrives, sets $\text{cwnd} = \text{ssthresh}$ and transitions to Congestion Avoidance.
    * Note: TCP Tahoe lacks Fast Recovery, unconditionally resetting $\text{cwnd} = 1\text{ MSS}$ on both timeouts and 3 duplicate ACKs.

### 5.3 Modern Congestion Control Variants

#### TCP CUBIC (RFC 8312)

TCP CUBIC replaces the linear congestion-avoidance phase with a cubic growth function based on the time elapsed $t$ since the last loss event:

$$W(t) = C \cdot (t - K)^3 + W_{\text{max}}$$

Where $W_{\text{max}}$ is the congestion window size at the previous loss event, C is a scaling parameter, and K is the time needed to ramp back to $W_{\text{max}}$:

$$K = \sqrt[3]{\frac{W_{\text{max}} \cdot \beta}{C}}$$

* **Behavior:** Ramps up rapidly when far below $W_{\text{max}}$, flattens out near $W_{\text{max}}$ to probe cautiously, and accelerates growth if the network remains loss-free. Default congestion control protocol in Linux.

#### Explicit Congestion Notification (ECN - RFC 3168)

Network-assisted congestion control where routers mark bits in the IP header (ECN = 11) when queuing delay builds up. The receiver echoes this via the ECE flag in TCP ACKs. The sender reduces cwnd as if a loss occurred and sets the CWR flag in the next segment.

#### Delay-Based Protocols (TCP Vegas and Google BBR)

* **TCP Vegas:** Measures $\text{RTT}_{\text{min}}$. Decreases sending rate when measured throughput falls significantly below $\frac{\text{cwnd}}{\text{RTT}_{\text{min}}}$.
* **BBR (Bottleneck Bandwidth and Round-trip propagation time):** Models the network's maximum bandwidth and minimum RTT independently, avoiding bufferbloat by keeping link utilization high without filling router queues.

## 6. Evolution of Transport-Layer Functionality: QUIC and HTTP/3

To bypass the constraints of legacy operating system kernels and eliminate Head-of-Line (HOL) blocking inherent in TCP streams, Google developed QUIC (Quick UDP Internet Connection), now standardized by IETF as the foundation for HTTP/3.

```text
Standard Secure Web Stack                  HTTP/3 Stack
+-----------------------+              +-----------------------+
|       HTTP/1.1        |              |        HTTP/3         |
+-----------------------+              +-----------------------+
|       TLS 1.3         |              |         QUIC          |
+-----------------------+              | (Streams, Crypto, RDT)|
|         TCP           |              +-----------------------+
+-----------------------+              |         UDP           |
|          IP           |              +-----------------------+
                                       +-----------------------+
```

### Key Architectural Advances of QUIC

1. **Application-Layer Implementation over UDP:** QUIC runs in user space over UDP, enabling rapid protocol updates without operating system kernel upgrades.
2. **0-RTT / 1-RTT Connection & Security Handshake:** Combines transport connection setup and TLS 1.3 cryptographic key exchange into a single handshake, reducing startup latency to 0-RTT for returning clients.
3. **Independent Multistreaming without Head-of-Line (HOL) Blocking:** Multiple application streams are multiplexed within a single QUIC connection. Packet loss on one stream does not pause processing on unrelated streams.
4. **Connection IDs for Seamless Mobility:** Connections are identified by a 64-bit Connection ID rather than a 4-tuple, allowing mobile devices to switch between Wi-Fi and cellular networks without dropping ongoing streams.

## 7. Practice Examination & Review Quiz

### Short-Answer Questions

1. **Question:** Explain why a transport protocol can provide reliable data transfer even over an unreliable network layer.
2. **Question:** How does a receiving host demultiplex an arriving UDP segment versus a TCP segment?
3. **Question:** Calculate the UDP checksum for the two 8-bit words 10101011 and 01001101. Show all work.
4. **Question:** What was the fatal flaw in protocol rdt2.0, and how did rdt2.1 solve it?
5. **Question:** Explain the difference between cumulative acknowledgments in Go-Back-N and individual acknowledgments in Selective Repeat.
6. **Question:** In Selective Repeat, why must the window size N be less than or equal to half the sequence number space size?
7. **Question:** Suppose a TCP connection has an estimated RTT of $80\text{ ms}$ and a DevRTT of $10\text{ ms}$. Calculate the TimeoutInterval.
8. **Question:** Why does TCP use 3 duplicate ACKs to trigger Fast Retransmit instead of retransmitting on the very first duplicate ACK?
9. **Question:** What is the zero-window probing mechanism in TCP flow control, and why is it necessary?
10. **Question:** Describe how SYN Cookies defend a server against SYN Flood Denial-of-Service attacks.

### Detailed Answer Key

1. **Answer:** Transport protocols run on end systems and can implement recovery mechanisms—such as error-detection checksums, sequence numbers, timers, acknowledgments, and retransmissions—that detect lost or garbled network datagrams and automatically retransmit missing data.
2. **Answer:** A UDP segment is demultiplexed based on a 2-tuple (Destination IP, Destination Port). A TCP segment is demultiplexed based on a 4-tuple (Source IP, Source Port, Destination IP, Destination Port).
3. **Answer:**

    ```text
    \begin{array}{r@{\quad}l} 10101011 & \
      * 01001101 & \ \hline 11111000 & \text{(Sum without carry-out)} \ \end{array}
    ```

    Taking the 1s complement: $\sim(11111000) = \mathbf{00000111}$.
4. **Answer:** Protocol rdt2.0 failed if ACK or NAK control packets were corrupted, because the sender could not tell if the receiver received the data. Protocol rdt2.1 added 1-bit sequence numbers (0 and 1) to data packets so receivers can recognize retransmitted duplicates.
5. **Answer:** In GBN, an ACK for sequence number n cumulatively acknowledges all packets up to and including n. In SR, an ACK for sequence number n acknowledges only packet n, and out-of-order packets are buffered.
6. **Answer:** If $N > 2^{k-1}$, the receive window and send window overlap in sequence space, making a retransmitted old packet indistinguishable from a brand-new packet.
7. **Answer:**

    $$\text{TimeoutInterval} = \text{EstimatedRTT} + 4 \cdot \text{DevRTT} = 80\text{ ms} + 4(10\text{ ms}) = \mathbf{120\text{ ms}}$$
8. **Answer:** A single duplicate ACK frequently occurs due to minor out-of-order packet routing. Waiting for 3 duplicate ACKs ensures with high probability that a packet was actually lost rather than slightly delayed.
9. **Answer:** When $\text{rwnd} = 0$, the sender stops transmitting. The sender sends 1-byte probe segments to force the receiver to return ACKs with updated $\text{rwnd}$ values, preventing deadlocks when buffer space frees up.
10. **Answer:** The server refrains from allocating connection state upon receiving a SYN. Instead, it generates an initial sequence number (cookie) derived from a secret hash of IP/port tuples and a timestamp. When the client returns an ACK with $\text{ack} = \text{cookie} + 1$, the server verifies the hash and creates the connection socket.

## 8. Synthesis & Discussion Prompts

1. **Trade-offs in Transport Protocol Design:** Evaluate why real-time video communications are shifting from legacy pure UDP streams to HTTP-based adaptive streaming (DASH over TCP) and modern QUIC stacks.
2. **Impact of Link Latency on Pipelining:** Compare how satellite links ($250\text{ ms}$ propagation delay) impact Go-Back-N vs. Selective Repeat performance compared to terrestrial fiber links.
3. **Congestion Control Evolution:** Discuss why loss-based congestion control (Reno) performs poorly on links with high random bit-error rates (such as wireless networks) compared to delay-based protocols (BBR).
4. **Fairness vs. Multi-Connection Exploitation:** Analyze how web browsers opening parallel TCP connections bypass TCP's AIMD fairness guarantees.
5. **Architectural Migration to QUIC:** Examine the architectural trade-offs of shifting transport mechanisms out of the operating system kernel and into application-level software stacks.

## 9. Comprehensive Glossary of Core Terms

* **Additive-Increase, Multiplicative-Decrease (AIMD):** A congestion control algorithm that linearly increases the congestion window until packet loss occurs, at which point the window is halved.
* **Automatic Repeat reQuest (ARQ):** Protocols that use error detection, positive/negative acknowledgments, and retransmissions to achieve reliable data transfer over unreliable channels.
* **Bandwidth-Delay Product (BDP):** The product of a link's transmission rate and round-trip time ($R \cdot \text{RTT}$), representing the maximum volume of unacknowledged data in transit.
* **Congestion Window (cwnd):** A sender-side TCP state variable restricting the maximum volume of unacknowledged data allowed in the network.
* **Cumulative Acknowledgment:** An acknowledgment scheme indicating that all bytes or packets up to the specified sequence number have been successfully received.
* **Demultiplexing:** Delivering transport-layer segment payloads to the correct application socket based on header fields.
* **End-to-End Principle:** A network design framework stating that complex protocol functions should be placed in end systems rather than intermediate network nodes.
* **Explicit Congestion Notification (ECN):** A network-assisted mechanism where routers mark IP header bits to warn end systems of impending congestion.
* **Fast Retransmit:** A TCP mechanism that retransmits a missing segment upon receiving 3 duplicate ACKs before its timer expires.
* **Flow Control:** A protocol mechanism matching the sender's transmission rate to the receiver's application reading rate to prevent buffer overflow.
* **Go-Back-N (GBN):** A pipelined protocol permitting up to N unacknowledged packets, using cumulative ACKs and retransmitting all unacknowledged packets upon timeout.
* **Maximum Segment Size (MSS):** The maximum amount of application-layer data encapsulated in a single TCP segment.
* **Multiplexing:** Gathering data from multiple sockets, adding transport headers, and passing segments down to the network layer.
* **QUIC:** A UDP-based multiplexed transport protocol featuring zero-delay handshakes, stream-isolated reliability, and integrated encryption.
* **Receive Window (rwnd):** A receiver-advertised TCP header field indicating available receive buffer space for flow control.
* **Selective Repeat (SR):** A pipelined protocol where receivers individually ACK packets and senders retransmit only lost packets.
* **SYN Cookie:** A defense against SYN flood attacks using cryptographically generated sequence numbers to defer connection state allocation.
* **Three-Way Handshake:** A 3-step segment exchange (SYN, SYNACK, ACK) used to establish a TCP connection and initialize sequence numbers.