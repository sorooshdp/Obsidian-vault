Comprehensive Study Guide: Computer Networking — Chapter 7 (Wireless and Mobile Networks)

This study guide provides an authoritative, self-contained rewrite of Chapter 7 ("Wireless and Mobile Networks") from Computer Networking: A Top-Down Approach (8th Edition by James Kurose and Keith Ross). It systematically examines the physical properties of wireless transmission links, medium access control protocols, the architecture of IEEE 802.11 (WiFi) and Bluetooth personal area networks, 4G LTE and 5G cellular architectures, fundamental network-layer mobility management principles, and the operational impact of wireless channels on higher-layer transport protocols.

1. Introduction & Wireless Network Taxonomy

Wireless networks extend the reach of the Internet to billions of untethered end systems, ranging from smartphones and laptops to autonomous vehicles and Internet of Things (IoT) sensors. While wireless communication relies on the radio spectrum rather than physical copper or optical cables, the underlying network architecture retains core structural principles: protocol layering, edge/core separation, hierarchical addressing, and the decoupling of control and data planes.

1.1 Elements of a Wireless Network

A generalized wireless network architecture consists of four primary structural components:

* Wireless Hosts: End-system devices running application-layer software. Wireless hosts may be stationary (e.g., a desktop PC with a WiFi card or a fixed surveillance camera) or mobile (e.g., a smartphone or autonomous drone).
* Wireless Links: The physical communication channels operating over the electromagnetic radio spectrum. Wireless links connect hosts to base stations or to other wireless hosts, displaying distinct transmission rates, operational frequencies, and bit-error characteristics compared to wired links.
* Base Stations: Fixed infrastructure elements responsible for managing wireless radio resources and coordinating transmissions within their coverage area. In infrastructure-based networks, base stations function as link-layer relays between wireless hosts and the wired network backbone. Examples include IEEE 802.11 Access Points (APs) and cellular base stations (4G eNode-Bs and 5G gNode-Bs).
* Network Backbone / Core Infrastructure: The underlying wired or core network (e.g., the public Internet, enterprise subnets, or cellular Enhanced Packet Core) that interconnects base stations and provides core services such as address allocation, routing, authentication, and gateway interconnectivity.

1.2 Taxonomy of Wireless Networks

Wireless networks are broadly categorized along two fundamental dimensions: (1) whether communication occurs over a single wireless hop or multiple wireless hops, and (2) whether the network relies on fixed base station infrastructure.

Topology & Architecture	Single-Hop	Multi-Hop
Infrastructure-Based	Class 1: Host connects directly to a base station (e.g., standard 802.11 WiFi BSS, 4G LTE, 5G NR cellular cells). All traffic flows through the base station.	Class 2: Host connects to a base station via intermediate wireless relay nodes (e.g., wireless mesh networks, corporate ESS with wireless backhaul).
Infrastructure-Less (Ad-Hoc)	Class 3: Wireless hosts communicate directly with each other without a base station (e.g., Bluetooth piconets, 802.11 ad-hoc mode, Wi-Fi Direct).	Class 4: Wireless nodes relay packets across multiple wireless hops without central control (e.g., Mobile Ad-Hoc Networks [MANETs], Vehicular Ad-Hoc Networks [VANETs]).

1.3 Comparative Standards Matrix

Wireless access technologies span different operational distances, transmission speeds, and radio frequency bands:

Technology / Standard	Common Name	Frequency Band	Max Bitrate Range	Typical Distance Range	Primary Application
IEEE 802.15.1	Bluetooth	2.4 GHz ISM	1 – 3 Mbps	10 – 30 m	Personal Area Networks (WPANs), peripherals
IEEE 802.11b	WiFi 1	2.4 GHz	11 Mbps	30 – 100 m	Legacy residential/office WLAN
IEEE 802.11g	WiFi 3	2.4 GHz	54 Mbps	30 – 100 m	Residential/office WLAN
IEEE 802.11n	WiFi 4	2.4 GHz / 5 GHz	Up to 600 Mbps	50 – 100 m	High-speed indoor WLAN
IEEE 802.11ac	WiFi 5	5 GHz	Up to 3.5 Gbps	30 – 50 m	High-density indoor WLAN
IEEE 802.11ax	WiFi 6	2.4 GHz / 5 GHz / 6 GHz	Up to 9.6 Gbps	30 – 50 m	High-efficiency indoor/outdoor WLAN
4G LTE	Cellular 4G	700 MHz – 2.6 GHz	100 Mbps – 1 Gbps	2 km – 10 km	Wide-area mobile cellular data
5G FR1	5G Sub-6 GHz	410 MHz – 7.125 GHz	100 Mbps – 1 Gbps	1 km – 5 km	Medium-range high-capacity cellular
5G FR2	5G mmWave	24.25 GHz – 52.6 GHz	Up to 10 Gbps	100 m – 500 m	Ultra-high-speed dense urban wireless

2. Wireless Links & Physical-Layer Channel Characteristics

Wireless transmission media differ significantly from wired media (e.g., copper twisted-pair or optical fiber) due to electromagnetic wave attenuation, environmental noise, and physical obstacle interactions.

2.1 Physical Channel Impairments

Three main phenomena impair wireless signal transmission:

1. Path Loss and Attenuation: Electromagnetic radiation disperses over distance in free space. When propagating through matter (walls, foliage, precipitation), signal strength decreases rapidly. Path loss increases with both distance and carrier frequency.
2. Interference from Other Sources: Unlicensed frequency bands (e.g., 2.4 GHz ISM) are shared by multiple consumer electronics, including 802.11 WiFi, Bluetooth, cordless telephones, and microwave ovens. Signals transmitted on overlapping frequencies induce destructive interference at the receiver.
3. Multipath Propagation: Electromagnetic waves reflect off physical structures, terrain, and moving objects, creating multiple propagation paths of varying physical lengths between sender and receiver. The arriving signal components experience phase shifts, resulting in inter-symbol interference and multipath fading.

2.2 Signal-to-Noise Ratio (SNR) and Bit Error Rate (BER)

At the receiver, an incoming signal is received alongside environmental thermal noise. The relative strength of the received signal to background noise is quantified by the Signal-to-Noise Ratio (SNR), defined in decibels (dB) as:

