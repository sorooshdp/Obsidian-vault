# Comprehensive Study Guide: Computer Networking — Chapter 6 (The Link Layer and LANs)

## 1. Introduction to the Link Layer and Fundamental Services

The link layer occupies Layer 2 of the Internet protocol stack, operating immediately beneath the network layer (Layer 3) and directly above the physical layer (Layer 1). While the network layer is responsible for transferring datagrams end-to-end between a source host and a destination host across a heterogeneous mesh of interconnected networks, the link layer is responsible for transmitting network-layer datagrams across an individual, node-to-node communication link.

In networking terminology, any device that executes a network-layer or link-layer protocol is referred to as a node (including hosts, routers, Ethernet switches, and wireless access points). The communication channels that connect adjacent nodes along a communication path are defined as links. To move a datagram from a source host to a destination host, the datagram must be moved across each individual link along the end-to-end path.

### 1.1 Essential Services Provided by the Link Layer

While individual link-layer protocols offer varying feature sets depending on the physical medium and channel characteristics, the link layer can provide the following foundational services:

* **Framing:** Virtually all link-layer protocols encapsulate network-layer datagrams within a link-layer frame prior to transmission over the link. A frame consists of a data field, into which the network-layer datagram is inserted, and a set of header and trailer fields. The structure of the frame is defined by the specific link-layer protocol.
* **Link Access:** A Medium Access Control (MAC) protocol specifies the rules by which frame bits are transmitted onto the link. For point-to-point links (such as a dedicated fiber link connecting two routers), the MAC protocol is simple: the sender can transmit whenever the link is idle. However, when multiple nodes share a single broadcast link (such as a wireless LAN or a legacy coaxial cable network), the MAC protocol serves to coordinate frame transmissions and prevent collisions between competing nodes.
* **Reliable Delivery:** A link-layer protocol can provide reliable data transfer service over an individual link, ensuring that each network-layer datagram is transmitted across the link without error. Similar to transport-layer reliability protocols (such as TCP), link-layer reliable delivery is typically achieved through acknowledgments (ACKs) and retransmissions. Link-layer reliable delivery is primarily deployed on links that suffer from high bit-error rates, such as wireless channels, where correcting errors locally prevents expensive end-to-end retransmissions across the global Internet. Conversely, low-error wired links (such as fiber optic or twisted-pair copper) often omit link-layer reliability to minimize protocol overhead.
* **Flow Control:** Nodes on a link have finite frame-buffering capacity. If a sending node transmits frames faster than a receiving node can process and buffer them, frame overflow and packet loss occur. Link-layer flow control prevents a fast sending node from overwhelming a slower receiving node on the same physical link.
* **Error Detection:** Signal attenuation and electromagnetic noise on the physical medium can cause bit flips (turning a 0 into a 1, or vice versa). Because delivering corrupted datagrams to the network layer is wasteful, link-layer hardware includes mechanisms to detect bit errors. The transmitting node appends error-detection bits to the frame, and the receiving node executes an error check. If an error is detected, the receiver typically discards the corrupted frame.
* **Error Correction:** Beyond merely detecting bit errors, advanced link-layer error correction (Forward Error Correction, or FEC) enables the receiver to identify the exact locations of corrupted bits within a frame and correct them without requesting retransmission from the sender.
* **Full-Duplex and Half-Duplex Transmission:** In full-duplex mode, both nodes connected to a link can transmit and receive frames simultaneously. In half-duplex mode, a node can transmit and receive, but cannot perform both actions at the exact same instant.

### 1.2 Implementation Architecture of the Link Layer

Unlike upper-layer protocols (such as HTTP, DNS, TCP, and IP) that are implemented entirely in software within the operating system kernel or user space, the link layer is implemented at the boundary between hardware and software.

```text
+-------------------------------------------------------------+
|                        Host / Router                        |
|                                                             |
|  +-------------------------------------------------------+  |
|  | Network Layer (IP) & Upper Layers (Software / CPU)    |  |
|  +-------------------------------------------------------+  |
|                            | (Bus Interface / Interrupts)   |
|  +-------------------------------------------------------+  |
|  | Network Interface Card (NIC / Hardware Adapter)        |  |
|  |  +-------------------------------------------------+  |  |
|  |  | Link-Layer Controller (MAC / PHY Execution)     |  |  |
|  |  +-------------------------------------------------+  |  |
|  +-------------------------------------------------------+  |
+-------------------------------------------------------------+
                             | (Physical Link / Cable)
```

The primary physical host for link-layer implementation is the Network Interface Card (NIC), also known as the network adapter. At the heart of the NIC is a specialized link-layer controller—a dedicated hardware chip that implements MAC protocols, frame assembly/parsing, error detection, and physical-layer signal modulation/demodulation.

The hardware-software division operates as follows:

1. **Sending Side:** The operating system prepares an IP datagram in host memory and passes it to the NIC via an internal system bus (such as PCIe). The link-layer controller receives the datagram, encapsulates it in a link-layer frame header and trailer (including error-detection bits), and transmits the frame onto the physical medium as electrical, optical, or radio signals.
2. **Receiving Side:** The receiving NIC's controller detects physical signals on the medium, reconstructs the bit stream, isolates the frame, performs error checking, and strips away the link-layer header and trailer. If no errors are detected, the controller triggers a hardware interrupt to notify the host CPU and transfers the extracted IP datagram into host memory for network-layer processing.

## 2. Error-Detection and Error-Correction Techniques

Bit errors occur when physical noise, electromagnetic interference, or signal attenuation corrupts transmitted signals. To prevent corrupted data from propagating up the protocol stack, link-layer frames append Error-Detection and -Correction (EDC) bits to the data payload D.

```text
Sender:    Data payload (D) + Error-Detection Code (EDC)  ---> Transmitted Frame
Receiver:  Received (D')   + Received (EDC')              ---> Error Check Calculation
```

The sender computes EDC based on payload D. The receiver receives D' and EDC' (which may differ from D and EDC due to channel noise) and verifies whether the mathematical relationship between D' and EDC' holds. If it fails, an error is detected. However, error detection is not 100% foolproof; sophisticated bit corruption patterns can occasionally pass error checks undetected. Increasing EDC overhead reduces the probability of undetected errors.

### 2.1 Parity Checks

Parity checking is the simplest form of error detection.

#### Single-Bit Parity

In a single-bit even parity scheme, the sender appends a single parity bit to $d$ data bits such that the total number of $1\text{s}$ in the resulting $d+1$ bits is even. In an odd parity scheme, the parity bit is chosen so that the total number of $1\text{s}$ is odd.

