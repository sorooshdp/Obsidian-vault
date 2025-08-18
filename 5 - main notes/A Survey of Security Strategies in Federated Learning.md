2025/06/05  -  19:29
status: 

Tags: [[Federated learning]] [[AI]] 

### Types of FL

Categories Based on Data Partitioning

Horizontal Federated Learning (HFL): Horizontal FL, also known as sample-based FL, occurs when datasets from different sources share the same feature space but differ in samples [35–38]. It is applicable when organizations or devices possess data of the same type (i.e., having the same features) but of different users.
- Healthcare: Collaborative training using patient data from different hospitals. 
- Finance: Banks combining transaction records to improve fraud detection models. 
- Mobile Devices: Enhancing predictive text models using data from different smartphones.

Vertical Federated Learning (VFL):
Vertical Federated Learning, also known as feature-based FL, involves datasets that have different feature spaces but share the same sample ID space.

Federated Transfer Learning (FTL) Federated Transfer Learning is used when datasets have different feature spaces and only partially overlap in sample ID space [44].

Categories Based on System Architecture 

Centralized Federated Learning: In Centralized Federated Learning, a central server coordinates the training process. Clients train local models on their data and send updates to the central server, which aggregates these updates to form a global model.

Decentralized Federated: Learning Decentralized FL eliminates the central server, with clients directly communicating and sharing model updates with each other [46,47].

Categories Based on Operational Strategies
 Cross-Silo Federated Learning: Cross-Silo FL typically involves a small number of data silos (e.g., organizations or institutions) with relatively large datasets [52,53].
 Cross-Device FL Cross-device FL involves a large number of devices (e.g., smartphones and IoT devices) with relatively small local datasets [45].

### Types of Attacks in FL

Data Attack 
Data Poisoning: Malicious clients inject false or misleading data into their local datasets [17]. This can skew the model’s training process, leading to a degraded or biased global model [57].
Label Flipping: A specific type of data poisoning where the labels of the training data are intentionally flipped [59,60].
Backdoor Attacks: The attacker modifies the training data to introduce a backdoor into the model [61].

Model Attack 
In model attack, malicious clients intentionally manipulate their local model updates before sending them to the central server [62,63]. The severity of the privacy breach depends on the extent to which the attacker can infer details about the training data.

Privacy Attacks 
Privacy attacks in FL are aimed at extracting sensitive information from the model updates or the aggregated model itself, thereby compromising the confidentiality of the training data [65].

### **Defense Frameworks in Federated Learning (FL)**

Due to the wide range of attacks targeting FL—including data poisoning, model poisoning, and privacy breaches—a variety of tailored defense mechanisms are necessary. A **one-size-fits-all approach is ineffective**, so layered and phase-specific defenses are preferred.

 **Key Defense Types:**
1. **Data and Model Defense:**
    - **Data poisoning:** Mitigated by data validation and anomaly detection.
    - **Model poisoning:** Addressed through secure aggregation and Byzantine fault tolerance.
2. **Privacy Defense:**
    - Techniques like **differential privacy**, **homomorphic encryption**, and **secure multi-party computation** prevent inference attacks.

 **Defense Phases in FL:**
- **Pre-aggregation:** Detect and filter malicious updates early.
- **In-aggregation:** Use robust aggregation methods to reduce the impact of adversarial data.
- **Post-aggregation:** Repair compromised models after training.



# References