\text{SNR}_{\text{dB}} = 10 \cdot \log_{10}\left(\frac{P_{\text{signal}}}{P_{\text{noise}}}\right) = 20 \cdot \log_{10}\left(\frac{A_{\text{signal}}}{A_{\text{noise}}}\right)

Where P represents signal power and A represents wave amplitude. A higher SNR indicates a clearer signal that is easier for the receiver's physical layer hardware to decode correctly.

The physical layer uses digital modulation schemes to encode binary data onto radio carrier waves:

* Low-Order Modulation (e.g., BPSK / QPSK): Encodes fewer bits per signal symbol (e.g., 1 or 2 bits/symbol). Highly resilient to noise, maintaining a low Bit Error Rate (BER) even at low SNR values, but yields lower transmission bitrates.
* High-Order Modulation (e.g., QAM16 / QAM256): Encodes many bits per signal symbol (e.g., 4 or 8 bits/symbol). Delivers significantly higher bitrates but requires a very high SNR; minor noise fluctuations induce symbol misclassification and higher BER.

Modern wireless protocols implement Adaptive Modulation and Coding (AMC). As a mobile host moves away from a base station and its SNR drops, the system dynamically switches from high-order modulation (e.g., QAM256) down to resilient low-order modulation (e.g., QPSK) to preserve link reliability at the cost of transmission speed.

2.3 Multiple Access Impairments: Hidden Terminals and Fading

Unlike wired broadcast networks (where every attached adapter detects transmissions from all other adapters), wireless channels suffer from spatial physical constraints:

1. Hidden Terminal Problem: Station A and Station C are separated by a physical obstruction (e.g., a mountain or building). A cannot hear C's transmissions and vice versa. However, both A and C can reach Base Station B. If A and C transmit simultaneously, their signals collide at B, even though neither sender detected the other's ongoing transmission.
2. Signal Fading / Attenuation: Station A and Station C are physically far apart such that their signal power decays below their respective carrier-sensing detection thresholds. However, both signals retain sufficient power to reach intermediate Station B, resulting in uncoordinated collisions at B.

2.4 Code Division Multiple Access (CDMA)

Code Division Multiple Access (CDMA) is a channel partitioning protocol widely used in cellular and satellite systems. Instead of partitioning the channel by distinct time slots (TDM) or distinct frequency bands (FDM), CDMA assigns each sender a unique M-bit code vector called a chipping code, denoted as:

c = (c_1, c_2, c_3, \dots, c_M) \quad \text{where } c_m \in \{+1, -1\}

Mathematical Model of CDMA:

1. Data Representation: A binary data bit d_i = 1 is represented as +1, and d_i = 0 is represented as -1. An idle sender transmits 0.
2. Sender Encoding: For the i-th bit slot, the sender computes the M-element chip output vector Z_{i,m} by multiplying data bit d_i by the chipping code vector c_m:

Z_{i,m} = d_i \cdot c_m \quad \text{for } m = 1, 2, \dots, M

1. Additive Channel Property: Assuming synchronized transmission over the physical air interface, the composite signal received on the channel Z^*_{i,m} is the linear sum of encoded signals from all N active senders:

Z^*_{i,m} = \sum_{s=1}^N Z^s_{i,m} = \sum_{s=1}^N d_i^s \cdot c_m^s

1. Receiver Decoding: To extract the data transmitted by a specific target sender s, the receiver computes the normalized inner product (dot product) of the aggregate channel signal Z^*_{i,m} and sender s's unique code vector c^s:

d_i^s = \frac{1}{M} \sum_{m=1}^M Z^*_{i,m} \cdot c_m^s

Orthogonal code design ensures that c^s \cdot c^k = 0 for s \neq k and c^s \cdot c^s = M, allowing the receiver to completely filter out interfering senders.

Worked Numeric Example of 2-Sender CDMA:

Let M = 8.

* Sender 1 Code: c^1 = (1, 1, 1, -1, 1, -1, -1, -1)
* Sender 2 Code: c^2 = (1, -1, 1, 1, 1, -1, 1, 1)

Suppose Sender 1 wishes to send data bit d_1^1 = 1 (+1) and Sender 2 wishes to send data bit d_1^2 = 0 (represented as -1).

1. Sender 1 Transmits: Z^1_{1,m} = (+1) \cdot c^1 = (1, 1, 1, -1, 1, -1, -1, -1)
2. Sender 2 Transmits: Z^2_{1,m} = (-1) \cdot c^2 = (-1, 1, -1, -1, -1, 1, -1, -1)
3. Aggregate Signal on Channel:

Z^*_{1,m} = Z^1_{1,m} + Z^2_{1,m} = (1-1, 1+1, 1-1, -1-1, 1-1, -1+1, -1-1, -1-1) = (0, 2, 0, -2, 0, 0, -2, -2)

1. Decoding at Receiver for Sender 1:

\sum_{m=1}^8 Z^*_{1,m} \cdot c_m^1 = (0)(1) + (2)(1) + (0)(1) + (-2)(-1) + (0)(1) + (0)(-1) + (-2)(-1) + (-2)(-1) = 8

d_1^1 = \frac{+8}{8} = +1 \implies \mathbf{\text{Bit } 1}

1. Decoding at Receiver for Sender 2:

\sum_{m=1}^8 Z^*_{1,m} \cdot c_m^2 = (0)(1) + (2)(-1) + (0)(1) + (-2)(1) + (0)(1) + (0)(-1) + (-2)(1) + (-2)(1) = -8

d_1^2 = \frac{-8}{8} = -1 \implies \mathbf{\text{Bit } 0}

Both original bits are recovered without error.

3. WiFi: IEEE 802.11 Wireless LANs

The IEEE 802.11 standard family governs wireless local area networks. All 802.11 variants share a common frame format and medium access protocol.

3.1 Architecture, BSS, and Channels

An 802.11 architecture is built around the Basic Service Set (BSS). A BSS contains one or more wireless stations and a central Access Point (AP) operating in infrastructure mode.

* Service Set Identifier (SSID): A 1-to-32-byte alphanumeric name assigned to an AP by a network administrator to identify the wireless network.
* Frequency Channels: 802.11b/g/n operates in the 2.4 GHz to 2.4835 GHz band, divided into 11 partially overlapping channels in North America. Channels must be separated by at least 4 channel numbers to avoid spectral overlap; the set {Channel 1, Channel 6, Channel 11} is the only set of three non-overlapping channels in 2.4 GHz space.
* WiFi Jungle: Any location where a mobile node receives beacon frames from multiple overlapping APs.

