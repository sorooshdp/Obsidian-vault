---
created: 2026-11-10 14:03
updated: 2026-11-10 14:03
type: chapter
status: seed
topic:
tags:
  - cloud
aliases: []
---

Here is a summary of the important notes from **Chapter 1: Introduction** of *Cloud Computing: Theory and Practice* (3rd Edition):

### **1. Definition & Core Philosophy of Cloud Computing**
* **Utility Computing Model:** Cloud computing concentrates hardware and software resources into large data centers, allowing users to consume compute, storage, and networking as a utility on a pay-as-you-go basis.
* **NIST Definition & 5 Essential Attributes:** The National Institute of Standards and Technology defines cloud computing by five key characteristics: **on-demand self-service**, **broad network access**, **resource pooling**, **rapid elasticity**, and **measured service**.
* **Shift to Commodity Hardware:** Unlike early supercomputing paradigms that relied on expensive, custom-built hardware, modern computer clouds are built using **off-the-shelf commodity components**.
* **Paradigm Shift:** Shift from local computing to network-centric computing and content, where control over software and data is largely handed to Cloud Service Providers (CSPs).

---

### **2. Deployment Models & Economic Drivers**
* **4 Deployment Models:**
  1. **Public Cloud:** Infrastructure available to the general public or large industry groups, owned by a CSP.
  2. **Private Cloud:** Infrastructure operated exclusively for a single organization.
  3. **Hybrid Cloud:** A composition of two or more distinct clouds (e.g., public and private) bound by standardized technology.
  4. **Community Cloud:** Infrastructure shared by several organizations with shared concerns or compliance needs.
* **Economies of Scale:** Data centers with over 50,000 servers experience a **5 to 7 times reduction in resource and energy consumption** compared to medium-sized data centers (~1,000 servers).
* **Cost Efficiency:** Achieved through **resource multiplexing** (sharing resources among workloads with unsynchronized peak demands).

---

### **3. Energy Use & Ecological Impact (Section 1.2)**
* Data centers and IT infrastructure contribute significantly to global greenhouse-gas emissions and carbon footprints.
* Reducing the carbon footprint requires periodic hardware refreshes to adopt energy-efficient technologies, alongside energy-optimized resource management software.

---

### **4. Ethical & Governance Issues (Section 1.3)**
* **Relinquishing Control:** Users delegate data and execution control to third-party providers.
* **De-perimeterization & Fuzzy Accountability:** Because data spans multiple systems and networks, establishing clear boundaries of liability and responsibility in the event of failures or data leaks is difficult.
* **Vendor Lock-In & Market Dominance:** Risks associated with customer dependency on proprietary cloud platforms and market concentration among a few dominant CSPs.

---

### **5. Factors Affecting Cloud Service Availability (Section 1.4)**
* Availability is threatened by hardware/power failures, malicious cyberattacks (such as DDoS/DNS attacks), and unforeseen interactions between complex subsystems under extreme conditions.
* **Geographic Dispersion:** Distributing resources across geographically separated data centers reduces the probability of catastrophic failures while lowering communication latency and energy costs.

---

### **6. Network-Centric Computing & Content (Section 1.5)**
* **Key Characteristics:**
  * **Data-Intensive Workloads:** AI, Machine Learning, sensors, and simulations require processing massive amounts of data.
  * **Network-Intensive:** Low latency and high bandwidth are necessary for streaming and interactive computation.
  * **Thin Clients:** Users interact with massive cloud systems via lightweight devices like smartphones and laptops.
* **Major Obstacles:**
  * **Data Transfer Bottlenecks:** Moving large datasets (e.g., Terabytes/Petabytes) over low-bandwidth networks can take days; sending physical media via courier can sometimes be faster.
  * **Performance Unpredictability:** Multi-tenant resource sharing makes guaranteed Quality of Service (QoS) and performance isolation difficult.

## References
*Cloud Computing: Theory and Practice* (3rd Edition)