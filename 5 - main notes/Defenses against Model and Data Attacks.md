2025/06/05  -  20:32
status: 

Tags: [[Federated learning]] [[AI]] [[machine learning]] [[security]] 

Detect and Remove

**Detect and Remove** is a server-side defense strategy in federated learning (FL) that aims to detect and eliminate malicious updates before they influence the global model. This method relies on monitoring client submissions and identifying abnormal patterns through techniques like statistical analysis, machine-learning-based anomaly detection, or comparison to trusted baselines. Updates flagged as suspicious are excluded from aggregation to maintain model integrity.

Several advanced frameworks implement this approach. The **`LoMar` algorithm** [82] uses a two-phase process: first scoring each client update’s “maliciousness” via kernel density estimation, and then applying a threshold derived from statistical bounds to classify updates. **`FederatedReverse`** [83] introduces a reverse engineering approach, where each client generates triggers to expose backdoor patterns; the server aggregates and analyzes them using outlier detection, and retrains clients to “unlearn” the backdoor.

The **Dynamic Defense Against Byzantine Attacks (`DDaBA`)** [84] uses an IOWA operator to assign weights to client updates based on their validation performance, filtering adversarial clients via statistical outlier detection. Similarly, **`SecFedNIDS`** [85] provides a two-layered defense for FL-based network intrusion detection systems, using gradient-based SOS anomaly detection on the model level and relevance-propagation-based filtering at the data level.

In [86], **two targeted defenses** are proposed. The first, PASM, detects malicious updates by comparing their cosine similarity with an interim model and excluding those above a threshold. The second, ACCDR, uses a data-free `TrojanNet` detector to compare neuron activations and flag backdoor-affected models.

A density-based anomaly detection framework [87] evaluates how isolated each client model is compared to its neighbors. Based on this, the system assigns anomaly scores and adjusts aggregation weights accordingly. **BAFFLE** [61] adds a feedback loop where a client subset validates each global model round; models flagged by a quorum vote for suspicious error patterns are rejected.

In [88], a KPCA-based defense reduces dimensionality in client updates and detects deviations from the global update. Detected outliers may be ignored or blacklisted, with optional clustering applied. It also supports real-time detection during training.

Lastly, [89] proposes a generative classifier using Gaussian Discriminant Analysis on pre-trained soft-max outputs. It assesses sample likelihoods via `Mahalanobis` distance and uses noise calibration to enhance detection. This method is robust under noisy conditions, minimal tuning, and supports out-of-distribution and class-incremental learning scenarios.


# References