3.2 AP Scanning and Association

Before exchanging network-layer datagrams, a wireless host must associate with an AP through either passive or active scanning:

1. Passive Scanning:
  * APs periodically broadcast Beacon Frames (typically every 100 ms) containing the AP's SSID, MAC address, and supported data rates.
  * The host scans the 11 channels, records arriving beacons, selects an AP (typically the one with the strongest SNR), and sends an Association Request frame.
  * The AP verifies credentials and replies with an Association Response frame.
2. Active Scanning:
  * The wireless host broadcasts a Probe Request frame across all channels.
  * All APs within range process the probe request and respond with a Probe Response frame.
  * The host chooses an AP and executes the Association Request/Response exchange.

3.3 IEEE 802.11 CSMA/CA MAC Protocol

Because wireless adapters cannot reliably detect collisions while transmitting (due to signal attenuation between local transmit power and incoming receive power, as well as hidden terminals), 802.11 uses Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA) rather than CSMA/CD.

Step-by-Step CSMA/CA Protocol Algorithm:

1. Initial Transmit Check: If a station with a frame to send senses the channel idle for a period of Distributed Inter-frame Space (DIFS), it transmits the frame immediately.
2. Backoff Timer Execution: If the channel is sensed busy, the station defers transmission and selects a random backoff value chosen uniformly from a contention window [0, CW-1].
3. Timer Decrement and Freezing: The station monitors the channel. While the channel remains idle, the backoff timer counts down. If the channel becomes busy, the timer is frozen. When the channel becomes idle again for DIFS, countdown resumes.
4. Transmission: When the backoff timer reaches zero, the station transmits the complete 802.11 data frame.
5. Link-Layer Acknowledgment: Because bit error rates are high, the destination station, upon receiving a frame that passes the CRC check, waits a short interval called Short Inter-frame Space (SIFS) and returns an explicit Acknowledgment (ACK) frame.
6. Success / Collision Determination: If the sender receives the ACK, the transaction succeeds. If no ACK arrives within a timeout window, the sender assumes a collision or channel error occurred, doubles the contention window size CW (exponential backoff), and returns to Step 2 to attempt retransmission.

RTS/CTS Channel Reservation Mechanism

To mitigate the hidden terminal problem for long data frames, 802.11 provides an optional reservation exchange using short control frames:

1. The sender transmits a Request to Send (RTS) frame to the AP. The RTS header includes a Duration field specifying the total microsecond window required to transmit the DATA frame and its returning ACK.
2. The AP responds after SIFS with a Clear to Send (CTS) frame, echoing the requested reservation duration.
3. The CTS frame is heard by all stations within the AP's range (including terminals hidden from the original sender). All hearing stations defer transmissions for the specified duration (setting their Network Allocation Vector [NAV]).
4. The sender transmits its DATA frame after SIFS, and the AP responds with an ACK after SIFS.

RTS Threshold: Because RTS/CTS control exchanges consume channel overhead, stations only trigger RTS/CTS when frame payload size exceeds an adjustable parameter called the RTS Threshold.

3.4 IEEE 802.11 Frame Structure

An 802.11 MAC frame contains four 6-byte address fields, contrasting with the two address fields in an Ethernet frame:

\begin{array}{|c|c|c|c|c|c|c|c|c|} \hline \text{Frame Ctrl} & \text{Duration} & \text{Address 1} & \text{Address 2} & \text{Address 3} & \text{Seq Ctrl} & \text{Address 4} & \text{Payload} & \text{CRC} \ \text{(2 B)} & \text{(2 B)} & \text{(6 B)} & \text{(6 B)} & \text{(6 B)} & \text{(2 B)} & \text{(6 B)} & \text{(0-2312 B)} & \text{(4 B)} \ \hline \end{array}

Address Field Semantics in Infrastructure Mode:

* Address 1 (Receiver MAC): The MAC address of the wireless station or AP that is the immediate wireless receiver of the frame (e.g., AP MAC when sending upstream; Host MAC when sending downstream).
* Address 2 (Transmitter MAC): The MAC address of the wireless station or AP that transmitted the frame onto the wireless link.
* Address 3 (Router Interface MAC): The MAC address of the destination/source router interface attached to the wired distribution network. Address 3 enables the AP to bridge frames between the 802.11 wireless domain and an Ethernet 802.3 wired subnet.
* Address 4: Reserved exclusively for ad-hoc mesh network topologies where APs forward frames to each other wirelessly.

Frame Control Subfields:

* Type and Subtype: Identifies frame function (Management: Beacon/Association; Control: RTS/CTS/ACK; Data).
* To AP / From AP: Indicates directional data flow relative to the infrastructure AP, controlling interpretation of the address fields.
* More Frag / Retry: Indicates fragment continuation or frame retransmission status.
* Power Management: Set to 1 if the transmitting station enters sleep mode after frame exchange.

3.5 Intra-Subnet Mobility

When a mobile host moves physically from BSS 1 to BSS 2 within the same IP subnet (connected to an Ethernet switch):

1. Host H1 detects decaying signal strength from AP 1 and receives strong beacons from AP 2 (sharing the same SSID).
2. H1 disassociates from AP 1 and executes association with AP 2.
3. H1 retains its assigned IP address and keeps all active transport-layer (TCP) socket connections open.
4. To update the link-layer forwarding table of the attached Ethernet switch, AP 2 generates an Ethernet frame with H1's source MAC address and broadcasts it over the wired switch port. The switch processes the frame, updates its MAC self-learning table to point H1's MAC address to AP 2's switch port, and future frames for H1 are routed to AP 2.

3.6 Advanced Features: Rate Adaptation and Power Management

* Rate Adaptation: 802.11 physical layers monitor transmission error rates. If a host misses two consecutive ACKs, the firmware automatically steps down its modulation order (e.g., from QAM54 to QAM16). If 10 consecutive frames succeed, the transmitter probes a higher bitrate step.
* Power Management: Wireless nodes conserve battery by entering sleep states. Before sleeping, a node sets the Power Management bit in a frame header to 1. The AP buffers all downstream frames destined for the sleeping node. The node wakes up every 100 ms to inspect the Traffic Indication Map (TIM) inside the AP's beacon frame. If its MAC address is flagged, the node sends a PS-Poll frame to retrieve buffered frames; otherwise, it returns to sleep (achieving ~99% idle power reduction).