* Detection capability: Single-bit parity detects any single bit flip, or any odd number of bit flips.
* Limitation: If an even number of bits flip (e.g., two bits flip simultaneously), the total parity remains unchanged, and the error goes completely undetected.

#### Two-Dimensional Parity

To enhance detection and enable error correction, $d$ bits are arranged into a matrix of $i$ rows and $j$ columns. A parity bit is computed for each row and for each column.

```text
Data Bits:                 Row Parity:
d_(1,1)  d_(1,2) ... d_(1,j) |  d_(1,j+1)
d_(2,1)  d_(2,2) ... d_(2,j) |  d_(2,j+1)
  ...      ...   ...   ...   |     ...
d_(i,1)  d_(i,2) ... d_(i,j) |  d_(i,j+1)
-----------------------------+-----------
Column Parity:
d_(i+1,1) d_(i+1,2) ... d_(i+1,j) | d_(i+1,j+1)
```

* Single-Bit Correction: When a single bit flips anywhere in the data or parity matrix, both its row parity and column parity will indicate an error. The intersection of the error row and error column pinpoints the exact bit location, allowing the receiver to correct the bit flip (Forward Error Correction, FEC).
* Multi-Bit Detection: Two-dimensional parity can detect (though not always correct) any pair of bit errors in a frame.

### 2.2 Checksumming Methods

Checksumming protocols treat data as a sequence of $k$-bit integers (typically 16-bit integers). The integers are summed using 1s complement arithmetic, and the 1s complement of the sum forms the checksum.

* The Internet transport layer (UDP and TCP) uses 16-bit checksums computed in software.
* While simple to implement in software, checksums provide relatively weak protection against bit errors compared to hardware-based techniques like CRC, making them less suitable for link-layer hardware execution.

### 2.3 Cyclic Redundancy Check (CRC)

Modern link-layer adapters implement Cyclic Redundancy Check (CRC) codes (also known as polynomial codes). CRC views bit strings as polynomials with binary coefficients (0 and 1), evaluating them using modulo-2 arithmetic (where addition and subtraction are identical and equivalent to the bitwise XOR operation, with no carries or borrows).

#### Mathematical Formulation

1. Let D be a $d$-bit data payload.
2. Let G be a predefined $(r+1)$-bit pattern agreed upon by sender and receiver, known as the generator polynomial. The most significant bit of G must be 1.
3. The sender wants to append $r$ CRC bits (denoted as $R$) to D such that the resulting $(d+r)$-bit pattern is exactly divisible by G using modulo-2 division.

```text
Positioning:  [  d bits of Data (D)  ][  r bits of CRC (R)  ]
Mathematical Value:  D * 2^r  XOR  R
```

The mathematical condition required at the receiver is: $\text{Remainder of }\frac{D \cdot 2^r \text{ XOR } R}{G} = 0$

#### Step-by-Step Derivation of R

Dividing $D \cdot 2^r$ by G using modulo-2 division yields a quotient n and a remainder R: $D \cdot 2^r = n \cdot G \text{ XOR } R$

Transposing R via modulo-2 arithmetic yields: $R = \text{remainder}\left( \frac{D \cdot 2^r}{G} \right)$

#### Worked Numerical Example

Suppose the data payload $D = 101110$ ($d=6$ bits) and the generator $G = 1001$ ($r+1 = 4$ bits, so $r=3$).

1. Shift D left by $r=3$ bits: $D \cdot 2^3 = 101110000$.
2. Divide 101110000 by $G = 1001$ using XOR modulo-2 long division:

```text
          101011  (Quotient)
       +---------
1001   | 101110000
         1001
         ----
          0101
          0000
          ----
           1010
           1001
           ----
            0110
            0000
            ----
             1100
             1001
             ----
              1010
              1001
              ----
               011  (Remainder R = 011)
```

1. The sender appends $R = 011$ to D, transmitting the 9-bit frame 101110011.
2. Receiver Verification: The receiver divides 101110011 by $G = 1001$. If the modulo-2 remainder is 000, the frame is accepted as uncorrupted; otherwise, an error is flagged.

CRC standards (such as CRC-12, CRC-16, CRC-32) use standardized generators. CRC-32 (used in Ethernet and Wi-Fi) utilizes a 33-bit generator pattern capable of detecting burst errors up to 32 bits long and all odd numbers of bit errors.

## 3. Multiple Access Links and Protocols

Link-layer channels fall into two primary structural categories:

1. Point-to-Point Links: Consisting of a single sender and a single receiver connected by a dedicated link (e.g., PPP or HDLC links).
2. Broadcast Links: Connecting multiple nodes to a shared physical channel. When any node transmits a frame, the signal propagates across the medium, and every other node connected to the channel receives a copy.

When multiple nodes transmit simultaneously on a broadcast channel, their signals interfere at the receivers, resulting in a collision. During a collision, the overlapping signals distort one another, rendering all colliding frames unreadable and wasting channel capacity.

To manage access to the shared channel, networks deploy a Multiple Access Protocol.

### 3.1 Ideal Criteria for a Multiple Access Protocol

For a shared broadcast channel of transmission rate $R\text{ bps}$, an ideal MAC protocol should possess four properties:

1. Full Throughput: When only one node has data to transmit, that node achieves a throughput of $R\text{ bps}$.
2. Fair Equal Sharing: When M nodes have data to transmit, each node achieves an average throughput of $R/M\text{ bps}$.
3. Fully Decentralized: There is no central master node to orchestrate transmissions (avoiding a single point of failure and clock synchronization bottlenecks).
4. Simplicity: The protocol is inexpensive and simple to implement in hardware.

Multiple access protocols are classified into three primary categories: Channel Partitioning, Random Access, and Taking-Turns.

### 3.2 Category 1: Channel Partitioning Protocols

Channel partitioning divides the total channel capacity into smaller, independent sub-channels allocated to specific nodes.

* Time-Division Multiplexing (TDM): Time is divided into frame intervals, and each frame interval is subdivided into N fixed-duration time slots. Each of the N nodes is assigned a dedicated slot per frame.
  * Pros: Completely eliminates collisions; perfectly fair ($R/N$ throughput per node).
  * Cons: Inefficient. If a node has no data to send, its assigned slot sits completely idle, limiting an active node's maximum rate to $R/N$ even when no other node is transmitting.
* Frequency-Division Multiplexing (FDM): The total channel bandwidth $R\text{ bps}$ across frequency spectrum Z is divided into N smaller frequency bands, each with bandwidth $Z/N$.
  * Pros: Eliminates collisions; fair distribution.
  * Cons: Unused frequency bands remain idle, limiting a single active node to $1/N$-th of the total spectrum.
