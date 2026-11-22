---
created: 2026-11-12 18:37
updated: 2026-11-12 18:37
type: chapter
status: seed
topic:
tags:
  - cloud
aliases: []
---
Here is a study guide for **Chapter 3: Parallel Processing and Distributed Computing** from *Cloud Computing: Theory and Practice*:

---

## 1. Core Architectural Concepts & Physical Limits

* **Subsystems & Busses:** Digital computers consist of three main subsystems: the **CPU** (transforms data), **Memory** (stores data), and **I/O** (handles communication). These communicate via internal computer busses carrying instructions/control, addresses, and data.
* **The Bandwidth-Latency Gap:** Hiding latency is a primary goal of computer architecture. Over three decades, CPU bandwidth improved by 10,000–25,000$\times$ while memory and disk bandwidth grew only 300–1,200$\times$. Processor latency improved 30–80$\times$ compared to just 6–8$\times$ for memory and storage.
* **Instruction Set Architecture (ISA) & Control Flow:** CPU design is divided into **CISC** (Complex Instruction Set Computer) and **RISC** (Reduced Instruction Set Computer) architectures. Most general-purpose computing relies on the von Neumann **control flow model**, whereas data-flow models execute instructions as soon as input data becomes available.
* **Limits of Clock Frequency Scaling:** Total application execution time $T$ is governed by:

  $$T = N \times \text{CPI} \times R$$

  where $N$ is instruction count, $\text{CPI}$ is cycles per instruction, and $R$ is clock cycle time.
* **Dennard's Law & Thermal Constraints:** Clock rates peaked around 3.2–4.0 GHz due to power dissipation ($P \propto f^2 \text{ or } f^3$). Because transistor packing scales with volume ($V = \frac{4}{3}\pi r^3$) while heat dissipation is limited by surface area ($S = 4\pi r^2$), physical limits forced a shift toward **multicore processors** and **Domain-Specific Architectures (DSA)**.

---

## 2. Memory Hierarchy, Cache Optimizations, & Accelerators

* **Memory Hierarchy:** Arranged by latency, bandwidth, and cost:
  * **Registers:** SRAM, $<300\text{ ps}$ latency, $200+\text{ GB/s}$ bandwidth.
  * **L1 / L2 / L3 Cache:** On-chip SRAM, $0.3\text{–}2+\text{ ns}$ latency.
  * **Main Memory:** DRAM, $50+\text{ ns}$ latency, $2+\text{ GB/s}$ bandwidth.
  * **Secondary Storage:** Magnetic Hard Disk / Flash, $10\text{ ms}$ latency, $10+\text{ MB/s}$ bandwidth.
* **Locality & Cache Misses:** Leverages **spatial locality** (referencing nearby data blocks) and **temporal locality** (reusing recently accessed code/data, forming a program's *working set*). Miss types include **compulsory**, **capacity**, and **conflict**. Modified cache blocks are handled via **write-through** or **write-back** strategies.

  $$\text{AMAT} = \text{Hit Time} + \text{Miss Rate} \times \text{Miss Penalty}$$
* **SIMD & The Roofline Performance Model:**
  * **SIMD (Single Instruction Multiple Data):** Fetches one instruction for multiple data operations, providing higher energy efficiency and speedup than MIMD.
  * **Arithmetic Intensity ($\text{ArI}$):** Defined as floating-point operations per byte of memory accessed ($\text{ArI} = \frac{\text{FLOPs}}{\text{Bytes}}$).
  * **Roofline Model:** Illustrates performance limits: at low $\text{ArI}$, application speed is bounded by memory bandwidth; at high $\text{ArI}$, performance is bounded by peak processor GFLOPS.
* **GPUs, TPUs, and Systems-on-a-Chip (SoC):**
  * **GPUs:** Adapted from graphics geometric engines for data-intensive parallel matrix computations.
  * **TPUs (Tensor Processing Units):** Custom ASICs designed specifically for machine learning inference (Deep Neural Networks).
  * **SoCs:** Highly integrated chips built with advanced processes (e.g., 5nm) combining CPU, GPU, memory, and bus fabrics (AMBA/NoC) to optimize performance-per-watt for mobile and edge computing.

---

## 3. Parallelism Types & Performance Laws

* **Types of Parallelism:**
  * **Data-Level Parallelism:** Applying identical operations concurrently across segmented data sets (e.g., MapReduce).
  * **Task- & Thread-Level Parallelism:** Scheduling independent tasks or fine-grained execution threads across multiple processing units.
* **Amdahl's Law (Fixed Problem Size):** Predicts maximum speedup $S$ when a sequential fraction $\alpha$ cannot be parallelized:

  $$S(N) = \frac{1}{\alpha + \frac{1-\alpha}{N}} \xrightarrow{N \to \infty} \frac{1}{\alpha}$$
* **Gustafson's Law (Scaled Speedup):** Evaluates speedup when the problem size grows with the number of parallel processors $N$:

  $$S(N) = N - \alpha(N - 1) = \alpha + N(1 - \alpha)$$
* **Multicore Processor Speedup Models (BCE):** Evaluated using **Basic Core Equivalent (BCE)** units:
  * **Symmetric Cores:** All cores are identical on-chip ($n/r$ cores of $r$ BCEs each).
  * **Asymmetric Cores:** Combines one powerful core (e.g., $r$ BCEs) for sequential phases with many smaller 1-BCE cores for parallel phases; provides higher overall speedup than purely symmetric configurations.
  * **Dynamic Cores:** Reconfigures core resources dynamically between sequential and parallel modes.

---

## 4. Distributed Systems Principles & Modularity

* **Distributed System Definition:** A collection of autonomous, heterogeneous computers connected via a network and unified by **middleware** so users perceive it as a single integrated facility.
* **8 Key Transparencies:** Access, Location, Concurrency, Replication, Failure, Migration, Performance, and Scaling transparencies.
* **Modularity & Enforced Modularity:**
  * Modularity separates concerns and supports specialization.
  * **Enforced Modularity:** Implemented via the **client-server paradigm**, forcing modules to communicate exclusively through message passing across strict boundaries. Stateless servers enhance system fault tolerance because failed servers can restart without corrupting client state.
* **Communication Abstractions:** Includes Remote Procedure Calls (**RPC**), Round-Trip Time (**RTT**) latency considerations, and asynchronous **Publish-Subscribe** event queues.
* **Grid vs. Cloud Computing:** Grid systems were heterogeneous, dispersed across multiple administrative domains, and focused on scientific computing. Clouds rely on single administrative domains, offer homogeneous software stacks, and focus on enterprise computing.

---

## 5. Composability Bounds, CAP Theorem, & Blockchain

* **Composability & Scaling Limits:** Software liberates system composition from strict physical hardware limits, but system algorithms must adapt across physical, spatial, and societal scale dimensions.
* **CAP Theorem (Brewer's Theorem):** A distributed data store can simultaneously provide at most two of three properties:
  1. **Consistency (C):** Every read receives the most recent write or an error.
  2. **Availability (A):** Every request receives a non-error response.
  3. **Partition Tolerance (P):** The system operates despite network message loss or delays.
* **PACELC Theorem (Abadi's Extension):** Extends CAP to non-partitioned normal operations:
  * If there is a **P**artition, trade off **A**vailability vs. **C**onsistency.
  * **E**lse (normal operation), trade off **L**atency vs. **C**onsistency.
* **Blockchain Technology:** A decentralized, distributed digital ledger managed over a P2P network. Sacrifices immediate consistency for availability and partition tolerance, achieving eventual consensus via distributed block validation over time.

## References