3.7 Personal Area Networks (WPANs) & Bluetooth

IEEE 802.15.1 (Bluetooth) provides low-cost, short-range, low-power ad-hoc connectivity:

* Frequency Hopping Spread Spectrum (FHSS): Bluetooth operates in the 2.4 GHz ISM band across 79 discrete channels. Transmitters hop pseudo-randomly across channels 1,600 times per second over 625 \mus TDM slots to mitigate interference from co-located WiFi devices.
* Piconet Topology: An ad-hoc collection of up to 8 active devices. One node is designated as Master, controlling time synchronization, slot sequences, and polling. Up to 7 active Client nodes transmit only when polled by the Master. Up to 255 Parked nodes remain synchronized in low-power sleep mode until activated by the Master.

4. Cellular Networks: 4G LTE and 5G

Cellular networks provide wide-area, high-speed mobile Internet access across connected geographic cells managed by commercial carriers.

4.1 4G LTE Architecture and System Elements

4G Long-Term Evolution (4G LTE) uses an all-IP network architecture divided into the Radio Access Network (RAN) at the edge and the Enhanced Packet Core (EPC) in the core.

Core System Elements:

* Mobile Device / User Equipment (UE): The end-system host running user applications. Contains a Subscriber Identity Module (SIM) card storing a globally unique 64-bit International Mobile Subscriber Identity (IMSI) and symmetric authentication key K_{\text{HSS-M}}.
* Base Station (eNode-B): Manages radio resources and cell coverage area. Handles wireless channel access, scheduling, mutual authentication relaying, and IP tunnel endpoint management.
* Home Subscriber Server (HSS): A central control-plane database in the subscriber's home network storing account profile data, access privileges, and key derivation secrets.
* Mobility Management Entity (MME): The primary control-plane coordinator in the visited network. Responsible for device authentication (interacting with HSS), mobility session setup, paging sleeping devices, and tunnel establishment. It is not in the user data forwarding path.
* Serving Gateway (S-GW): A data-plane router acting as the local mobility anchor point. All user IP packets pass through the S-GW via encapsulated GTP tunnels.
* Packet Data Network Gateway (P-GW): The edge gateway router connecting the EPC to the public Internet. Assigns NAT IP addresses to UEs, enforces QoS policies, and masks UE mobility from external servers.

4.2 LTE Protocol Stacks

The 4G LTE user data plane encapsulates network-layer IP datagrams within carrier-specific link sublayers and tunneling protocols:

Link Sublayers in LTE:

1. Packet Data Convergence Protocol (PDCP): Sits directly below IP. Performs IP header compression (ROHC) to reduce wireless transmission overhead and executes AES encryption/decryption of user payload bits.
2. Radio Link Control (RLC): Handles packet fragmentation and reassembly when IP datagrams exceed physical slot sizes. Implements link-layer reliable data transfer using ACK/NAK ARQ.
3. Medium Access Control (MAC): Coordinates transmission scheduling across physical frequency/time slots and executes Hybrid ARQ (HARQ) with forward error correction.

User Data Path Tunneling:

Data plane traffic between the eNode-B, S-GW, and P-GW is encapsulated inside GPRS Tunneling Protocol (GTP-U) tunnels running over UDP/IP. Each tunnel is identified by a 32-bit Tunnel Endpoint Identifier (TEID).

4.3 LTE Radio Access Network Scheduling

LTE downlinks use Orthogonal Frequency Division Multiplexing (OFDM), combining frequency and time division multiplexing:

* Channel bandwidth is structured into 0.5 ms time slots spanning distinct subcarrier frequencies.
* Two 0.5 ms slots constitute a 1.0 ms subframe; 10 subframes form a 10 ms LTE frame.
* Opportunistic Scheduling: The eNode-B dynamically allocates slots to UEs every millisecond based on channel SNR measurements, user priority tiers (Gold/Silver), and queue backlogs.

4.4 LTE Network Attachment and Power Management

3-Phase Network Attachment Process:

1. Base Station Synchronization: The UE scans radio channels for Primary and Secondary Synchronization Signals (PSS/SSS) broadcast every 5 ms by nearby base stations. It determines frame timing, cell identity, and carrier parameters, selecting a base station for access.
2. Mutual Authentication and Key Agreement (AKA): The base station relays the UE's attach request and IMSI to the local MME. The MME communicates with the home HSS to execute mutual cryptographic challenge-response authentication, deriving link session keys (K_{\text{BS-M}}).
3. Data Path Tunnel Configuration: The MME signals the P-GW to allocate an IP address (via NAT) and instructs the S-GW and eNode-B to construct GTP user-plane tunnels.

Power Management States:

* Discontinuous Reception (DRX / Light Sleep): Entered after several hundred milliseconds of inactivity. The UE and eNode-B synchronize periodic wake-up intervals (e.g., every 1280 ms) for the UE to monitor downstream control channels.
* Idle State (Deep Sleep): Entered after 5–10 seconds of inactivity. The eNode-B releases radio resources and UE context. If downstream packets arrive at the P-GW for an idle UE, the MME issues a Paging Message broadcast across all base stations in the UE's tracking area to locate and wake the UE.

4.5 5G Cellular Networks

5G New Radio (NR) networks enhance performance across three primary application pillars:

1. Enhanced Mobile Broadband (eMBB): Delivers gigabit bitrates (up to 10–20 Gbps) for high-definition media streaming and AR/VR.
2. Ultra-Reliable Low-Latency Communications (URLLC): Targets sub-millisecond latencies (< 1 ms) and 99.999% reliability for mission-critical services such as autonomous driving, remote surgery, and industrial robotics.
3. Massive Machine Type Communications (mMTC): Supports dense deployments of low-cost, low-power IoT sensors (up to 1,000,000 devices/km^2).

Spectrum Bands and Physical Innovations:

* Frequency Range 1 (FR1): Sub-6 GHz bands (410 MHz – 7.125 GHz). Provides wide coverage and wall penetration comparable to 4G LTE.
* Frequency Range 2 (FR2 - mmWave): Millimeter-wave bands (24.25 GHz – 52.6 GHz). Offers ultra-wide channel bandwidths (up to 400 MHz) delivering extreme throughput over short line-of-sight distances (100–300 m).
* Massive MIMO & Beamforming: Base stations deploy large antenna arrays (e.g., 64 or 128 elements). Spatial signal processing concentrates radio energy into narrow, targeted directional beams toward specific users, expanding capacity and spectral efficiency.

5G Core Architecture (5GC):

The 5GC uses a virtualized, cloud-native Service-Based Architecture (SBA) leveraging Network Functions Virtualization (NFV) and Network Slicing:

* Access and Mobility Management Function (AMF): Handles connection and mobility management, decoupling these from session management.
* Session Management Function (SMF): Manages IP address allocation, DHCP, and session control.
* User Plane Function (UPF): High-performance data forwarding engine deployed at network edges to minimize latency.

5. Mobility Management Principles

Mobility management ensures that a mobile device maintains active transport-layer connections as it changes its network point of attachment.

5.1 Device Mobility Spectrum

From a network-layer perspective, device mobility falls along a spectrum:

1. Serial Attachment (Powered Off During Transit): Host disconnects from Network A, powers off, travels to Network B, and powers on. Obtains a new IP address via DHCP. Higher-layer connections are not preserved.
2. Intra-Subnet Mobility: Host moves between access points attached to the same IP subnet. Handled entirely at the link layer (802.11 re-association) without altering the IP address or disrupting TCP connections.
3. Inter-Subnet Handover: Host moves across distinct IP subnets or cellular cells while actively sending/receiving data. Requires network-layer/EPC handover mechanisms to maintain uninterrupted socket communication.
4. Global Network Roaming: Host moves into a visited network managed by a different cellular service provider. Requires coordination between the visited network and the subscriber's home network.

5.2 Addressing and Routing Options for Mobile Devices

When a host roams into a visited network, it possesses two identity attributes:

* Permanent Identifier: An address/identity associated with its home network (e.g., home IP address or cellular SIM IMSI).
* Transient / Care-of Address (CoA): A temporary network address allocated within the visited network's address space (or allocated via NAT at the visited gateway).

Evaluation of Routing Architecture Paradigms:

1. Naive BGP Host-Route Injection:
  * Mechanism: When a host enters a visited network, the visited network advertises a specific /32 host route for the device's permanent IP address via BGP to all global routers.
  * Failure Mode: Unscalable. Global Internet routing tables would be forced to store billions of dynamic /32 host entries, triggering routing table inflation and global update churn whenever a device moves.
2. Indirect Routing (Home-Anchored Tunneling):
  * Mechanism: The correspondent sends datagrams to the mobile node's permanent IP address. Datagrams naturally route to the home network's gateway router. The home gateway looks up the mobile node's current location in the HSS database, encapsulates the original packet inside an IP tunnel, and forwards it to the visited network's gateway. The visited gateway decapsulates the packet and delivers it to the mobile node.
  * Triangle Routing Problem: If the correspondent and mobile host are physically close to each other, but far from the home network, packets must travel to the home network first and then back to the visited network, introducing latency.
  * Local Breakout: For mobile-originated traffic, the visited network can route outgoing packets directly to the correspondent over the public Internet, bypassing the home network for the reverse path.
3. Direct Routing:
  * Mechanism: The correspondent first queries the home network HSS to obtain the mobile host's current Care-of Address in the visited network. The correspondent then tunnels datagrams directly to the visited network gateway, avoiding the home gateway data path.
  * Trade-offs: Eliminates triangle routing latency, but introduces initial query delay and requires complex signaling to update the correspondent if the mobile node moves mid-session.

6. Mobility Management in Practice & Impact on Higher-Layer Protocols

6.1 4G/5G Handover Execution Procedure

In 4G LTE/5G networks, handover transfers an active mobile session from a Source Base Station (eNode-B) to a Target Base Station (eNode-B) without dropping packets:

1. Measurement Reporting: The UE periodically measures signal quality (RSRP/RSRQ) from its serving cell and adjacent candidate cells, transmitting reports to the source eNode-B.
2. Handover Decision: Based on signal degradation or cell load balancing, the source eNode-B selects a target eNode-B and sends a Handover Request over the direct inter-base-station X2 interface.
3. Target Resource Reservation: The target eNode-B verifies available radio capacity and pre-allocates time/frequency slots and a target GTP tunnel TEID. It returns a Handover Request Acknowledge message containing channel parameters.
4. Handover Execution Command: The source eNode-B transmits an RRC Connection Reconfiguration command to the UE. The UE detaches from the source eNode-B and synchronizes with the target eNode-B.
5. Data Forwarding: The source eNode-B stops transmitting packets over the air and forwards all undelivered, buffered downstream IP datagrams to the target eNode-B over the X2 tunnel.
6. Path Switching (Control Plane): The target eNode-B notifies the MME via a Path Switch Request. The MME signals the S-GW to update its user-plane GTP tunnel endpoint to point to the target eNode-B.
7. Resource Release: Once the S-GW updates its tunnel path, the target eNode-B notifies the source eNode-B, which releases the old radio and context resources.

6.2 Mobile IP (RFC 5944)

Mobile IP provides network-layer mobility for IPv4 networks:

* Home Agent (HA): A router in the mobile node's home network that maintains location state and tunnels packets to the visited network.
* Foreign Agent (FA): A router in the visited (foreign) network that assigns a Care-of Address (CoA), decapsulates tunneled packets from the HA, and delivers them to the mobile node.
* Agent Discovery: Foreign agents periodically broadcast Agent Advertisement messages. Arriving mobile nodes process these advertisements to discover their foreign network location and obtain a CoA.
* Registration: The mobile node sends a Mobile IP Registration Request to the FA, which relays it to the HA. The HA updates its mobility binding table, establishing an IP-in-IP tunnel to the FA CoA.

6.3 Impact of Wireless Links and Mobility on Transport Protocols

Standard TCP implementations assume that packet loss is caused primarily by network congestion (router buffer overflows). When a segment is lost, TCP halves its congestion window (CWND) and enters congestion avoidance, reducing transmission throughput.

The Wireless TCP Problem:

1. Bit Errors over Wireless: High BER on wireless links causes frame drops. Standard TCP misinterprets these losses as network congestion, inappropriately shrinking CWND and underutilizing available channel capacity.
2. Handover Delays: During inter-cell handover, packet forwarding pauses briefly while tunnels reconfigure. Standard TCP timers may expire (RTO timeout), triggering slow-start recovery.

Architectural Solutions:

1. Local Recovery (Link-Layer ARQ/HARQ): The wireless link layer uses local retransmissions (ARQ) and forward error correction (FEC) to correct bit errors locally before higher-layer TCP timers observe the loss.
2. TCP Sender Wireless Awareness: Transport protocols utilize explicit signaling (e.g., Explicit Congestion Notification [ECN] or Selective Acknowledgments [SACK]) to distinguish random bit-error losses from true router queue congestion.
3. Split-Connection Approaches (Indirect TCP / Split TCP): The end-to-end TCP connection is split into two independent connections at the wireless access point or proxy:
  * A standard TCP connection between the wired sender and the AP.
  * A specialized, loss-tolerant wireless transport protocol (or tuned TCP instance) between the AP and the mobile host.
  * Trade-off: Violates end-to-end TCP semantics (an ACK arrives at the sender before the data reaches the final mobile application), but improves throughput over lossy wireless hops.

7. Practice Examination & Quantitative Problem Set

Question 1: CDMA Code Orthogonality and Decoding

Suppose a 2-sender CDMA system uses M=4-bit chipping codes:

* Sender 1 Code: c^1 = (1, 1, -1, -1)
* Sender 2 Code: c^2 = (1, -1, -1, 1)

Part A: Verify that codes c^1 and c^2 are orthogonal. Part B: Sender 1 transmits data bit d^1 = 0 (represented as -1). Sender 2 transmits data bit d^2 = 1 (represented as +1). Compute the aggregate channel vector Z^*. Part C: Show step-by-step how the receiver uses vector Z^* and code c^1 to decode Sender 1's data bit.

Solution:

* Part A: Two codes are orthogonal if their inner product equals zero:

c^1 \cdot c^2 = (1)(1) + (1)(-1) + (-1)(-1) + (-1)(1) = 1 - 1 + 1 - 1 = \mathbf{0}

Codes c^1 and c^2 are orthogonal.

* Part B:
  * Sender 1 output: Z^1 = d^1 \cdot c^1 = (-1) \cdot (1, 1, -1, -1) = (-1, -1, 1, 1)
  * Sender 2 output: Z^2 = d^2 \cdot c^2 = (+1) \cdot (1, -1, -1, 1) = (1, -1, -1, 1)
  * Aggregate signal Z^* = Z^1 + Z^2 = (-1+1, -1-1, 1-1, 1+1) = \mathbf{(0, -2, 0, 2)}
* Part C: To decode Sender 1's bit:

d^1 = \frac{1}{M} \sum_{m=1}^4 Z^*_m \cdot c_m^1 = \frac{1}{4} \Big( (0)(1) + (-2)(1) + (0)(-1) + (2)(-1) \Big)

d^1 = \frac{1}{4} (0 - 2 + 0 - 2) = \frac{-4}{4} = -1 \implies \mathbf{\text{Bit } 0}

Sender 1's transmitted bit 0 is recovered.

Question 2: 802.11 RTS/CTS Transmission Delay

Consider an 802.11b station transmitting a 1,200-byte DATA frame. Assume the following parameters:

* Data Rate = 11 Mbps
* RTS Frame Size = 20 bytes
* CTS Frame Size = 14 bytes
* ACK Frame Size = 14 bytes
* DIFS = 50 \mus, SIFS = 10 \mus
* Ignore propagation delay and backoff counter.

Calculate the total time required to transmit the data frame using the RTS/CTS reservation sequence from the initial channel check to the final ACK receipt.

Solution:

1. Frame bit conversions:
  * RTS Size = 20 \text{ bytes} \times 8 = 160 \text{ bits}
  * CTS Size = 14 \text{ bytes} \times 8 = 112 \text{ bits}
  * DATA Size = 1200 \text{ bytes} \times 8 = 9,600 \text{ bits}
  * ACK Size = 14 \text{ bytes} \times 8 = 112 \text{ bits}
2. Transmission times (t = \text{bits} / 11 \text{ Mbps}):
  * t_{\text{RTS}} = 160 / 11,000,000 = 14.55 \ \mu\text{s}
  * t_{\text{CTS}} = 112 / 11,000,000 = 10.18 \ \mu\text{s}
  * t_{\text{DATA}} = 9600 / 11,000,000 = 872.73 \ \mu\text{s}
  * t_{\text{ACK}} = 112 / 11,000,000 = 10.18 \ \mu\text{s}
3. Total Delay Sequence:

\text{Total Time} = \text{DIFS} + t_{\text{RTS}} + \text{SIFS} + t_{\text{CTS}} + \text{SIFS} + t_{\text{DATA}} + \text{SIFS} + t_{\text{ACK}}

\text{Total Time} = 50 + 14.55 + 10 + 10.18 + 10 + 872.73 + 10 + 10.18 = \mathbf{987.64 \ \mu\text{s}}

Question 3: CSMA/CD vs. CSMA/CA Protocol Mechanics

Explain why Ethernet uses CSMA/CD while 802.11 WiFi uses CSMA/CA. Identify two distinct reasons why collision detection is impractical over wireless radio links.

Solution:

1. Power Differential: In wired Ethernet, an adapter can transmit and listen simultaneously because transmitted and received signal strengths on the wire are comparable. In wireless links, local transmit power is vastly higher than incoming receive power from a remote node. The local transmitter's antenna saturates its receiver circuit, preventing it from detecting a weak interfering signal.
2. Hidden Terminal Problem: Even if a wireless station could detect collisions at its own antenna, it cannot detect collisions occurring at the remote receiving base station caused by a hidden terminal. Therefore, 802.11 relies on collision avoidance (CSMA/CA) via random backoff and link-layer ACKs.

Question 4: 802.11 MAC Addressing

In an 802.11 infrastructure BSS, a wireless host H1 transmits an IP packet to a fixed server S attached to a wired network router interface R1.

1. What address is placed in Address 1?
2. What address is placed in Address 2?
3. What address is placed in Address 3?

Solution:

1. Address 1 (Receiver MAC): MAC address of the destination Access Point (AP1).
2. Address 2 (Transmitter MAC): MAC address of the sending wireless station (H1).
3. Address 3 (Router Interface MAC): MAC address of the destination router interface (R1).

Question 5: LTE Resource Slot Calculation

An LTE base station operates a 20 MHz channel using downstream OFDM. The channel provides 100 subcarriers.

* A single time slot lasts 0.5 ms.
* Each subcarrier in a slot carries 72 symbols.
* Modulation used is QAM64 (6 bits per symbol).

Calculate the theoretical maximum physical-layer raw bitrate achievable if a single User Equipment (UE) is allocated all 100 subcarriers across all time slots.

Solution:

1. Bits per subcarrier per 0.5 ms slot = 72 \text{ symbols} \times 6 \text{ bits/symbol} = 432 \text{ bits}.
2. Bits across all 100 subcarriers per 0.5 ms slot = 432 \text{ bits} \times 100 = 43,200 \text{ bits}.
3. Slots per second = 1 \text{ second} / 0.0005 \text{ seconds} = 2,000 \text{ slots/sec}.
4. Total Bitrate = 43,200 \text{ bits/slot} \times 2,000 \text{ slots/sec} = 86,400,000 \text{ bps} = \mathbf{86.4 \text{ Mbps}}.

Question 6: 4G LTE System Component Identification

Match each LTE system element to its primary functional description:

1. HSS
2. MME
3. P-GW
4. eNode-B

A. User-plane anchor that assigns NAT IP addresses and connects the EPC to the Internet. B. Radio access node managing radio channel scheduling and LTE cell coverage. C. Database in the home network storing subscriber secret keys and profile privileges. D. Visited network control-plane node executing AKA authentication, session setup, and paging.

Solution:

* 1 \to C (HSS stores subscriber secrets and profile data).
* 2 \to D (MME coordinates control plane, authentication, and mobility paging).
* 3 \to A (P-GW is the gateway router offering NAT IP addresses to the Internet).
* 4 \to B (eNode-B is the base station managing radio channels).

Question 7: Triangle Routing Delay Analysis

A mobile host is roaming in a visited network in Tokyo. Its home network is located in New York. A correspondent in Tokyo wants to send data to the mobile host.

* Propagation delay between Tokyo and New York = 100 ms.
* Propagation delay within Tokyo = 2 ms.
* Processing and transmission delays are negligible.

Calculate the single-packet one-way propagation delay under:

1. Indirect Routing (without local breakout).
2. Direct Routing.

Solution:

1. Indirect Routing:
  * Packet travels from Correspondent (Tokyo) \to Home Gateway (New York) = 100 ms.
  * Encapsulated packet travels from Home Gateway (New York) \to Visited Gateway (Tokyo) = 100 ms.
  * Packet travels from Visited Gateway (Tokyo) \to Mobile Host (Tokyo) = 2 ms.
  * Total One-Way Delay = 100 + 100 + 2 = \mathbf{202 \text{ ms}}.
2. Direct Routing:
  * Correspondent queries Home HSS (New York) and receives response: 100 + 100 = 200 \text{ ms} (initial setup).
  * Once location is known, packet travels directly from Correspondent (Tokyo) \to Visited Gateway (Tokyo) \to Mobile Host (Tokyo) = 2 \text{ ms}.
  * Data Packet Propagation Delay = \mathbf{2 \text{ ms}} (suffers 200 ms query latency only for the initial lookup).

Question 8: 5G Spectrum and Service Classes

Compare 5G FR1 (sub-6 GHz) and 5G FR2 (mmWave) in terms of coverage distance, wall penetration, and bandwidth capacity. Briefly state which 5G service pillar (eMBB, URLLC, or mMTC) benefits most from mmWave.

Solution:

* 5G FR1 (Sub-6 GHz): Long coverage range (1–5 km), high wall penetration, moderate bandwidth capacity. Ideal for broad coverage and baseline mobility.
* 5G FR2 (mmWave): Short coverage range (100–300 m), poor wall penetration (blocked by glass/foliage), massive bandwidth capacity (multi-gigabit speeds).
* Benefiting Pillar: eMBB (Enhanced Mobile Broadband) benefits most from mmWave due to its high bandwidth requirements for ultra-high-definition video and immersive media.

Question 9: Indirect TCP Trade-offs

Explain how Split-Connection TCP (Indirect TCP) improves data throughput over lossy wireless links. What fundamental transport-layer property does Indirect TCP violate?

Solution:

* Throughput Improvement: Indirect TCP splits the TCP connection into a wired segment and a wireless segment at the access point/proxy. Bit-error losses on the wireless hop trigger local TCP retransmissions from the AP buffer, preventing the wired sender from halving its CWND needlessly.
* Property Violated: Violates end-to-end TCP semantics. The AP sends a TCP ACK back to the sender as soon as it receives bytes, before those bytes are successfully delivered over the air to the destination host application. If the AP fails before air transmission completes, the sender assumes data was safely received when it was actually lost.

Question 10: 4G LTE Handover Tunnel Switching

During an LTE handover from Source eNode-B to Target eNode-B, how are in-flight packets handled to prevent packet loss before the Serving Gateway updates its GTP tunnel path?

Solution:

1. The Source eNode-B stops transmitting packets over the air to the UE.
2. The Source eNode-B establishes an X2 interface data forwarding tunnel to the Target eNode-B.
3. Undelivered packets buffered at the Source eNode-B, as well as new packets arriving from the S-GW before tunnel path switching completes, are forwarded over the X2 tunnel to the Target eNode-B.
4. The Target eNode-B buffers these forwarded packets and delivers them in order to the UE as soon as the radio link is active, preventing packet drops during control-plane path switching.

5. Comprehensive Technical Glossary