* Code Division Multiple Access (CDMA): Assigns a unique mathematical code to each node, allowing nodes to transmit simultaneously across the entire frequency spectrum while receivers disentangle overlapping signals.

### 3.3 Category 2: Random Access Protocols

In a random access protocol, active nodes transmit at the full rate of the channel ($R\text{ bps}$) without prior reservation. When a collision occurs, colliding nodes wait a random delay before retransmitting, allowing one node to transmit successfully.

#### 1. Slotted ALOHA

* Assumptions:
  * All frames consist of $L$ bits.
  * Time is partitioned into discrete slots of size $L/R\text{ seconds}$.
  * Nodes transmit only at the beginning of a slot.
  * Nodes are synchronized so every node knows when slots begin.
* Operation:
  * When a node has a new frame, it transmits at the start of the next slot.
  * If no collision occurs, the frame is successfully transmitted.
  * If a collision occurs, the node detects it before the slot ends and retransmits the frame in each subsequent slot with probability p until success occurs.
* Efficiency Derivation:
  * Suppose N active nodes compete in slotted ALOHA with transmission probability p.
  * The probability that a specific node succeeds in a slot is $p(1-p)^{N-1}$.
  * The probability that any node succeeds in a slot is $S(p) = N p (1-p)^{N-1}$.
  * Maximizing S(p) with respect to p yields optimal probability $p^* = 1/N$.
  * Substituting $p^* = 1/N$: $S(p^*) = \left(1 - \frac{1}{N}\right)^{N-1}$.
  * Taking the limit as $N \to \infty$: $\lim_{N \to \infty} \left(1 - \frac{1}{N}\right)^{N-1} = \frac{1}{e} \approx 0.368$
  * Maximum Slotted ALOHA Efficiency: $36.8\%$. (63.2% of channel slots are wasted due to collisions or idle gaps).

#### 2. Pure (Unslotted) ALOHA

Pure ALOHA eliminates slot synchronization. When a frame arrives from the network layer, the node immediately transmits it onto the channel.

```text
Node i Frame:                  |==== Frame i ====|
Interfering Window:     |-- Frame j --|     |-- Frame k --|
Time Axis:           t_0 - t_frame   t_0   t_0 + t_frame
```

* Vulnerability Window: A frame starting at time $t_0$ collides if any other node transmits between $t_0 - t_{\text{frame}}$ and $t_0 + t_{\text{frame}}$. The vulnerability window is twice the frame transmission time ($2 t_{\text{frame}}$).
* Efficiency Derivation:
  * Probability a node successfully transmits a frame is $p(1-p)^{2(N-1)}$.
  * Taking the limit as $N \to \infty$ yields maximum efficiency:

    $$\text{Maximum Pure ALOHA Efficiency} = \frac{1}{2e} \approx 0.184$$
  * Maximum Pure ALOHA Efficiency: $18.4\%$.

#### 3. Carrier Sense Multiple Access (CSMA)

To overcome ALOHA's high collision rate, CSMA introduces carrier sensing ("listen before speaking"):

* Rule: A node listens to the channel prior to transmitting. If the channel is sensed busy, the node defers transmission; if idle, the node transmits.

Why collisions still occur in CSMA: Channel propagation delay! It takes finite time for signals to travel down the physical wire.

```text
Node A: Transmits at t = 0 -------------------> [Signal propagating...]
Node B: Senses channel at t = t_prop - delta ---> Senses IDLE! ---> Transmits! ---> COLLISION!
```

If Node A transmits at $t=0$, Node B (located distance $d$ away) will not detect A's signal until $t = d/s$ (propagation delay $t_{\text{prop}}$). If B senses the channel during this window, it will perceive the link as idle and begin transmitting, causing a collision.

#### 4. CSMA with Collision Detection (CSMA/CD)

CSMA/CD adds collision detection ("listen while speaking"). A node monitors the medium while transmitting. If it detects interfering signal energy, it aborts transmission immediately, preventing the unnecessary transmission of damaged frames.

```text
          +--------------------------------------------------+
          | Adapter receives datagram from Network Layer and |
          | encapsulates it into an Ethernet frame in buffer |
          +--------------------------------------------------+
                                   |
                                   v
                   +-------------------------------+
                   | Is channel sensed IDLE?       |
                   +-------------------------------+
                      /                         \
            YES      /                           \ NO
                    v                             v
   +-------------------------------+   +-----------------------------+
   | Start transmitting frame bits |   | Wait until channel is idle, |
   +-------------------------------+   | then begin transmission     |
   +-------------------------------+   +-----------------------------+
                                   |
                                   v
             +-------------------------------------------+
             | Transmitting... Is a Collision Detected?  |
             +-------------------------------------------+
                /                                     \
      NO       /                                       \ YES
              v                                         v
   +---------------------+             +----------------------------------+
   | Successful Frame    |             | Abort transmission immediately.  |
   | Transmission!       |             | Send 48-bit Jam Signal.          |
   +---------------------+             +----------------------------------+
                                                        |
                                                        v
                                       +----------------------------------+
                                       | Increment Collision Counter n    |
                                       | Choose K randomly from           |
                                       | {0, 1, 2, ..., 2^min(n,10) - 1}  |
                                       | Wait K * 512 bit times           |
                                       +----------------------------------+
                                                        |
                                                        v
                                       +----------------------------------+
                                       | Return to Channel Sensing Step   |
                                       +----------------------------------+
```

* Binary Exponential Backoff Algorithm:
  * After the n-th consecutive collision, the adapter chooses integer K uniformly at random from the set $\{0, 1, 2, \dots, 2^{\min(n, 10)} - 1\}$.
  * The adapter waits $K \cdot 512\text{ bit times}$ (for 10 Mbps Ethernet, 1 bit time = $0.1\,\mu\text{s}$; 512 bit times = $51.2\,\mu\text{s}$) before attempting to retransmit.
  * After 16 consecutive collisions, the adapter gives up and reports a transmission failure up the stack.
* CSMA/CD Efficiency:

  $$\text{Efficiency} = \frac{1}{1 + 5 \cdot \frac{d_{\text{prop}}}{d_{\text{trans}}}}$$

  Where $d_{\text{prop}}$ is the maximum propagation delay between any pair of nodes and $d_{\text{trans}}$ is the time to transmit a maximum-size frame. As $d_{\text{prop}} \to 0$ or $d_{\text{trans}} \to \infty$, efficiency approaches $100\%$.

### 3.4 Category 3: Taking-Turns Protocols

