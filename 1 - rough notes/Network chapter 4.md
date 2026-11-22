# Computer Networking: Chapter 4 Study Notes (The Network Layer: Data Plane)

## 1. Overview of the Network Layer & Service Models

### 1.1 Data Plane vs. Control Plane Mechanics

The primary function of the network layer is deceptively simple: transport packets from a sending host to a receiving host across a mesh of interconnected routers. To achieve this, the network layer is fundamentally partitioned into two distinct operational planes:

* **The Data Plane (Forwarding):** The per-router, local processing mechanism that moves an incoming packet from an input link interface to the appropriate output link interface. Forwarding operates on extremely fast time scales (nanoseconds) and is implemented almost exclusively in dedicated hardware line cards.
* **The Control Plane (Routing):** The network-wide logic that determines the end-to-end paths taken by packets from their source to their destination. Routing operates on longer time scales (milliseconds to seconds) and is implemented in software executing on routing processors or centralized controllers.

```text
+-------------------------------------------------------------------+
|                        CONTROL PLANE                              |
|   Software: Routing Algorithms (OSPF, BGP) / SDN Controller       |
|   Operates on Millisecond-to-Second Timescales                     |
+-------------------------------------------------------------------+
                                  | Forwarding Table Updates
                                  v
+-------------------------------------------------------------------+
|                         DATA PLANE                                |
|   Hardware: Line Cards, Input/Output Ports, Switching Fabric      |
|   Operates on Nanosecond Timescales (Per-Packet Matching)         |
+-------------------------------------------------------------------+
```

#### Traditional IP Forwarding vs. Software-Defined Networking (SDN)

In traditional IP networking, the data plane and control plane are monolithically integrated inside every individual router. Each router runs its own routing protocol instance (e.g., OSPF or BGP), calculates its own local forwarding table, and executes destination-based forwarding.

In Software-Defined Networking (SDN), the data plane and control plane are explicitly decoupled. The control plane is centralized into a remote SDN controller (or logically centralized controller cluster). The controller computes flow tables in software and installs them into the hardware flow tables of underlying packet switches via standard protocols such as OpenFlow.

```text
Traditional Router Architecture:
+-----------------------------------+
| Control Plane (Routing Software)  |  <- Monolithic Integration
+-----------------------------------+     Inside Each Switch/Router
| Data Plane (Forwarding Hardware)  |
+-----------------------------------+

SDN Architecture:
+-----------------------------------+
| Centralized Remote SDN Controller |  <- Decoupled Control Plane
+-----------------------------------+
                  | (e.g., OpenFlow Protocol)
                  v
+-----------------------------------+
| Data Plane Switch (Flow Tables)   |  <- Decoupled Data Plane
+-----------------------------------+
```

### 1.2 Network Service Models

A network service model defines the end-to-end performance guarantees provided by the network layer for packets traveling between host systems.

| Service Model Architecture | Network Service Model Type | Delay Guarantees | Loss Guarantees | Bandwidth Guarantees | In-Order Delivery | Congestion Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| Internet IP | Best-Effort Service | None | None | None | No (Packets may reorder) | None (at network layer; handled at transport) |
| ATM / VC | Constant Bit Rate (CBR) | Guaranteed bounded delay | Guaranteed zero loss | Guaranteed fixed rate | Yes | None needed (reserved capacity) |
| ATM / VC | Available Bit Rate (ABR) | Unbounded | Bounded probability | Guaranteed minimum rate | Yes | Explicit network congestion signals |

The Internet relies entirely on the **Best-Effort Service Model**. Datagrams are transmitted without prior resource reservation; there are no guarantees regarding delivery, end-to-end delay, jitter, throughput, or packet ordering. This architectural simplicity is the core reason for the Internet's extraordinary scalability and rapid adoption across diverse underlying physical link media.

## 2. Router Architecture & Packet Processing

### 2.1 Component Architecture & Line-Card Shadow Memory

A high-speed router consists of four primary structural components:

```text
                  +-----------------------------------+
                  |     Routing Processor (Software)  |
                  +-----------------------------------+
                                    | (PCI Bus Update)
        +---------------------------+---------------------------+
        |                                                       |
        v                                                       v
  +-----------+           +-------------------+           +-----------+
  | Input     | --------> |                   | --------> | Output    |
  | Port      |           | Switching Fabric  |           | Port      |
  +-----------+           |                   |           +-----------+
  | Input     | --------> |                   | --------> | Output    |
  | Port      |           |                   |           | Port      |
  +-----------+           +-------------------+           +-----------+
```

1. **Input Ports:** Terminate physical links, perform link-layer demultiplexing, and execute lookup functions to determine output ports.
2. **Switching Fabric:** The internal hardware interconnect that switches packets from input ports to output ports.
3. **Output Ports:** Queue outgoing packets, perform link-layer framing, and transmit signals onto outgoing physical links.
4. **Routing Processor:** Executes control-plane software (routing protocols, network management, interface maintenance) and updates forwarding tables.

#### Line-Card Shadow Memory

To prevent the central routing processor from becoming a bottleneck during packet lookup, modern routers maintain a shadow copy of the forwarding table on every input line card. Forwarding tables are computed by the routing processor and copied across an internal bus (e.g., PCIe) to memory on each line card. Lookups are thus executed entirely locally on line card hardware in nanoseconds without central processor intervention.

### 2.2 Input Port Processing & TCAM Lookup

An input port executes three sequential layer functions:

```text
+---------------------+     +-----------------------+     +------------------------+
| Physical Layer      | --> | Link-Layer Protocol   | --> | Lookup & Forwarding    |
| Line Termination    |     | (Framing/Decap)       |     | (Shadow Table / TCAM)  |
+---------------------+     +-----------------------+     +------------------------+
```

1. **Physical Layer Termination:** Receives physical signals from the incoming link.
2. **Link-Layer Processing:** Decapsulates link-layer frames (e.g., Ethernet) and checks frame integrity.
3. **Lookup & Forwarding:** Extracts destination addresses and determines target output ports.

#### Longest Prefix Matching (LPM)

Internet routers forward IPv4 datagrams using **Longest Prefix Matching**. When a packet arrives, the router searches its forwarding table for the entry whose address prefix matches the maximum number of leading bits of the destination IP address.

##### Longest Prefix Matching Lookup Example

Consider a router with the following forwarding table entries:

| Address Prefix Entry | Output Link Interface |
| --- | --- |
| 11001000 00010111 00010000 00000000 / 20 | Interface 0 |
| 11001000 00010111 00011000 00000000 / 21 | Interface 1 |
| 11001000 00010111 00011000 10000000 / 22 | Interface 2 |
| Otherwise | Interface 3 (Default) |

Now consider two arriving datagrams:

* **Datagram A Destination:** 11001000 00010111 00011000 10100001
  * Matches Prefix 1 (first 20 bits match).
  * Matches Prefix 2 (first 21 bits match).
  * Matches Prefix 3 (first 22 bits match).
  * **Result:** The longest matching prefix is Prefix 3 (22 bits). The datagram is forwarded to Interface 2.
* **Datagram B Destination:** 11001000 00010111 00011001 01000000
  * Matches Prefix 1 (first 20 bits match).
  * Matches Prefix 2 (first 21 bits match).
  * Does NOT match Prefix 3 (bit 22 is 0, but Prefix 3 requires 1).
  * **Result:** The longest matching prefix is Prefix 2 (21 bits). The datagram is forwarded to Interface 1.

##### Ternary Content Addressable Memory (TCAM)

Searching millions of prefixes in software via linear search or tries is far too slow for 100+ Gbps links. High-speed routers utilize Ternary Content Addressable Memory (TCAM). A 32-bit IP address is presented to the TCAM hardware, which compares all entries simultaneously in parallel.