* Adaptive Modulation and Coding (AMC): A physical-layer mechanism that dynamically adjusts signal modulation order (e.g., QAM256 to QPSK) based on real-time SNR measurements to optimize bitrates while controlling BER.
* Agent Advertisement: A Mobile IP control message periodically broadcast by Foreign Agents to advertise mobility services and care-of addresses to roaming mobile nodes.
* Authentication and Key Agreement (AKA): A mutual cryptographic challenge-response protocol executed between a mobile user equipment (UE) and the cellular network (MME/HSS) to authenticate identity and derive session encryption keys.
* Base Station: A fixed radio transceiver in a wireless or cellular network that manages wireless links for hosts within its geographic coverage area and relays traffic to a wired backbone.
* Basic Service Set (BSS): The fundamental building block of an IEEE 802.11 wireless LAN, consisting of an Access Point (AP) and associated wireless stations operating in infrastructure mode.
* Beacon Frame: A management frame periodically broadcast by an IEEE 802.11 AP containing its SSID, MAC address, TIM, and operational parameters to allow nearby stations to discover and associate with the network.
* Beamforming: A spatial signal processing technique used in multi-antenna array systems (MIMO) that adjusts phase and amplitude to direct radio energy toward a specific receiver.
* Care-of Address (CoA): A temporary network-layer IP address assigned to a roaming mobile node in a visited/foreign network, used as a tunnel endpoint by home mobility agents.
* Carrier Sense Multiple Access with Collision Avoidance (CSMA/CA): A random access MAC protocol used in IEEE 802.11 WiFi where stations sense the channel, wait for DIFS, execute random backoff countdowns, and rely on explicit ACKs and RTS/CTS reservations to avoid collisions.
* Chipping Code: A high-rate binary vector sequence assigned to a host in CDMA systems to encode data bits prior to transmission over a shared radio channel.
* Code Division Multiple Access (CDMA): A channel partitioning MAC protocol that assigns orthogonal chipping codes to senders, allowing multiple hosts to transmit simultaneously over the same frequency band without mutual interference.
* Discontinuous Reception (DRX): A power-saving state in 4G LTE/5G where the mobile device sleeps for scheduled intervals, waking up periodically to check for downstream control messages.
* Distributed Inter-frame Space (DIFS): The mandatory minimum idle time an IEEE 802.11 station must sense on the channel before initiating frame transmission or resuming backoff countdown.
* Enhanced Packet Core (EPC): The all-IP core architecture of 4G LTE networks comprising the MME, HSS, S-GW, and P-GW nodes.
* eNode-B (Evolved Node B): The 4G LTE cellular base station responsible for managing radio resources, channel scheduling, and user-plane IP tunneling.
* Frequency Hopping Spread Spectrum (FHSS): A physical-layer transmission technique (used in Bluetooth) that rapidly switches carrier frequencies across time slots in a pseudo-random sequence to resist narrow-band interference.
* GPRS Tunneling Protocol (GTP-U): An IP encapsulation protocol used in cellular core networks (4G EPC / 5G) to transport user data plane packets between base stations and gateway nodes across TEID-identified tunnels.
* Handover / Handoff: The process of transferring an active wireless communication session and user data path from one base station or access point to another as a mobile node moves between cells.
* Hidden Terminal Problem: A multiple access impairment where two wireless senders are physically shielded or separated beyond carrier sensing range from each other, but both transmit to a common receiver, causing uncoordinated signal collisions at the receiver.
* Home Subscriber Server (HSS): A central control database in a cellular subscriber's home network that maintains user profiles, authorization privileges, and secret authentication keys.
* International Mobile Subscriber Identity (IMSI): A globally unique 64-bit identification number assigned to a cellular subscriber and stored securely inside their SIM card.
* Local Breakout: A cellular routing optimization where a roaming mobile host's outgoing data traffic is routed directly from the visited network gateway to the public Internet, bypassing indirect tunneling through the home network.
* Massive MIMO: An advanced antenna technology utilizing tens or hundreds of antenna elements at a base station to serve multiple users simultaneously over the same frequency channel via spatial multiplexing.
* Mobility Management Entity (MME): The primary control-plane node in a 4G LTE network responsible for user authentication, tracking area management, signaling, and data path tunnel creation.
* Network Allocation Vector (NAV): An internal timer maintained by IEEE 802.11 stations that tracks reserved channel durations indicated in overheard RTS/CTS frames, deferring transmissions until the NAV reaches zero.
* Orthogonal Frequency Division Multiplexing (OFDM): A digital multi-carrier modulation technique that divides a wireless channel into multiple closely spaced orthogonal subcarriers, widely used in 802.11a/g/n/ac/ax and 4G/5G radio interfaces.
* Packet Data Convergence Protocol (PDCP): The uppermost link sublayer in the 4G LTE user-plane protocol stack responsible for IP header compression (ROHC) and AES payload encryption.
* Packet Data Network Gateway (P-GW): The boundary gateway router in a 4G LTE core network that connects the cellular carrier network to the public Internet and allocates NAT IP addresses to mobile nodes.
* Paging: A control-plane process where cellular base stations broadcast wake-up signals across a tracking area to locate an idle/sleeping mobile device when downstream packets arrive for it.
* Piconet: An ad-hoc personal area network formed by Bluetooth devices containing one master node and up to seven active client nodes.
* Radio Link Control (RLC): A link sublayer in the LTE stack that handles fragmentation, reassembly, and link-layer reliable data transfer via ARQ.
* Request to Send / Clear to Send (RTS/CTS): An optional IEEE 802.11 channel reservation handshake used to clear the channel around both sender and receiver, mitigating hidden terminal collisions for long data frames.
* Serving Gateway (S-GW): The user data-plane mobility anchor router in a 4G LTE network that relays tunneled user IP packets between base stations and the P-GW.
* Short Inter-frame Space (SIFS): A short mandatory delay interval in IEEE 802.11 used between immediate high-priority frame exchanges (such as DATA to ACK, or RTS to CTS).
* Signal-to-Noise Ratio (SNR): The ratio of received signal power to background noise power, typically expressed in decibels (dB), governing achievable modulation order and bit error rates.
* Split-Connection TCP: An architecture that breaks an end-to-end TCP connection into separate wired and wireless segments at an intermediate proxy/AP to prevent wireless bit-error drops from reducing wired congestion window sizes.
* Triangle Routing Problem: An efficiency penalty in indirect routing architectures where packets sent to a roaming mobile node must travel to its home gateway first before being tunneled to the visited network, increasing latency when sender and receiver are co-located.
* User Equipment (UE): The official 3GPP designation for a 4G/5G mobile subscriber device (e.g., smartphone, tablet, IoT modem).
* User Plane Function (UPF): The decoupled, high-performance packet forwarding node in a 5G Core network responsible for data processing, routing, and inspection at the network edge.