Taking-turns protocols combine the high-load efficiency of TDM with the single-node full throughput of random access.

1. **Polling Protocol:** Designates a master node that queries slave nodes in round-robin sequence, granting each permission to transmit up to a maximum number of frames.
  * Cons: Polling overhead delay; master node represents a single point of failure.
2. **Token-Passing Protocol:** Nodes form a logical ring, passing a specialized frame called a token sequentially. A node may transmit data only while holding the token.
  * Cons: Token propagation overhead; a failed node or lost token halts channel operation until complex recovery mechanisms restore the token.

### 3.5 Case Study: Cable Internet Access (DOCSIS)

The Data Over Cable Service Interface Specification (DOCSIS) protocol governs Hybrid Fiber-Coaxial (HFC) cable access networks, combining all three MAC protocol classes:

* FDM: Downstream and upstream channels are divided into dedicated frequency bands.
* TDM / Grant Allocation: The Cable Modem Termination System (CMTS) located at the headend allocates upstream time slots to individual cable modems.
* Random Access: Modems request upstream bandwidth by transmitting Mini-Slot Request frames during designated contention slots using slotted ALOHA with binary exponential backoff.

## 4. Switched Local Area Networks (LANs)

Switched LANs deploy link-layer switches to interconnect hosts, enterprise servers, and routers within an institution.

### 4.1 Link-Layer Addressing and ARP

Unlike IP addresses, which are 32-bit (IPv4) or 128-bit (IPv6) hierarchical addresses structured based on network subnets, link-layer addresses are flat and bound to physical hardware.

#### MAC Addresses

* A MAC Address (Media Access Control address, or LAN address/hardware address) is a 48-bit binary address expressed as six pairs of hexadecimal digits (e.g., 88-B2-2F-54-1A-0F).
* Flat Structure: MAC addresses do not change when a device moves to a different network. They are burned into NIC non-volatile memory during manufacturing.
* IEEE Allocation: The IEEE manages MAC address allocation by assigning a 24-bit Organizationally Unique Identifier (OUI) prefix to each hardware manufacturer.
* Broadcast Address: The link-layer broadcast address consists of 48 ones (FF-FF-FF-FF-FF-FF). Frames sent to this address are processed by all NICs on the LAN.

#### Address Resolution Protocol (ARP)

Because IP datagrams must be encapsulated inside link-layer frames to cross a physical link, nodes must translate network-layer IP addresses into link-layer MAC addresses. This function is performed by the Address Resolution Protocol (ARP).

ARP maintains an ARP Table in host RAM, mapping IP addresses to MAC addresses along with a Time-to-Live (TTL) expiration value (typically 20 minutes):

| IP Address | MAC Address | TTL |
| --- | --- | --- |
| 222.222.222.221 | 88-B2-2F-54-1A-0F | 13:14 min |
| 222.222.222.222 | 50-62-B9-11-22-33 | 19:50 min |

#### Intra-Subnet ARP Operation

Suppose Host A (222.222.222.220) wants to send an IP datagram to Host B (222.222.222.222) on the same IP subnet, but Host A's ARP table lacks Host B's MAC address:

1. Host A constructs an ARP Query Packet containing target IP 222.222.222.222 and ARP sender hardware/IP info.
2. Host A encapsulates the ARP query in an Ethernet frame with destination MAC FF-FF-FF-FF-FF-FF (Broadcast) and sends it onto the LAN.
3. Every adapter connected to the switch receives and parses the broadcast frame, passing the ARP query to its ARP module.
4. Host B recognizes its own IP address in the query and constructs an ARP Response Packet containing its MAC address (50-62-B9-11-22-33).
5. Host B sends the ARP response as a unicast frame directly to Host A's MAC address.
6. Host A extracts Host B's MAC address, updates its ARP table, and transmits the buffered IP datagram inside a unicast Ethernet frame to Host B.

#### Inter-Subnet ARP Operation

If Host A (222.222.222.220) sends a datagram to Host C (111.111.111.088) located on a different subnet, Host A does not issue an ARP query for Host C. Instead, Host A's IP routing logic selects the first-hop default gateway router interface (222.222.222.254). Host A uses ARP to resolve the MAC address of the gateway router interface, encapsulates the datagram destined for Host C in a frame addressed to the router's MAC, and transmits it. The router decapsulates the datagram, consults its IP routing table, and forwards the datagram across the next subnet using a new link-layer frame with updated MAC addresses.

### 4.2 Ethernet

Ethernet is the dominant wired LAN technology.

#### Ethernet Frame Structure

Ethernet frame fields are organized as follows:

```text
+----------+---------------+------------+-----------+---------------+---------+
| Preamble | Destination   | Source     | Type/     | Data Payload  | CRC     |
| (8 Bytes)| MAC (6 Bytes) | MAC (6B)   | Length(2B)| (46-1500 B)   | (4 Bytes|
+----------+---------------+------------+-----------+---------------+---------+
```

* Preamble (8 bytes): The first 7 bytes contain bit pattern 10101010, and the 8th byte contains 10101011 (Start Frame Delimiter, SFD). Used to synchronize clock frequencies between sender and receiver adapters.
* Destination Address (6 bytes): MAC address of destination adapter (or broadcast FF-FF-FF-FF-FF-FF).
* Source Address (6 bytes): MAC address of transmitting adapter.
* Type/Length Field (2 bytes): Multiplexing tag specifying the upper-layer protocol bound to the payload (e.g., 0x0800 for IPv4, 0x0806 for ARP, 0x86DD for IPv6).
* Data Field (46 to 1500 bytes): Payload containing the IP datagram. The Minimum Transmission Unit (MTU) of Ethernet is 1500 bytes. If a payload is under 46 bytes, it is padded to reach 46 bytes.
* Cyclic Redundancy Check (CRC) (4 bytes): Contains the 32-bit CRC calculation to detect bit errors.

#### Connectionless and Unreliable Service

Ethernet is connectionless (no handshaking prior to frame transmission) and unreliable (receiving adapters discard corrupted frames without sending ACKs or NACKs). Lost data recovery relies entirely on upper-layer protocols like TCP.

#### Physical Topology Evolution

* Coaxial Bus Topology (1980s–1990s): Nodes connected to a shared coaxial cable bus forming a single physical collision domain.
* Hub-Based Star Topology (1990s): Nodes connected to a central physical-layer hub. Hubs re-broadcast arriving bits onto all other ports, maintaining a shared collision domain.
* Switch-Based Star Topology (2000s–Present): Hubs replaced by Layer-2 Switches. Each link becomes an isolated full-duplex domain, eliminating collisions entirely.

### 4.3 Link-Layer Switches