* **Ternary Values:** TCAM cells store three states: 0, 1, and \* (don't care / mask bit).
* **Speed:** TCAM returns the matching output interface in $O(1)$ constant time (a few nanoseconds). High-end enterprise routers (such as Cisco Catalyst Series) utilize TCAM arrays capable of storing over 1,000,000 table entries.

### 2.3 Switching Fabrics

The switching fabric transfers packets from input queues to output queues. Three classic switching architectures exist:

```text
Switching via Memory           Switching via Bus            Switching via Crossbar
+-------------------+        +-------------------+        +-------------------+
|  Input -> Memory  |        |  Input -> Shared  |        | Input  |  |  |    |
|  Memory -> Output |        |  Bus -> Output    |        | -------+--+--+--- |
|  (CPU Bottleneck) |        |  (Bus Contention) |        | Output |  |  |    |
+-------------------+        +-------------------+        +-------------------+
```

1. **Switching via Memory:**
    * Early routers used standard general-purpose computers. An incoming packet triggered an interrupt, copying the packet from the input line card into main memory across the system bus. The CPU extracted the destination address, looked up the output port, and copied the packet to the output line card buffer.
    * **Constraint:** Throughput is strictly bounded by memory bandwidth. Because every packet must traverse the system bus twice, total switching rate is $\le \frac{\text{Memory Bandwidth}}{2}$.
2. **Switching via a Bus:**
    * Input ports transfer packets directly to output ports across a shared backplane bus without CPU intervention. Input line cards prepend an internal switch header containing the output port index. All output line cards receive the packet on the shared bus, but only the card matching the switch header keeps it.
    * **Constraint:** Switching speed is limited by shared bus bandwidth. Only one packet can traverse the bus at a time. If multiple packets arrive simultaneously on different input ports, all but one must wait in input buffers. Total router throughput equals the bus speed (e.g., 100 Gbps).
3. **Switching via an Interconnection Network (Crossbar):**
    * An $N \times N$ crossbar switch consists of N horizontal buses connected to N vertical buses through electronic switches at each cross-point.
    * **Advantage:** Non-blocking architecture. Multiple packets can be switched in parallel, provided no two packets are destined for the same output port at the same time. Advanced multi-stage switching fabrics scale to thousands of line cards.

### 2.4 Queuing, Buffer Sizing, & Active Queue Management (AQM)

#### Where Does Queuing Occur?

Queuing occurs at both input ports and output ports whenever packet arrival rates temporarily exceed switching or transmission capacity.

```text
Input Queuing (HOL Blocking):              Output Queuing (Buffer Overflow):
Input Port 1 [ A->Out1 ]                   Switching Fabric 
Input Port 2 [ B->Out1 ][ C->Out2 ] --->  [ Packet A ][ Packet B ] ---> Output Port 1
               ^ (Blocked by B!)           (High arrival rate -> Buffer full -> Loss)
```

#### Head-of-Line (HOL) Blocking

Input queuing occurs when the switching fabric speed is slower than the aggregate line rate of all input ports. A critical bottleneck in input-queued switches is **Head-of-Line (HOL) Blocking**: a packet at the front of an input queue is delayed because the target output port is currently busy. As a result, all packets queued behind it are blocked from moving forward, even if their own destination output ports are completely idle.

#### Output Queuing & Loss

Output queuing occurs when packets arrive from the switching fabric at a rate exceeding the outgoing link's transmission speed. When the output port buffer fills completely, arriving packets must either be dropped or cause older packets to be evicted.

#### Router Buffer Sizing Formulas

Determining the optimal buffer capacity B at a router output port is a critical design decision:

* **Traditional Rule of Thumb:** $B = \text{RTT} \cdot C$ where $\text{RTT}$ is the average round-trip time of TCP flows traversing the link and C is the link capacity. For a 10 Gbps link with a 250 ms average RTT, the required buffer size is $2.5 \text{ Gbits}$ (312.5 MB).
* **High-Volume Flow Scaling (Stanford Formula):** When a large number N of independent TCP flows share an bottleneck link, buffer requirements decrease significantly due to statistical multiplexing:

  $$B = \frac{\text{RTT} \cdot C}{\sqrt{N}}$$

#### Active Queue Management (AQM) & Drop Policies

When a router buffer fills, it must enforce a drop policy:

* **Tail Drop:** The default, reactive policy. When the buffer is full, any newly arriving packet is dropped. Tail drop causes **Global Synchronization**: multiple TCP flows simultaneously drop packets, enter slow start, and reduce their congestion windows in lockstep, severely degrading link utilization.
* **Random Early Detection (RED):** An Active Queue Management (AQM) algorithm that proactively drops (or marks) packets before the buffer fills. RED calculates an exponentially weighted moving average queue length $\text{Avg}_q$:
  * If $\text{Avg}_q < \text{Min}_{th}$: Accept the packet.
  * If $\text{Min}_{th} \le \text{Avg}_q \le \text{Max}_{th}$: Drop the packet with a probability $p_p$ that increases linearly as $\text{Avg}_q$ approaches $\text{Max}_{th}$.
  * If $\text{Avg}_q > \text{Max}_{th}$: Drop all incoming packets (Tail drop).
* **Explicit Congestion Notification (ECN):** Rather than dropping packets, ECN-enabled routers mark two bits in the IPv4/IPv6 header (the ECN field in ToS/Traffic Class) when congestion begins. The receiving endpoint echoes the congestion mark back to the sender via the ECE (ECN-Echo) flag in the TCP header, instructing the sender to reduce its congestion window without incurring packet loss.

### 2.5 Packet Scheduling Algorithms

Packet scheduling governs the order in which queued packets are transmitted onto an outgoing link.

```text
FIFO Scheduling:
[ Pk1 ][ Pk2 ][ Pk3 ] ---> [ Link Transmit ] (In Order of Arrival)

Priority Scheduling:
High Queue [ Pk1_High ][ Pk2_High ] ---> [ Served First ]
Low Queue  [ Pk1_Low  ]             ---> [ Served Only When High Queue Empty ]

Weighted Fair Queuing (WFQ):
Class 1 (Weight w1) ---+
Class 2 (Weight w2) ---+---> [ Round Robin Service Proportional to Weights ]
Class 3 (Weight w3) ---+
```

1. **First-In, First-Out (FIFO / FCFS):** Packets are served strictly in the order of their arrival.
2. **Priority Scheduling:** Packets are classified into priority queues (e.g., real-time VoIP vs. bulk SMTP). The link always transmits packets from the highest non-empty priority queue first (non-preemptive priority).
3. **Round Robin (RR):** Packets are sorted into service classes. A scheduler rotates cyclically among non-empty queues, serving one packet from each class in turn.
4. **Weighted Fair Queuing (WFQ):** A generalized round-robin scheduling algorithm widely used in QoS architectures. Each class i is assigned a numerical weight $w_i$. During any interval when packets are queued across all classes, class i is guaranteed a minimum normalized throughput share $R_i$ of link capacity R:

   $$R_i = R \cdot \frac{w_i}{\sum_{j} w_j}$$

## 3. The Internet Protocol (IPv4, Addressing, & IPv6)

### 3.1 IPv4 Datagram Format

An IPv4 datagram consists of a header portion followed by a payload portion.

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version|  IHL  |Type of Service|          Total Length         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Identification        |Flags|      Fragment Offset    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Time to Live |    Protocol   |        Header Checksum        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Source IP Address                       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Destination IP Address                     |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options (0 to 40 bytes)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                            Payload                            |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### Detailed IPv4 Header Field Breakdown

* **Version (4 bits):** Specifies the IP protocol version (set to 4 or 0100).
* **Internet Header Length (IHL / 4 bits):** Specifies the header length in 32-bit (4-byte) words. The standard header without options has an IHL value of 5 ($5 \times 4 = 20 \text{ bytes}$).
* **Type of Service (ToS / DiffServ / ECN - 8 bits):** Allows traffic classification (Differentiated Services) and explicit congestion notification (ECN).
* **Total Length (16 bits):** Total datagram length in bytes (header + payload). Maximum theoretical IPv4 datagram size is $2^{16} - 1 = 65{,}535 \text{ bytes}$.
* **Identification (16 bits), Flags (3 bits), Fragment Offset (13 bits):** Fields dedicated to IP fragmentation and reassembly.
  * **Flags:** Bit 0 is reserved. Bit 1 is DF (Don't Fragment). Bit 2 is MF (More Fragments).
  * **Fragment Offset:** Indicates where in the original unfragmented datagram this fragment belongs, measured in 8-byte (64-bit) units.
* **Time to Live (TTL - 8 bits):** Decremented by 1 at every router that forwards the datagram. If TTL drops to 0, the datagram is discarded and an ICMP "Time Exceeded" message is sent to the source, preventing endless routing loops.
* **Protocol (8 bits):** Indicates the upper-layer transport protocol payload (6 for TCP, 17 for UDP, 1 for ICMP).
* **Header Checksum (16 bits):** Stores the 1's complement sum of all 16-bit words in the IP header. Must be recomputed at every hop because TTL (and options) change.
* **Source IP Address (32 bits):** 32-bit IP address of the originating host.
* **Destination IP Address (32 bits):** 32-bit IP address of the target destination host.
* **Options (0 to 40 bytes):** Variable-length optional fields (e.g., record route, source routing). Padded to a 32-bit boundary.

### 3.2 IP Fragmentation & Reassembly Mechanics

Different link-layer protocols can transmit frames of different sizes. The maximum amount of data a link-layer frame can carry is called the **Maximum Transmission Unit (MTU)**. When an IP router receives a datagram whose total size exceeds the MTU of the outgoing link interface, it must divide the payload into smaller datagrams called fragments.

#### Fragmentation Rules

1. Fragmentation occurs at routers along the path (or at the sending host).
2. Reassembly occurs exclusively at the destination host, never at intermediate routers.
3. Each fragment receives its own IP header carrying the original datagram's Identification value.
4. The Fragment Offset field stores the starting payload position divided by 8.
5. All fragments except the last have $\text{MF} = 1$. The final fragment has $\text{MF} = 0$.

#### Worked Numerical Fragmentation Example

A host transmits a 4,000-byte IPv4 datagram (20-byte IP header + 3,980 bytes payload) over a link with an MTU of 1,500 bytes.

* **Available Payload Capacity per Fragment:** $1500 - 20 = 1480 \text{ bytes}$.
* **Check Offset Division:** $1480 \div 8 = 185$ (must be a whole integer).
* **Number of Fragments Required:** $\lceil 3980 / 1480 \rceil = 3 \text{ fragments}$.

```text
Original Datagram:
[ IP Header (20B) ][ Payload: 3980 Bytes ] Total Length = 4000
ID = 777, MF = 0, Offset = 0

Fragment 1:
[ IP Header (20B) ][ Payload: 1480 Bytes ] Total Length = 1500
ID = 777, MF = 1, Offset = 0

Fragment 2:
[ IP Header (20B) ][ Payload: 1480 Bytes ] Total Length = 1500
ID = 777, MF = 1, Offset = 185  (1480 / 8)

Fragment 3:
[ IP Header (20B) ][ Payload: 1020 Bytes ] Total Length = 1040
ID = 777, MF = 0, Offset = 370  ((1480 + 1480) / 8)
```

| Fragment Parameter | Original Datagram | Fragment 1 | Fragment 2 | Fragment 3 |
| --- | --- | --- | --- | --- |
| Total Length | 4,000 bytes | 1,500 bytes | 1,500 bytes | 1,040 bytes |
| Payload Size | 3,980 bytes | 1,480 bytes | 1,480 bytes | 1,020 bytes |
| Identification | 777 | 777 | 777 | 777 |
| More Fragments (MF) | 0 | 1 | 1 | 0 |
| Fragment Offset | 0 | 0 | 185 (1480/8) | 370 (2960/8) |

### 3.3 Subnetting, CIDR Notation, & Route Aggregation

#### IPv4 Addressing & Interface Structure

An IP address is a 32-bit binary number uniquely identifying a network interface (a boundary between a host/router and a physical link). IPv4 addresses are formatted in dotted-decimal notation: four 8-bit octets separated by decimal points (e.g., 192.168.1.1).

#### Subnet Definition

A subnet is a isolated network segment connecting multiple device interfaces without passing through an intervening router.

```text
       +-----------------------------------+
       |          Subnet 1                 |
       |  Host 1: 223.1.1.1/24             |
       |  Host 2: 223.1.1.2/24             |
       |  Router Interface 1: 223.1.1.4/24 |
       +-----------------------------------+
                         | (Router R1)
       +-----------------------------------+
       |          Subnet 2                 |
       |  Router Interface 2: 223.1.2.9/24 |
       |  Host 3: 223.1.2.1/24             |
       +-----------------------------------+
```

#### Classless Inter-Domain Routing (CIDR)

CIDR (a.b.c.d/x) replaced rigid legacy classful addressing (Class A, B, C).

* **Subnet Prefix (/x):** The leading x bits of the 32-bit address constitute the network prefix. Routers outside the subnet use only these x bits to route packets to the subnet.
* **Host ID ($32 - x$ bits):** The remaining $32 - x$ bits distinguish individual interfaces within the subnet.
* **Subnet Mask:** A 32-bit mask consisting of x contiguous ones followed by $32 - x$ zeros. For /24, the mask is 255.255.255.0.

#### Route Aggregation (Supernetting)

CIDR enables route aggregation: an Internet Service Provider (ISP) uses a single prefix address to advertise multiple internal subnets to the global Internet.

```text
Organization 0: 200.23.16.0/23 --+
Organization 1: 200.23.18.0/23 --|
Organization 2: 200.23.20.0/23 --+--> ISP Advertises Aggregate Prefix:
Organization 3: 200.23.22.0/23 --|     200.23.16.0/20 to Global Internet
Organization 4: 200.23.24.0/23 --|
Organization 5: 200.23.26.0/23 --|
Organization 6: 200.23.28.0/23 --|
Organization 7: 200.23.30.0/23 --+
```

If Organization 1 switches to a new ISP ("Fly-By-Night ISP") while keeping its address block 200.23.18.0/23, the global routing table handles this via Longest Prefix Matching:

* Original ISP advertises: 200.23.16.0/20
* New ISP advertises specific route: 200.23.18.0/23
* Routers globally send packets for Organization 1 to the new ISP because /23 is a longer prefix match than /20.

### 3.4 Dynamic Host Configuration Protocol (DHCP - The DORA Process)

DHCP is an application-layer protocol (utilizing UDP ports 67 for server and 68 for client) that automatically configures network parameters for newly connected hosts ("plug-and-play" or zeroconf networking).

```text
Client (0.0.0.0:68)                               Server (255.255.255.255:67)
       |                                                      |
       | ------------- 1. DHCP Discover (Broadcast) --------> |
       |                                                      |
       | <------------ 2. DHCP Offer (Broadcast) ------------ |
       |                                                      |
       | ------------- 3. DHCP Request (Broadcast) ---------> |
       |                                                      |
       | <------------ 4. DHCP ACK (Broadcast) -------------- |
       |                                                      |
```

1. **DHCP Discover:** The client broadcasts a message (src=0.0.0.0:68, dst=255.255.255.255:67) requesting IP parameters.
2. **DHCP Offer:** A DHCP server responds with a broadcast offer containing an available IP address, subnet mask, IP lease time, and transaction ID.
3. **DHCP Request:** The client broadcasts a request echoing the chosen server's offer, implicitly rejecting offers from any other responding DHCP servers.
4. **DHCP ACK:** The server confirms the IP allocation lease via a final broadcast ACK.

**Parameters Provided by DHCP:** Assigned IP address, subnet mask, default gateway (first-hop router IP), and local DNS server address.

### 3.5 Network Address Translation (NAT) & Translation Tables

To combat IPv4 address exhaustion, private network address ranges were reserved for local use (RFC 1918):

* 10.0.0.0 / 8
* 172.16.0.0 / 12
* 192.168.0.0 / 16

```text
Private LAN (10.0.0.0/24)             NAT Router (WAN IP: 138.76.29.7)          Public Internet
Host 10.0.0.1:3345 ---------------> [ Outbound: Src 10.0.0.1:3345 ]
                                    [ Translates Src to 138.76.29.7:5001 ] ---> Web Server 128.119.40.186:80
                                    
                                    NAT Translation Table:
                                    10.0.0.1:3345 <-> 138.76.29.7:5001
                                    
Host 10.0.0.1:3345 <--------------- [ Inbound: Dst 138.76.29.7:5001 ]
                                    [ Translates Dst to 10.0.0.1:3345 ] <----- Web Server 128.119.40.186:80
```

#### NAT Operation

A NAT-enabled router presents a single public IP address (138.76.29.7) to the public Internet while managing hundreds of private internal hosts (10.0.0.x).

* **Outbound Mapping:** When host 10.0.0.1 sends a packet from port 3345 to 128.119.40.186:80, the NAT router rewrites the source IP to 138.76.29.7 and replaces the source port with an unused NAT port (e.g., 5001).
* **Translation Table:** Maps (Internal IP: 10.0.0.1, Internal Port: 3345) <-> (External NAT IP: 138.76.29.7, NAT Port: 5001).
* **Inbound Mapping:** Returning packets arriving at 138.76.29.7:5001 are mapped back to 10.0.0.1:3345 and forwarded into the LAN.

#### Architectural Controversies

1. **Layer Violation:** NAT routers inspect and modify Layer 4 TCP/UDP port numbers in a Layer 3 network device.
2. **End-to-End Principle Violation:** Host nodes are no longer directly addressable from the public Internet, interfering with incoming P2P connections and VoIP calls (requiring NAT traversal techniques like STUN/UPnP).

### 3.6 IPv6 Architecture, Fixed Header, & Transition Protocols

#### IPv6 Motivation

The explosive growth of connected internet endpoints exhausted IPv4's 32-bit address space ($2^{32} \approx 4.3 \text{ billion addresses}$). IPv6 utilizes 128-bit addresses ($2^{128} \approx 3.4 \times 10^{38}$ addresses).

```text
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version| Traffic Class |           Flow Label                  |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Payload Length        |  Next Header  |   Hop Limit   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                         Source Address                        +
|                           (128 bits)                          |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
+                                                               +
|                                                               |
+                      Destination Address                      +
|                           (128 bits)                          |
+                                                               +
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### Streamlined Fixed 40-Byte Header Format

* **Version (4 bits):** Set to 6 (0110).
* **Traffic Class (8 bits):** Equivalent to IPv4 ToS/DiffServ.
* **Flow Label (20 bits):** Identifies specific packet flows requiring special QoS handling.
* **Payload Length (16 bits):** Number of bytes following the fixed 40-byte header.
* **Next Header (8 bits):** Identifies upper-layer protocol (TCP/UDP) or points to an IPv6 Extension Header.
* **Hop Limit (8 bits):** Decremented at each hop; drops packet when 0 (equivalent to IPv4 TTL).
* **Source & Destination Addresses (128 bits each):** 16-byte IPv6 addresses formatted in hexadecimal colon notation (e.g., 2001:0db8:85a3:0000:0000:8a2e:0370:7334).

#### Removed Header Fields (Why IPv6 is Faster)

1. **No Header Checksum:** Removed entirely because link layers and transport layers already compute checksums. Eliminates per-hop checksum recalculation.
2. **No Base Fragmentation Fields:** Routers do not fragment IPv6 datagrams. If a datagram exceeds MTU, the router drops it and sends an ICMPv6 "Packet Too Big" message back to the sender, enforcing Path MTU Discovery.
3. **No Options Field:** Options are handled via chainable Extension Headers.

#### Transition Strategies: Dual Stack vs. Tunneling

Since thousands of core routers cannot be upgraded simultaneously, two transition protocols exist:

* **Dual Stack:** IPv6-capable nodes maintain both complete IPv4 and IPv6 protocol stacks simultaneously.
* **Tunneling:** An IPv6 datagram is encapsulated as the payload inside a standard IPv4 datagram when passing through intermediate IPv4-only routers.

```text
IPv6 Node A ---> [ IPv6 Packet ] ---> IPv6/IPv4 Tunnel Router B
                                             |
                                  Encapsulates IPv6 inside IPv4
                                             v
IPv6 Node F <--- [ IPv6 Packet ] <--- IPv6/IPv4 Tunnel Router E <--- [ IPv4 (Payload: IPv6) ] <--- IPv4 Routers
```

## 4. Generalized Forwarding & SDN (OpenFlow)

### 4.1 Match-Plus-Action Abstraction

Traditional destination-based forwarding matches only the destination IP address and takes a single action (forward out an interface). Generalized Forwarding expands this into a flexible **match-plus-action** paradigm: match on criteria across multiple header fields (Layer 2, Layer 3, Layer 4) and execute arbitrary actions.

### 4.2 OpenFlow Flow Table Structure

In an OpenFlow switch, forwarding rules are maintained in a **Flow Table**. Each entry contains:

1. **Match Fields:** Header conditions to match against incoming packets.
2. **Counters:** Metrics tracking matching packets, bytes, and table entry duration.
3. **Actions:** Instructions executed when a match occurs.

```text
+---------------------------------------------------------------------------------------+
|                                OPENFLOW FLOW TABLE ENTRY                              |
+------------------------------------+-------------------------+------------------------+
|            MATCH FIELDS            |        COUNTERS         |        ACTIONS         |
|  12 Protocol Fields (L2, L3, L4)    |  Packet Count, Bytes    |  Forward, Drop, Modify |
+------------------------------------+-------------------------+------------------------+
```

#### OpenFlow 1.0 Match Fields

OpenFlow 1.0 supports matching across 12 distinct header fields:

```text
+-------------+-------------+-------------+-------------+-------------+-------------+
| Ingress Port| MAC Source  | MAC Dest    | Eth Type    | VLAN ID     | VLAN Priority|
+-------------+-------------+-------------+-------------+-------------+-------------+
| IP Source   | IP Dest     | IP Protocol | IP ToS      | TCP/UDP Src | TCP/UDP Dst |
+-------------+-------------+-------------+-------------+-------------+-------------+
```

#### OpenFlow Action Set

* **Forwarding:** Send packet to a physical port, broadcast, multicast, or encapsulate and send to the remote SDN controller.
* **Dropping:** An entry with no specified action drops matched packets (Firewall behavior).
* **Modify-Field:** Rewrite header fields (e.g., re-writing MAC addresses, IP addresses, or port numbers) before forwarding (NAT or VLAN mapping behavior).

### 4.3 OpenFlow Rule Examples

#### Example 1: Custom Path Routing (Bypassing Links)

Direct traffic from host h5 (10.3.0.5) to host h3 (10.2.0.3) through switch s1 instead of s2.

* **Switch s3 Rule:**
  * Match: IP Dst = 10.2.0.3
  * Action: Forward(Port 3) (toward switch s1)
* **Switch s1 Rule:**
  * Match: IP Dst = 10.2.0.3
  * Action: Forward(Port 2) (toward switch s2)

#### Example 2: Simple Firewalling

Block all Web traffic (HTTP/80) originating from host h1 (10.1.0.1).

* **Switch s1 Flow Table Entry:**
  * Match: IP Src = 10.1.0.1, IP Proto = 6 (TCP), TCP Dst Port = 80
  * Action: Drop

## 5. Middleboxes & Architectural Principles

### 5.1 Middlebox Taxonomy

A **middlebox** is any network node sitting on the data path between a source and destination host that performs functions other than standard IP packet forwarding.

```text
                           +------------------------+
                           |       MIDDLEBOXES      |
                           +------------------------+
                                       |
    +-------------------+--------------+--------------+-------------------+
    |                   |                             |                   |
    v                   v                             v                   v
+-------+       +---------------+             +---------------+   +---------------+
| NAT   |       | Firewalls/IDS |             | Load Balancers|   | Caches/WAN Acc|
+-------+       +---------------+             +---------------+   +---------------+
```

* **Network Address Translators (NAT):** Modify source/destination IP addresses and ports to manage private address spaces.
* **Firewalls & Intrusion Detection Systems (IDS):** Inspect packet headers and payloads (Deep Packet Inspection) to drop malicious or unauthorized traffic.
* **Load Balancers:** Distribute incoming HTTP/TCP requests across server farms.
* **Web Caches & WAN Accelerators:** Store web content locally and compress streams to reduce link utilization.

### 5.2 Architectural Tensions & The End-to-End Principle

The proliferation of middleboxes highlights an ongoing debate in network engineering:

* **The End-to-End Principle:** States that complex network functions (e.g., error recovery, state management, security) should be implemented at the endpoints of the network rather than inside intermediate network nodes.
* **Middlebox Reality:** While middleboxes violate IP layering and end-to-end host reachability, they provide essential security filtering, performance enhancement, and address space preservation. SDN helps harmonize this tension by unifying middlebox rules into software-managed match-plus-action flow tables.

## 6. Practice Examination & Quantitative Exercises

### 6.1 Short-Answer Examination Questions

#### Question 1

Explain the fundamental operational differences between the data plane and the control plane. In your response, contrast their typical implementation media (hardware vs. software) and processing time scales.

#### Question 2

A router's forwarding table contains four entries shown below. Determine the output interface for arriving packets with the given destination IP addresses using Longest Prefix Matching:

* Prefix 1: 128.119.40.0 / 21 -> Interface 0
* Prefix 2: 128.119.40.0 / 22 -> Interface 1
* Prefix 3: 128.119.42.0 / 23 -> Interface 2
* Otherwise -> Interface 3 Datagram A: 128.119.41.15 Datagram B: 128.119.42.10

#### Question 3

Explain Head-of-Line (HOL) blocking in an input-queued switch. At which component of the router does HOL blocking occur, and what architectural design prevents it?

#### Question 4

A host transmits a 3,200-byte IPv4 datagram (including a 20-byte IP header) over a link with an MTU of 1,000 bytes. Calculate the Total Length, Payload Length, Identification, Fragment Offset, and More Fragments (MF) flag for each generated fragment.

#### Question 5

Calculate the maximum number of simultaneous active TCP connections that a single NAT router WAN interface IP address can support. What field in the transport header dictates this limit?

#### Question 6

List the four step messages in the DHCP handshake in order (the DORA process), specifying the source and destination IP addresses and UDP port numbers for each step.

#### Question 7

Compare IPv4 and IPv6 headers. Identify three header fields present in IPv4 that were completely eliminated in the base IPv6 header, and explain why each was removed.

#### Question 8

In Weighted Fair Queuing (WFQ), a router link with total capacity $R = 30 \text{ Mbps}$ serves three traffic classes with weights $w_1 = 3$, $w_2 = 2$, and $w_3 = 1$. Calculate the guaranteed throughput share for Class 2 when all three queues are continuously backlogged.

#### Question 9

Define the match-plus-action paradigm in generalized forwarding. List three distinct header fields that can be matched and three distinct actions that can be executed in OpenFlow 1.0.

#### Question 10

Distinguish between Dual Stack and Tunneling as IPv6 transition mechanisms. Under what network topology conditions is tunneling strictly required?

### 6.2 Detailed Answer Key & Mathematical Solutions

#### Solution 1

* **Data Plane:** Local per-router function that forwards packets from input interfaces to output interfaces. Implemented in dedicated hardware (ASICs/line cards) operating at nanosecond time scales.
* **Control Plane:** Network-wide logic determining end-to-end routing paths. Implemented in software (routing processors or SDN controllers) operating at millisecond to second time scales.

#### Solution 2

Convert IP addresses and prefixes to binary:

* Prefix 1 (/21): 128.119.00101000.\*
* Prefix 2 (/22): 128.119.00101000.\* (matches 128.119.40.0 to 128.119.43.255)
* Prefix 3 (/23): 128.119.00101010.\* (matches 128.119.42.0 to 128.119.43.255)
* Datagram A (128.119.41.15): Binary octet 3 is 00101001. Matches Prefix 1 (21 bits) and Prefix 2 (22 bits). Does NOT match Prefix 3. Longest prefix is /22. Forwarded to Interface 1.
* Datagram B (128.119.42.10): Binary octet 3 is 00101010. Matches Prefix 1 (21 bits), Prefix 2 (22 bits), and Prefix 3 (23 bits). Longest prefix is /23. Forwarded to Interface 2.

#### Solution 3

HOL blocking occurs at input ports when a packet at the front of an input queue is delayed waiting for a busy output port. This blocks all packets behind it, even if their target output ports are idle. It is prevented by using non-blocking switching fabrics (e.g., crossbars) combined with Virtual Output Queuing (VOQ) at input line cards.

#### Solution 4

* Original payload = $3200 - 20 = 3180 \text{ bytes}$.
* Max payload per fragment = $1000 - 20 = 980 \text{ bytes}$.
* Check 8-byte alignment: $980 \div 8 = 122.5$ (not divisible by 8!). The largest multiple of 8 $\le 980$ is $976 \text{ bytes}$ ($122 \times 8$).
* **Fragment 1:** Total Length = $976 + 20 = 996 \text{ bytes}$, Payload = 976, Offset = 0, MF = 1.
* **Fragment 2:** Total Length = $976 + 20 = 996 \text{ bytes}$, Payload = 976, Offset = $976 / 8 = 122$, MF = 1.
* **Fragment 3:** Total Length = $976 + 20 = 996 \text{ bytes}$, Payload = 976, Offset = $1952 / 8 = 244$, MF = 1.
* **Fragment 4:** Remaining payload = $3180 - (3 \times 976) = 252 \text{ bytes}$. Total Length = $252 + 20 = 272 \text{ bytes}$, Offset = $2928 / 8 = 366$, MF = 0.

| Fragment | Total Length | Header Length | Payload Size | Fragment Offset | MF Flag |
| --- | --- | --- | --- | --- | --- |
| Fragment 1 | 996 bytes | 20 bytes | 976 bytes | 0 | 1 |
| Fragment 2 | 996 bytes | 20 bytes | 976 bytes | 122 | 1 |
| Fragment 3 | 996 bytes | 20 bytes | 976 bytes | 244 | 1 |
| Fragment 4 | 272 bytes | 20 bytes | 252 bytes | 366 | 0 |

#### Solution 5

A NAT router can support up to $2^{16} - 1024 = 64{,}511$ (approximately 60,000+) simultaneous active TCP connections per public WAN IP. This limit is dictated by the 16-bit Source Port Number field in the TCP/UDP transport-layer header.

#### Solution 6

1. **DHCP Discover:** src = 0.0.0.0:68, dst = 255.255.255.255:67 (Broadcast)
2. **DHCP Offer:** src = ServerIP:67, dst = 255.255.255.255:68 (Broadcast)
3. **DHCP Request:** src = 0.0.0.0:68, dst = 255.255.255.255:67 (Broadcast)
4. **DHCP ACK:** src = ServerIP:67, dst = 255.255.255.255:68 (Broadcast)

#### Solution 7

1. **Header Checksum:** Removed because error checking is already performed at the link layer and transport layer. Eliminates per-hop checksum recalculation overhead.
2. **Identification / Flags / Fragment Offset:** Removed because intermediate IPv6 routers do not fragment packets. Eliminates router fragmentation overhead (Path MTU Discovery is enforced).
3. **Header Length (IHL):** Removed because IPv6 has a fixed 40-byte header length.

#### Solution 8

Using the WFQ throughput formula:

$$R_2 = R \cdot \frac{w_2}{w_1 + w_2 + w_3} = 30 \text{ Mbps} \cdot \frac{2}{3 + 2 + 1} = 30 \cdot \frac{2}{6} = \mathbf{10 \text{ Mbps}}$$

#### Solution 9

* **Definition:** Match on arbitrary criteria across Layer 2, Layer 3, and Layer 4 header fields, then execute one or more actions.
* **Matches:** Source MAC Address, IPv4 Destination Address, TCP Destination Port.
* **Actions:** Forward to specific port, Drop packet, Rewrite header field (e.g., Modify IP/Port).

#### Solution 10

* **Dual Stack:** Nodes run both IPv4 and IPv6 stacks. Used when nodes can directly communicate over IPv6-enabled links.
* **Tunneling:** Encapsulates IPv6 packets inside IPv4 packets. Strictly required when two IPv6-capable networks are separated by intermediate routers that only support IPv4.

## 7. Synthesis & Essay Discussion Questions

1. **Architectural Evolution:** How does the decoupling of the data plane and control plane in SDN alter traditional internet governance, router vendor monopolies, and rapid protocol deployment?
2. **Bufferbloat & AQM:** Explain how excessive buffering in edge routers leads to "Bufferbloat." Why do traditional loss-based TCP congestion control algorithms struggle in bufferbloated networks, and how do modern AQM algorithms (like CoDel or PIE) mitigate this issue?
3. **End-to-End Principle vs. Middlebox Reality:** Argue for or against the statement: "The proliferation of Network Address Translation (NAT) and middleboxes has irreversibly damaged the Internet architecture." Ground your arguments in security, scalability, and protocol innovation.
4. **IPv6 Transition Challenges:** Despite IPv4 address exhaustion occurring years ago, full IPv6 adoption remains incomplete. Analyze the economic, technical, and operational disincentives delaying complete global IPv6 migration.
5. **Generalized Forwarding vs. ASIC Customization:** How does OpenFlow's match-plus-action abstraction democratize packet switch hardware design, and what are the hardware tradeoffs involved in building TCAM tables capable of matching 40+ header fields at terabit speeds?

## 8. Comprehensive Glossary of Core Terms

* **Active Queue Management (AQM):** Proactive router buffer drop or marking policies (e.g., RED, CoDel) designed to manage queue lengths and signal congestion before buffer overflow occurs.
* **Best-Effort Service:** A network service model offering no guarantees regarding packet delivery, delay, bandwidth, or packet ordering.
* **Classless Inter-Domain Routing (CIDR):** An IP addressing scheme (a.b.c.d/x) that allows arbitrary prefix lengths, replacing classful network categories to improve address allocation efficiency.
* **Control Plane:** Network-wide control logic that computes routing paths and configures forwarding tables across routers.
* **Data Plane:** Per-router local hardware logic responsible for moving packets from input link interfaces to output link interfaces.
* **Dynamic Host Configuration Protocol (DHCP):** An application-layer protocol that automatically assigns IP addresses, subnet masks, default gateways, and DNS servers to network hosts.
* **Explicit Congestion Notification (ECN):** An IP/TCP header marking mechanism that allows routers to signal impending congestion to end hosts without dropping packets.
* **Head-of-Line (HOL) Blocking:** A queuing delay in input-queued switches where a packet at the front of the queue is blocked by a busy output port, delaying all packets queued behind it.
* **Longest Prefix Matching (LPM):** A forwarding lookup rule where a router selects the forwarding table entry whose address prefix matches the greatest number of leading bits of the destination IP address.
* **Maximum Transmission Unit (MTU):** The maximum payload size (in bytes) that a specific physical link-layer frame can carry.
* **Middlebox:** Any network device sitting on the data path that performs functions other than standard IP forwarding (e.g., NAT, firewalls, load balancers).
* **Network Address Translation (NAT):** A mechanism that maps private IP addresses and transport port numbers within a local network to a single public IP address facing the Internet.
* **OpenFlow:** An open standard communications protocol that enables SDN controllers to program the flow tables of data-plane switches.
* **Software-Defined Networking (SDN):** A networking architecture where the control plane is decoupled from the data plane and centralized in software controllers.
* **Ternary Content Addressable Memory (TCAM):** Specialized high-speed memory hardware that searches all stored entries in parallel in $O(1)$ constant time, supporting 0, 1, and \* (wildcard) bits.
* **Weighted Fair Queuing (WFQ):** A packet scheduling algorithm that distributes link transmission bandwidth among queued traffic classes in proportion to assigned numerical weights.