A link-layer switch is a store-and-forward packet switch operating at Layer 2. Switches are transparent to hosts on the LAN—hosts address frames to target end-systems without addressing the switch.

#### Filtering and Forwarding

* Filtering: The switch operation that determines whether a frame should be forwarded to an interface or dropped.
* Forwarding: The switch operation that moves a frame to the appropriate output interface buffer.

Filtering and forwarding rely on a Switch Table containing:

1. A host's MAC address.
2. The switch interface leading to that host.
3. An entry creation timestamp (used for purging expired entries).

```text
Arriving Frame: [Src MAC: AA-AA-AA-AA-AA-AA] -> [Dst MAC: DD-DD-DD-DD-DD-DD] on Interface x
                                     |
                                     v
                   +------------------------------------+
                   | Check Switch Table for Dst MAC     |
                   +------------------------------------+
                                 /   |   \
                                /    |    \
            NO ENTRY FOUND     /     |     \  ENTRY MATCHES
                              /      |      \ INTERFACE x
                             /       |       \
                            v        |        v
          +-------------------+      |      +------------------+
          | Broadcast (Flood) |      |      | Drop Frame       |
          | to all ports      |      |      | (Filtering)      |
          | except port x     |      |      +------------------+
          +-------------------+      |
                                     v
                           ENTRY MATCHES INTERFACE y (y != x)
                                     |
                                     v
                           +-------------------+
                           | Forward frame to  |
                           | output port y     |
                           +-------------------+
```

#### Self-Learning Switches

Switches build their tables automatically using a self-learning algorithm:

1. The switch table starts empty.
2. For every frame received on interface x, the switch extracts the Source MAC Address and logs an entry mapping that MAC address to interface x, along with the current timestamp.
3. If no frame with that source MAC is received within an aging time (e.g., 60 seconds), the entry is purged from the table.

#### Switches vs. Routers: A Comprehensive Comparison

| Feature | Link-Layer Switch | Network-Layer Router |
| --- | --- | --- |
| Operating Layer | Layer 2 (Link Layer) | Layer 3 (Network Layer) |
| Forwarding Identifier | 48-bit MAC Address (Flat) | 32-bit / 128-bit IP Address (Hierarchical) |
| Configuration | Plug-and-Play (Self-learning tables) | Requires IP subnet and routing configuration |
| Forwarding Mechanics | Self-learning, Flooding on miss | Routing protocols (OSPF, BGP) / Longest Prefix Match |
| Active Topology | Restricted to a Spanning Tree (STP) | Arbitrary / Multi-path topologies permitted |
| Isolation & Security | Vulnerable to Layer-2 Broadcast Storms | Isolates broadcast domains; firewall filtering |

### 4.4 Virtual Local Area Networks (VLANs)

In institutional networks, physical switch boundaries do not always align with organizational structures. A single physical switch topology faces three challenges:

1. Lack of Traffic Isolation: Broadcast traffic (e.g., ARP queries) propagates across the entire institutional network.
2. Inefficient Switch Utilization: Small groups spread across buildings require dedicated switches.
3. User Mobility Limits: Moving an employee to a new office requires physical recabling.

These issues are resolved using Virtual Local Area Networks (VLANs), which partition a physical switch's ports into logical broadcast domains.

```text
Physical Switch:
  Ports [2, 3, 4, 5, 6, 7, 8]  --> Assigned to EE VLAN (VLAN 10)
  Ports [9, 10, 11, 12, 13, 14] --> Assigned to CS VLAN (VLAN 20)
```

Broadcast traffic originating on Port 2 (EE VLAN) is forwarded only to ports belonging to VLAN 10, isolating it from the CS VLAN. Communication between the EE and CS VLANs requires a Layer-3 device (such as an internal or external router).

#### VLAN Trunking and IEEE 802.1Q

When a VLAN spans multiple physical switches, switches are connected via Trunk Ports. A trunk port belongs to all VLANs, forwarding frames between switches.

To preserve VLAN identity across trunk links, the IEEE 802.1Q standard inserts a 4-byte VLAN tag into the standard Ethernet frame header:

```text
Standard Ethernet Frame:
+----------+----------+----------+------+--------------+-----+
| Preamble | Dst MAC  | Src MAC  | Type | Data Payload | CRC |
+----------+----------+----------+------+--------------+-----+

802.1Q Tagged Frame:
+----------+----------+----------+------+----------+------+--------------+------+
| Preamble | Dst MAC  | Src MAC  | TPID | Priority | VLAN | Data Payload | CRC' |
|          |          |          |0x8100| & CFI    | ID   |              |      |
+----------+----------+----------+------+----------+------+--------------+------+
                                 |<--- 802.1Q Tag (4B) --->|
```

* Tag Protocol Identifier (TPID): 2-byte field set to 0x8100, identifying the frame as 802.1Q tagged.
* Priority / DEI Bits: 3 bits for Class of Service (CoS) priority, 1 bit for Drop Eligibility Indicator.
* VLAN ID (VID): 12-bit field specifying the target VLAN (supporting up to $2^{12} = 4096$ distinct VLANs).

## 5. Link Virtualization: Multiprotocol Label Switching (MPLS)

Multiprotocol Label Switching (MPLS) is a Layer-2.5 protocol that augments IP networks by adopting fixed-length labels to forward packets.

```text
Link-Layer Header | MPLS Header | IP Header | TCP/UDP Header | Payload
                  |<-- 4B -->|
```

### MPLS Header Format

An MPLS header (4 bytes) is inserted between the Layer-2 (Ethernet) header and Layer-3 (IP) header:

```text
+--------------------------+---------------+----+----------+
| Label                    | Exp / TC      | S  | TTL      |
| (20 Bits)                | (3 Bits)      | (1)| (8 Bits) |
+--------------------------+---------------+----+----------+
```

* Label (20 bits): The virtual circuit identifier used for lookup.
* Experimental / Traffic Class (3 bits): Quality of Service (QoS) signaling.
* Stack Bit S (1 bit): Set to 1 to indicate the final label in a stacked label sequence.
* Time-to-Live (TTL) (8 bits): Decremented at each hop to prevent indefinite looping.

### Label-Switched Routers (LSRs)

An MPLS-capable router is called a Label-Switched Router (LSR). LSRs forward packets by indexing their tables with the MPLS label, ignoring the destination IP address:

```text
[In Interface: 0, In Label: 10] ---> [Action: Swap Label to 6, Out Interface: 1]
```

### Key Advantages of MPLS

1. High-Speed Forwarding: Fixed-length label indexing is faster than IP longest-prefix matching.
2. Traffic Engineering: MPLS can route packets along paths that override standard IP least-cost destination routes.
3. Virtual Private Networks (VPNs): Enables ISPs to build isolated corporate VPNs over a shared core infrastructure.

## 6. Data Center Networking

Modern cloud data centers house tens to hundreds of thousands of hosts (blade servers) interconnected via specialized data center network fabrics.

### 6.1 Data Center Network Architectures

```text
                   +------------------------+
                   |     Border Routers     |
                   +------------------------+
                               |
                   +------------------------+
                   |     Access Routers     |
                   +------------------------+
                               |
                   +------------------------+
                   |    Tier-1 Switches     |  (Core Layer)
                   +------------------------+
                               |
                   +------------------------+
                   |    Tier-2 Switches     |  (Aggregation Layer)
                   +------------------------+
                               |
                   +------------------------+
                   | Top-of-Rack (ToR) Sw.  |  (Access Layer)
                   +------------------------+
                    /          |           \
               [Blade 1]   [Blade 2] ... [Blade 30]  (Server Rack)
```

1. Blade Servers: Commodity hosts packed into racks (typically 20–40 blades per rack).
2. Top-of-Rack (ToR) Switch: A dedicated Layer-2 switch mounted at the top of each rack, directly connecting to each blade server at 10/40/100 Gbps Ethernet rates.
3. Hierarchical Switch Fabric: ToR switches connect up to Tier-2 (Aggregation) switches, which in turn connect to Tier-1 (Core) switches, access routers, and external Border Routers.

### 6.2 Load Balancing

External client requests are directed to a publicly visible Virtual IP (VIP) address. Inside the data center, Load Balancers (Layer-4 switches) distribute arriving requests across a pool of backend servers based on application load. Load balancers also act as NAT devices, translating VIP addresses to internal server IP addresses.

### 6.3 Bottlenecks, Clos Topologies, and ECMP

Hierarchical networks can suffer from bandwidth bottlenecks at upper-tier switches. To eliminate oversubscription and provide full bisection bandwidth, modern data centers deploy Clos (Fat-Tree) topologies, where every ToR switch connects to multiple Tier-2 switches via redundant paths.

Traffic load is distributed across these redundant paths using Equal-Cost Multi-Path (ECMP) routing, which hashes packet header fields (5-tuple) to split traffic flows evenly across available links.

## 7. Retrospective: A Day in the Life of a Web Page Request

To synthesize the interaction of protocols across all five layers, let us trace the complete step-by-step sequence of events when a user ("Bob") connects his laptop to an Ethernet switch and requests www.google.com.

Bob's Laptop === Ethernet Switch === School Router/DHCP === Comcast ISP/DNS === Google Server

### Phase 1: Bootstrapping via DHCP, UDP, IP, and Ethernet

1. The OS on Bob's laptop creates a DHCP Request Message (DHCP Discover) with destination port 67 (DHCP Server) and source port 68.
2. The DHCP message is encapsulated in a UDP segment, then into an IPv4 datagram with source IP 0.0.0.0 and destination IP 255.255.255.255 (Limited Broadcast).
3. The IP datagram is encapsulated in an Ethernet frame with destination MAC FF-FF-FF-FF-FF-FF and sent onto the link.
4. The local Ethernet switch receives the frame, logs Bob's MAC address in its switch table (Self-Learning), and broadcasts the frame onto all ports.
5. The gateway router (acting as a DHCP server) receives the frame, extracts the UDP datagram, and processes the DHCP request.
6. The DHCP server allocates IP address 68.85.2.101, default gateway 68.85.2.1, DNS server 68.87.71.226, and subnet mask 68.85.2.0/24.
7. The server encapsulates this info into a DHCP ACK frame, which is sent back through the switch to Bob's laptop, completing IP configuration.

### Phase 2: DNS Address Resolution & ARP

1. Bob types www.google.com into his browser. The browser initiates a TCP socket connection, requiring the IP address of www.google.com.
2. The OS generates a DNS Query Message requesting the A record for www.google.com.
3. The DNS query is encapsulated in a UDP segment addressed to DNS server IP 68.87.71.226 on port 53.
4. To send the packet to the DNS server outside the local subnet, Bob's laptop must forward it to the default gateway (68.85.2.1). Because its ARP cache is empty, the laptop generates an ARP Query Packet requesting the MAC address of 68.85.2.1.
5. The ARP query is sent in a broadcast Ethernet frame (FF-FF-FF-FF-FF-FF). The gateway router receives it and replies with an ARP Response Packet containing its MAC address (00:22:6B:45:1F:1B).
6. Bob's laptop receives the ARP response, updates its ARP table, and transmits the buffered DNS query frame directly to the gateway router's MAC address.

### Phase 3: Routing the DNS Request

1. The school router receives the frame, extracts the IP datagram, consults its routing table, and forwards the datagram across the Comcast ISP network.
2. The DNS query arrives at Comcast's DNS server (68.87.71.226), which looks up www.google.com in its database and returns a DNS Reply Message containing IP 64.233.169.105.
3. The DNS response is routed back to Bob's laptop, which extracts Google's IP address.

### Phase 4: TCP Connection Establishment

1. Bob's browser creates a TCP socket to IP 64.233.169.105 on port 80.
2. The OS generates a TCP SYN Segment (with SYN=1 and a initial sequence number).
3. The TCP SYN is encapsulated in an IP datagram and an Ethernet frame (addressed to the default gateway MAC) and sent into the network.
4. The packet is routed through the Internet to Google's HTTP server, which responds with a TCP SYN-ACK Segment.
5. Bob's laptop receives the SYN-ACK, transitions the socket to the ESTABLISHED state, and returns the final ACK.

### Phase 5: HTTP Request and Page Rendering

1. The browser writes an HTTP GET / message into the TCP socket.
2. The HTTP GET is encapsulated in a TCP segment, IP datagram, and Ethernet frame, and sent to Google's server.
3. Google's web server reads the HTTP request, retrieves the HTML content, generates an HTTP 200 OK Response, and transmits it back over the TCP connection.
4. Bob's browser receives the HTTP response, extracts the HTML text, and renders the web page!

## 8. Practice Examination

### Questions

#### Problem 1 (CRC Calculation)

Suppose the 6-bit data payload $D = 101011$ and the generator polynomial $G = 1011$ ($r=3$). Compute the 3-bit CRC remainder R using modulo-2 long division. What is the final 9-bit frame transmitted by the sender?

#### Problem 2 (Two-Dimensional Parity)

Suppose the following 16 bits of data are arranged into a 4x4 matrix using two-dimensional even parity:

```text
1 0 1 1
0 1 1 0
1 1 0 1
0 0 1 0
```

Compute the 4 row parity bits and 4 column parity bits. If the bit at Row 2, Column 3 flips from 1 to 0 during transmission, demonstrate how the receiver detects and corrects the bit flip.

#### Problem 3 (Slotted ALOHA Efficiency)

Show that the efficiency of Slotted ALOHA reaches its maximum when the transmission probability $p = 1/N$. Derive the limiting efficiency value as $N \to \infty$.

#### Problem 4 (CSMA/CD Backoff Calculation)

In CSMA/CD, suppose a host experiences its 3rd consecutive collision on a 10 Mbps Ethernet link.

1. What is the set of possible values for K?
2. What is the probability that the host chooses K = 2?
3. If K = 2, how long does the adapter wait in microseconds before attempting retransmission?

#### Problem 5 (Switch Learning Trace)

Consider a self-learning switch connected to 4 hosts (A, B, C, D) on ports 1, 2, 3, and 4, respectively. The switch table is initially empty. Show the state of the switch table and state whether the frame is Flooded or Forwarded Unicast after each event:

1. Host A sends a frame to B.
2. Host B replies with a frame to A.
3. Host C sends a frame to B.
4. Host A sends a frame to B.

#### Problem 6 (Switches vs. Routers)

State three distinct architectural or operational advantages of using Layer-2 switches over Layer-3 routers, and three advantages of using Layer-3 routers over Layer-2 switches.

#### Problem 7 (ARP vs. DNS)

Compare and contrast ARP and DNS across the following dimensions:

1. Protocol layer.
2. Address resolution domain (Local vs. Global).
3. Query transport mechanism (Broadcast vs. Unicast/UDP).

#### Problem 8 (802.1Q Tag Structure)

Explain why an 802.1Q tag must be inserted into an Ethernet frame when crossing a VLAN trunk link. Where in the Ethernet frame is the tag placed, and how many distinct VLAN IDs can it support?

#### Problem 9 (Datacenter ECMP Routing)

In a Fat-Tree datacenter topology, describe how Equal-Cost Multi-Path (ECMP) load balancing prevents network congestion between server racks. What packet header fields are hashed to perform ECMP?

#### Problem 10 (Day-in-the-Life Packet Inspection)

During the "Day in the Life of a Web Page Request" sequence, identify the source and destination IP addresses, as well as the source and destination MAC addresses, for the Ethernet frame carrying the DNS Query as it travels across:

1. The link from Bob's laptop to the local school router.
2. The ISP link from the school router to Comcast's DNS server.

### Answer Key

#### Solution 1

1. Shift D left by $r=3$ bits: $D \cdot 2^3 = 101011000$.
2. Modulo-2 division of 101011000 by 1011:

```text
          100111
       +---------
1011   | 101011000
         1011
         ----
          00111
          00000
          -----
           01110
           1011
           ----
            1010
            1011
            ----
             00100
             00000
             -----
              01000
              1011
              -----
               011  (Remainder R = 011)
```

1. Transmitted 9-bit frame: $D \cdot 2^3 \text{ XOR } R = \mathbf{101011011}$.

#### Solution 2

* Data Matrix with Row/Column Parity:

```text
1 0 1 1 | 1 (Row 1)
0 1 1 0 | 0 (Row 2)
1 1 0 1 | 1 (Row 3)
0 0 1 0 | 1 (Row 4)
--------+--
0 0 1 0 | 1 (Col Parity)
```

* Bit Flip: Row 2, Column 3 becomes 0.
* Detection & Correction:
  * Recalculating Row 2 parity yields 1 (differs from logged Row 2 parity 0 -> Error in Row 2).
  * Recalculating Column 3 parity yields 0 (differs from logged Column 3 parity 1 -> Error in Column 3).
  * Intersection (Row 2, Column 3) identifies the exact flipped bit, which is inverted back from 0 to 1.

#### Solution 3

1. $S(p) = N p (1-p)^{N-1}$.
2. Take derivative with respect to p and set to 0:

   $$\frac{dS}{dp} = N(1-p)^{N-1} - N(N-1)p(1-p)^{N-2} = 0 (1-p) - (N-1)p = 0 \implies 1 - N p = 0 \implies p^* = \frac{1}{N}$$
3. Substitute $p^* = 1/N$:

   $$\lim_{N \to \infty} S(p^*) = \lim_{N \to \infty} \left(1 - \frac{1}{N}\right)^{N-1} = \frac{1}{e} \approx 0.368 \text{ (36.8\%)}$$

#### Solution 4

1. For $n = 3$, $K \in \{0, 1, 2, \dots, 2^3 - 1\} = \{0, 1, 2, 3, 4, 5, 6, 7\}$.
2. $P(K = 2) = 1/8 = 0.125$.
3. For $K = 2$, wait time = $2 \times 512\text{ bit times} = 1024\text{ bit times}$. On a 10 Mbps link ($0.1\,\mu\text{s/bit}$):

   $$\text{Wait Time} = 1024 \times 0.1\,\mu\text{s} = \mathbf{102.4\,\mu\text{s}}$$

#### Solution 5

1. Event 1 (A -> B): Switch learns A is on Port 1. Target B not in table -> Flooded to Ports 2, 3, 4. Table: [A: Port 1].
2. Event 2 (B -> A): Switch learns B is on Port 2. Target A is in table on Port 1 -> Forwarded Unicast to Port 1. Table: [A: Port 1, B: Port 2].
3. Event 3 (C -> B): Switch learns C is on Port 3. Target B is in table on Port 2 -> Forwarded Unicast to Port 2. Table: [A: Port 1, B: Port 2, C: Port 3].
4. Event 4 (A -> B): Target B is in table on Port 2 -> Forwarded Unicast to Port 2. Table updated/refreshed.

#### Solution 6

* Advantages of Switches over Routers:
  1. Plug-and-play installation requiring zero IP subnet configuration.
  2. Extremely low forwarding latency due to hardware Layer-2 lookup.
  3. Lower cost per port compared to full Layer-3 routers.
* Advantages of Routers over Switches:
  1. Complete isolation of Layer-2 broadcast storms.
  2. Hierarchical IP addressing permits non-spanning tree, multi-path routing topologies.
  3. Fine-grained security filtering and bandwidth management across subnets.

#### Solution 7

1. Protocol Layer: ARP operates at Layer 2/3 boundary (translates IP to MAC); DNS is an Application-Layer protocol (Layer 7).
2. Resolution Domain: ARP resolves addresses strictly within a single local IP subnet; DNS resolves hostnames globally across the Internet.
3. Transport Mechanism: ARP uses Layer-2 broadcast frames (FF-FF-FF-FF-FF-FF); DNS uses unicast UDP/IP datagrams sent to specific DNS servers (port 53).

#### Solution 8

An 802.1Q tag identifies the originating VLAN of a frame as it traverses a shared trunk link between switches. It is inserted directly after the Source MAC address field (before the Type/Length field). It includes a 12-bit VLAN ID field, supporting up to $2^{12} = 4096$ distinct VLANs.

#### Solution 9

ECMP hashes a packet's 5-tuple (Source IP, Destination IP, Source Port, Destination Port, Protocol) to select one of multiple equal-cost paths through Tier-2 and Tier-1 switches. This splits traffic across parallel paths, preventing bottleneck links while keeping packets within a single flow on the same path to prevent out-of-order delivery.

#### Solution 10

1. Laptop -> School Router Link:
  * Source IP: 68.85.2.101 (Laptop IP)
  * Destination IP: 68.87.71.226 (Comcast DNS Server IP)
  * Source MAC: 00:16:D3:23:68:8A (Laptop MAC)
  * Destination MAC: 00:22:6B:45:1F:1B (School Gateway Router MAC)
2. School Router -> Comcast Network Link:
  * Source IP: 68.85.2.101 (Laptop IP preserved)
  * Destination IP: 68.87.71.226 (Comcast DNS Server IP preserved)
  * Source MAC: School Router's WAN Interface MAC
  * Destination MAC: Next-Hop Comcast Router's MAC

## 9. Synthesis Discussion Prompts

1. The Death of Collision Detection: Modern Ethernet LANs exclusively use full-duplex switches, completely eliminating collisions. Why does IEEE 802.3 retain CSMA/CD specification in the standard, and what architectural changes occur when shifting to 100GbE and 400GbE networks?
2. Layer-2 vs. Layer-3 Datacenter Fabrics: Examine the architectural trade-offs between deploying a flat, massive Layer-2 VLAN-switched datacenter fabric versus a Layer-3 routed IP Clos network.
3. Security Vulnerabilities in ARP: ARP lacks authentication mechanisms. Describe the mechanism of an ARP Poisoning (ARP Spoofing) attack, its impact on local network security, and how Dynamic ARP Inspection (DAI) mitigates the threat.
4. MPLS vs. Segment Routing (SRv6): How does modern Segment Routing simplify traditional MPLS label-distribution protocols (LDP/RSVP-TE) by leveraging native IPv6 extension headers?
5. Impact of Wireless MAC Protocols on Upper-Layer Performance: Contrast Ethernet CSMA/CD with Wi-Fi CSMA/CA (802.11). How do wireless frame ACKs and hidden terminal problems affect TCP congestion window dynamics over wireless access links?

## 10. Complete Glossary of Core Terminology

* Address Resolution Protocol (ARP): A Layer-2/3 protocol that dynamically resolves a known IP address to a corresponding physical MAC address within a local subnet.
* Aging Time: The duration for which a self-learning switch retains an entry in its switch table before purging it if no matching source frames arrive.
* ALOHA (Pure and Slotted): Early random access protocols for broadcast channels. Slotted ALOHA synchronizes transmissions to time slots, achieving 36.8% maximum efficiency, whereas unslotted Pure ALOHA achieves 18.4% maximum efficiency.
* Binary Exponential Backoff: An algorithm used in CSMA/CD and Wi-Fi where a node chooses a random delay from an exponentially expanding contention window after consecutive collisions.
* Broadcast Channel: A communication channel where transmitted signals propagate to all connected nodes.
* Carrier Sense Multiple Access (CSMA): A MAC protocol where nodes listen to the physical medium before transmitting to avoid sending over a busy channel.
* CSMA with Collision Detection (CSMA/CD): A variant of CSMA used in legacy Ethernet where nodes monitor the channel during transmission and abort immediately if a collision is detected.
* Cyclic Redundancy Check (CRC): A powerful, hardware-implemented error detection code based on modulo-2 polynomial division.
* Equal-Cost Multi-Path (ECMP): A routing technique that splits traffic flows across multiple equal-cost network paths using header hashing.
* Ethernet: The dominant wired LAN standard defined under IEEE 802.3, specifying Layer-1 physical media and Layer-2 frame formats.
* Filtering: The switch operation of dropping a frame if its destination MAC address resides on the same switch interface on which it arrived.
* Forward Error Correction (FEC): Error-correcting codes (such as 2D parity) that enable a receiver to detect and correct bit errors without retransmission.
* Forwarding: The switch or router operation of transferring a packet from an input interface to a target output interface.
* Frame: The protocol data unit (PDU) of the link layer, encapsulating a network-layer datagram with Layer-2 headers and trailers.
* Generator Polynomial (G): A bit pattern used as the divisor in CRC polynomial calculations.
* Hub: A Layer-1 physical device that broadcasts incoming bits from one port onto all other ports without buffering or filtering.
* Link: The physical or logical communication channel connecting adjacent networking nodes.
* Link-Layer Switch: A Layer-2 store-and-forward device that filters and forwards frames using self-learning MAC address tables.
* MAC Address: A flat, globally unique 48-bit hardware address burned into a NIC during manufacturing.
* Medium Access Control (MAC) Protocol: The set of rules governing how nodes share access to a broadcast communication channel.
* Middlebox: Any network device along the data path that performs functions beyond standard IP forwarding (e.g., NAT, Firewalls, Load Balancers).
* Multiprotocol Label Switching (MPLS): A Layer-2.5 forwarding protocol that routes packets using 20-bit fixed-length labels inserted between Layer 2 and Layer 3 headers.
* Network Interface Card (NIC): The physical expansion card or built-in chip implementing Layer-1 and Layer-2 protocol controllers.
* Parity Bit: An extra bit added to a bit sequence to make the total number of set bits even or odd, providing basic error detection.
* Self-Learning: The capability of a Layer-2 switch to construct its forwarding table dynamically by inspecting the source MAC addresses of arriving frames.
* Spanning Tree Protocol (STP): A Layer-2 network protocol that creates a loop-free logical topology within a switched Ethernet LAN with redundant physical links.
* Top-of-Rack (ToR) Switch: A switch mounted inside a server rack that connects the rack's blade servers to the broader datacenter switch fabric.
* VLAN Trunking: A method of connecting multi-VLAN switches using a shared link that carries tagged frames (IEEE 802.1Q).
* Virtual Local Area Network (VLAN): A logical partitioning of a physical switch into distinct, isolated broadcast domains.