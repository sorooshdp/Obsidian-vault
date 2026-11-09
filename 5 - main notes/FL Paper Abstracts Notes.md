---
created: 2026/01/18  -  23:23
updated: 2026-11-09 21:05
type: note
status: seed
topic:
tags:
  - federated_learning
aliases: []
---

## Centralized Aggregations
### Communication-Efficient Learning of Deep Networks from Decentralized Data (Centralized Aggregation)
Appearing in Proceedings of the 20th International Conference on Artificial Intelligence and Statistics (AISTATS) 2017, Fort Laud erdale, Flordia, USA. JMLR: W&CP volume 54
This is the main paper which presents a practical method for the federated learning of deep networks based on iterative model averaging, and conduct an extensive empirical evaluation, considering five different model architectures and four datasets.

### Shielding Federated Learning: Robust Aggregation with Adaptive Client Selection (Centralized Aggregation)
Proceedings of the Thirty-First International Joint Conference on Artificial Intelligence (IJCAI-22).
paper introduces a new method, MAB-RFL for robust aggregation against `sybil` attacks, an active client selection strategy. algorithm provided but no source code.

### RFLPA: A Robust Federated Learning Framework against Poisoning Attacks with Secure Aggregation (Centralized Aggregation)
38th Conference on Neural Information Processing Systems (NeurIPS 2024).
Introduces an aggregation algorithm compatible with secure aggregation against poisoning attacks, RFLPA, computes the cosine similarity between local updates and server updates to conduct robust aggregation, a secret sharing (Shamir) method.

### FEDBN: FEDERATED LEARNING ON NON-IID FEATURES VIA LOCAL BATCH NORMALIZATION (Central Aggregation)
Published as a conference paper at ICLR 2021 
Code is available at https://github.com/med-air/FedBN
this work, propose an effective method that uses local batch normalization to alleviate the feature shift before averaging models. The resulting scheme, called `FedBN` outperforms both classical `FedAvg`, as well as the state-of-the-art for `non-iid` data (`FedProx`) on extensive experiments.

### Defending against the Label-flipping Attack in Federated Learning
arXiv:2207.01982v1 [cs.CR] 5 Jul 2022
Code available at https://github.com/anonymized1/LF-Fighter
This paper propose a novel defense that first dynamically extracts those gradients from the peers’ local updates, and then clusters the extracted gradients, analyzes the resulting clusters and filters out potential bad updates before model aggregation.

### Machine Learning with Adversaries: Byzantine Tolerant Gradient Descent 
31st Conference on Neural Information Processing Systems (NIPS 2017), Long Beach, CA, USA
paper propose Krum, an aggregation rule that satisfies the resilience property, which paper argue is the first provably Byzantine-resilient algorithm for distributed SGD.

### Median-Krum: A Joint Distance-Statistical Based Byzantine-Robust Algorithm in Federated Learning
MobiWac ’23, October 30-November 3 2023, Montreal, QC, Canada 
©2023
ACM ISBN 979-8-4007-0367-6/23/10...$15.00 https://doi.org/10.1145/3616390.3618283
Median-Krum is proposed as a new aggregation algorithm whose validity is observed on a set of simulations that recreate realistic scenarios, in the absence and presence of Byzantine adversaries. It combines the Distance-based Krum approach with the Statistical strategy of median based aggregation algorithm.

### Mitigating Sybils in Federated Learning Poisoning
arXiv:1808.04866v5 [cs.LG] 15 Jul 2020
Paper describes `FoolsGold`, a novel defense to this problem that identifies poisoning sybils based on the diversity of client updates in the distributed learning process. Unlike prior work, the system does not bound the expected number of attackers, requires no auxiliary information outside of the learning process, and makes fewer assumptions about clients and their data.

### Robust Aggregation for Federated Learning
arXiv:1912.13445v2 [stat.ML] 17 Jan 2022
present a robust aggregation approach to make federated learning robust to settings when a fraction of the devices may be sending corrupted updates to the server. The approach relies on a robust aggregation oracle based on the geometric median, which returns a robust aggregate using a constant number of iterations of a regular non-robust averaging oracle.

### SCAFFOLD: Stochastic Controlled Averaging for Federated Learning
Proceedings of the 37th International Conference on Machine Learning, Online, PMLR 119, 2020. Copyright 2020 by the author(s).
Paper proposes a new algorithm (SCAFFOLD) which uses control variates (variance reduction) to correct for the ‘client-drift’ in its local updates. Paper proves that SCAFFOLD re quires significantly fewer communication rounds and is not affected by data heterogeneity or client sampling.

### The Hidden Vulnerability of Distributed Learning in Byzantium
Proceedings of the 35th International Conference on Machine Learning, Stockholm, Sweden, PMLR 80, 2018. Copyright 2018 by the author(s)
Paper show that convergence is not enough. In high dimension d ≫ 1, an adversary can build on the loss function’s non–convexity to make SGD converge to ineffective models. Paper introduce `Bulyan`, and prove it significantly reduces the at tacker’s leeway to a narrow O(1/√d) bound.

## Frameworks

### Model-Contrastive Federated Learning (Centralized Aggregation)
arXiv:2103.16257v1 [cs.LG] 30 Mar 2021 - CODE AVAILABLE - https://github.com/QinbinLi/MOON
In this paper, we propose MOON: model contrastive federated learning. MOON is a simple and effective federated learning framework. The key idea of MOON is to utilize the similarity between model representations to correct the local training of individual parties, i.e., conducting contrastive learning in model-level.

### `FedHealth`: A Federated Transfer Learning Framework for Wearable Healthcare (Framework)
DOI 10.1109/MIS.2020.2988604, IEEE Intelligent systems 2020
this paper, propose `FedHealth`, the first federated transfer learning framework for wearable healthcare to tackle these challenges. `FedHealth` performs data aggregation through federated learning, and then builds relatively personalized models by transfer learning. 
### FEDERATED OPTIMIZATION IN HETEROGENEOUS NETWORKS (Framework)
Proceedings of the 3rd `MLSys` Conference, Austin, TX, USA, 2020.
In this work, we introduce a framework, `FedProx`, to tackle heterogeneity in federated networks. `FedProx` can be viewed as a generalization and re-parametrization of `FedAvg`, the current state-of-the-art method for federated learning. in highly heterogeneous settings, `FedProx` demonstrates significantly more stable and accurate convergence behavior relative to `FedAvg`—improving absolute test accuracy by 22% on average.
### Federated learning model with dynamic scoring-based client selection for diabetes diagnosis (Framework)
Knowledge-Based Systems journal 2025
this study proposes a dynamic client selection mechanism based on a scoring system that evaluates clients based on three key parameters: accuracy, loss, and execution time. We propose a scoring-based framework for adaptive client selection in federated learning (FL) and implement it in an ML-driven diabetes detection system.
### FEDERATED LEARNING BASED ON DYNAMIC REGULARIZATION (Framework)
arXiv:2111.04263v2 [cs.LG] 9 Nov 2021
This paper propose a novel federated learning method for distributively training neural network models, named `FedDyn`, where the server orchestrates cooperation between a subset of randomly chosen devices in each round.
### Exploiting Shared Representations for Personalized Federated Learning (Framework)
Proceedings of the 38th International Conference on Machine Learning, PMLR 139, 2021.
The paper propose a novel federated learning framework and algorithm named `FedRep` for learning a shared data representation across clients and unique local heads for each client.
### Clustered Federated Learning: Model-Agnostic Distributed Multi-Task Optimization under Privacy Constraints (Framework)
arXiv:1910.01991v1 [cs.LG] 4 Oct 2019
This paper present Clustered Federated Learning (CFL), a novel Federated Multi-Task Learning (FMTL) framework, which exploits geometric properties of the FL loss surface, to group the client population into clusters with jointly trainable data distributions.
### CLIENT SELECTION IN FEDERATED LEARNING: CONVERGENCE ANALYSIS AND POWER-OF-CHOICE SELECTION STRATEGIES (Framework)
Under review as a conference paper at ICLR 2021
The first convergence analysis of federated optimization for biased client selection strategies, and quantify how the selection skew affects convergence speed is presented. Paper propose POWER-OF-CHOICE, a communication- and computation-efficient client selection framework that can flexibly span the trade-off between convergence speed and solution bias. They also propose an extension of POWER-OF-CHOICE that is able to maintain convergence speed improvement while diminishing the selection skew.
### Diverse Client Selection for Federated Learning: Sub-modularity and Convergence Analysis (Framework)
Proceedings of the 38th International Conference on Machine Learning, PMLR 139, 2021
this paper, propose to select only a small diverse subset of clients, namely those carrying representative gradient information, and we transmit only these updates to the server.Paper introduce “federated averaging with diverse client selection (`DivFL`)”.

### `FedADP`: Unified Model Aggregation for Federated Learning with Heterogeneous Model Architectures (Framework)
arXiv:2505.06497v1 [cs.LG] 10 May 2025
propose `FedADP`, a federated learning framework designed to adapt to client heterogeneity by dynamically adjusting model architectures during aggregation.

### `FedCVG`: a two-stage robust federated learning optimization algorithm (Framework)
https://doi.org/10.1038/s41598-025-02722-4 - nature journal
paper designs a two-stage federated learning framework for defending against poisoning attacks—`FedCVG`. During implementation, `FedCVG` first removes malicious clients using a reputation based clustering method, and then optimizes communication overhead through a virtual aggregation mechanism.

### `FedDefender`: Client-Side Attack-Tolerant Federated Learning (Framework) 
https://doi.org/10.1145/3580305.3599346 - nature journal
In this paper, they propose a new defense mechanism that focuses on the client-side, called `FedDefender`, to help benign clients train robust local models and avoid the adverse impact of malicious model updates from attackers, even when a server-side defense cannot identify or remove adversaries.

### FEDERATED LEARNING FRAMEWORK BASED ON TRIMMED MEAN AGGREGATION RULES 
Under review as a conference paper at ICLR 2021
Paper propose new aggregation rules, `Tmean`, to the federated learning algorithm, and propose a federated learning framework based on Byzantine resilient aggregation algorithm.

### FEDERATED LEARNING VIA PLURALITY VOTE
Under review as a conference paper at ICLR 2022
paper propose a new scheme named federated learning via plurality vote (`FedVote`). In each communication round of `FedVote`, workers transmit binary or ternary weights to the server with low communication overhead. The model parameters are aggregated via weighted voting to enhance the resilience against Byzantine attacks.

### `FEDREP`: A BYZANTINE-ROBUST, COMMUNICATION EFFICIENT AND PRIVACY-PRESERVING FRAMEWORK FOR FEDERATED LEARNING
Under review as a conference paper at ICLR 2023
paper propose a novel communication compression method called consensus sparsification (`ConSpar`). And further propose a novel FL framework called FedREP, which is Byzantine-robust, communication-efficient and privacy preserving.

### `FedSelect`: Personalized Federated Learning with Customized Selection of Parameters for Fine-Tuning
code available at https://github.com/lapisrocks/fedselect
this work,  propose `FEDSELECT`, a novel PFL algorithm inspired by the iterative subnetwork discovery procedure used for the Lottery Ticket Hypothesis. `FEDSELECT` incrementally expands subnet works to personalize client parameters, concurrently con ducting global aggregations on the remaining parameters.

### FLAME: Taming Backdoors in Federated Learning
This paper is included in the Proceedings of the 31st USENIX Security Symposium. August 10–12, 2022 • Boston, MA, USA
Paper introduce FLAME, a defense framework that estimates the sufficient amount of noise to be injected to ensure the elimination of backdoors. To minimize the required amount of noise, FLAME uses a model clustering and weight clipping approach.

### `FLDetector`: Defending Federated Learning Against Model Poisoning Attacks via Detecting Malicious Clients
KDD’22, August 14–18, 2022, Washington, DC, USA ©2022 Association for Computing Machinery. ACM ISBN 978-1-4503-9385-0/22/08...$15.00 https://doi.org/10.1145/3534678.3539231
`FLDetector` aims to detect and remove majority of the malicious clients such that a Byzantine-robust or provably robust FL method can learn an accurate global model using the remaining clients. The key observation is that, in model poisoning attacks, the model updates from a client in multiple iterations are inconsistent.

### `FLTrust`: Byzantine-robust Federated Learning via Trust Bootstrapping
arXiv:2012.13995v3 [cs.CR] 12 Apr 2022
This Paper provides a root a trust in the server side so it can detect the malicious clients.

### `LoMar`: A Local Defense Against Poisoning Attack on Federated Learning
arXiv:2201.02873v1 [cs.LG] 8 Jan 2022
paper provide a two-phase defense algorithm called Local Malicious Factor (`LoMar`). In phase I, `LoMar` scores model updates from each remote client by measuring the relative distribution over their neighbors using a kernel density estimation method. In phase II, an optimal threshold is approximated to distinguish malicious and clean updates from a statistical perspective.

### `LeadFL`: Client Self-Defense against Model Poisoning in Federated Learning
Proceedings of the 40th International Conference on Machine Learning, Honolulu, Hawaii, USA. PMLR 202, 2023.
Our code can be found at https://github.com/CarlosChu-c/LeadFL
https://github.com/JMGaljaard/fltk-testbed 
https://github.com/jeremy313/FL-WBC 
https://github.com/sparsefed/sparsefed 
https://github.com/ebagdasa/backdoors101
propose a client-self defense, `LeadFL`, that is combined with existing server-side defenses to thwart backdoor and targeted attacks. The core idea of `LeadFL` is a novel regularization term in local model training such that the Hessian matrix of local gradients is nullified.

### Provably Secure Federated Learning against Malicious Clients
The Thirty-Fifth AAAI Conference on Artificial Intelligence (AAAI-21)
Paper suggests given any base federated learning algorithm, it'll use the algorithm to learn multiple global models, each of which is learnt using a randomly selected subset of clients. When predicting the label of a testing example, it takes majority vote among the global models.

## Surveys

### Vulnerabilities in Federated Learning (Survey)
Received March 16, 2021, accepted April 14, 2021, date of publication April 23, 2021, date of current version May 3, 2021.
this paper is a comprehensive survey on vulnerabilities sources, key attacks on FL, defenses, as well as their unique challenges, and discuss promising future research directions towards more robust FL.

### PRIVACY-PRESERVING MACHINE LEARNING FOR HEALTHCARE: OPEN CHALLENGES AND FUTURE PER SPECTIVES (Survey)
ICLR 2023 Workshop on Trustworthy Machine Learning for Healthcare
In this paper, we conduct a review of recent literature concerning Privacy-Preserving Ma chine Learning (PPML) for healthcare. We primarily focus on privacy-preserving training and inference-as-a-service, and perform a comprehensive review of existing trends, identify challenges, and discuss opportunities for future research directions. 

### Federated learning for secure IoMT-applications in smart healthcare systems: A comprehensive review (Survey)
Knowledge-Based Systems 2023
in this work, the authors present survey on the security of FL-based `IoMT` applications in smart healthcare frameworks. First, the authors introduced `IoMT` devices, their types, applications, datasets, and the `IoMT` security framework. then, the concept of FL, its application domains, and various tools used to develop FL applications are discussed. A comparison of FL-based security techniques with other schemes in the smart healthcare is also presented.

### Advances and Open Problems in Federated Learning (Survey)
arXiv:1912.04977v3 [cs.LG] 9 Mar 2021
Motivated by the explosive growth in `FLresearch`, this paper discusses recent advances and presents an extensive collection of open problems and challenges.

### A Survey on Securing Federated Learning: Analysis of Applications, Attacks, Challenges, and Trends (Survey)
date of current version 3 May 2023 IEEE Access
paper provides a comprehensive analysis of secure FL proposals that both protect user privacy and enhance the performance of the model

### Dataset Security for Machine Learning: Data Poisoning, Backdoor Attacks, and Defenses (Survey)
The goal of this work is to systematically categorize and discuss a wide range of dataset vulnerabilities and exploits, approaches for defending against these threats, and an array of open problems in this space.

## Attacks

### Model Poisoning Attacks to Federated Learning via Multi-Round Consistency (Attack)
arXiv:2404.15611v2 [cs.CR] 6 Jun 2024
this paper propose `PoisonedFL`, which enforces multi-round consistency among the malicious clients’ model updates while not requiring any knowledge about the genuine clients.

### How To Backdoor Federated Learning (Attack)
arXiv:1807.00459v3 [cs.CR] 6 Aug 2019
this paper propose model replacement attack for FL and also a constrain and scale method to enhance the attack. This attack can introduce backdoor functionality in the system.

### `FedESP`: Effective, Stealthy, and Persistent Backdoor Attack on Federated Learning (Attack)
Journal of Information Security and Applications 2025
this paper introduces `FedESP`. It first optimizes the trigger through adversarial training, ensuring its effectiveness even after the attacker ceases the attack, thus enhancing its persistence. A regularization term is incorporated during trigger optimization to further enhance stealth.

### Data Poisoning Attacks Against Federated Learning Systems (Attack)
arXiv:2007.08432v2 [cs.LG] 11 Aug 2020
Code is available at https://github.com/git-disl/DataPoisoning_FL
this paper, we study targeted data poisoning attacks against FL systems in which a malicious subset of the participants aim to poison the global model by sending model updates derived from mislabeled data. They also study attack longevity in early/late round training, the impact of malicious participant availability, and the relationships between the two. They also propose a defense algorithm against data poisoning.

### A Little Is Enough: Circumventing Defenses For Distributed Learning (Attack)
33rd Conference on Neural Information Processing Systems (`NeurIPS` 2019), Vancouver, Canada.
The paper claim that it can abuse high variance between gradients to perform an non-omniscient attack. This paper shows that the variance is indeed high enough even for simple datasets such as MNIST, allowing an attack that is not only undetected by existing defenses, but also uses their power against them, causing those defense mechanisms to consistently select the byzantine workers while discarding legitimate ones.

### Exploring Clean Label Backdoor Attacks and Defense in Language Models (Attack) 
IEEE/ACM TRANSACTIONS ONAUDIO,SPEECH,ANDLANGUAGEPROCESSING,VOL.32,2024
this study, introduce `Cbat`, a novel and efficient method to perform clean-label backdoor attack with text style, which does not require external trigger, and the poisoned samples are correctly labeled. also introduce an algorithm for defending against backdoor attacks, named CbatD, which effectively erases the poisoned samples by locating the lowest training loss and calculating feature relevance.

### Breaking State-of-the-Art Poisoning Defenses to Federated Learning: An Optimization-Based Attack Framework (Attack)
Code is available at: https://github.com/Yuxin104/BreakSTOAPoisoningDefenses
CIKM ’24, October 21–25, 2024, Boise, ID, USA 
©2024 
ACM ISBN 979-8-4007-0436-9/24/10
https://doi.org/10.1145/3627673.3679566
In this paper, it is shown that novel robust AGRs are also vulnerable to carefully designed poisoning attacks. With the proposed framework a specific attack for each defense is customized.

### Security and Privacy Issues of Federated Learning
arXiv:2307.12181v1 [cs.CR] 22 Jul 2023
This paper presents a comprehensive taxonomy of security and privacy challenges in Federated Learning (FL) across various machine learning models, including large language models.
## Books
### Security and Privacy in Federated Learning (Book)
Springer 2023
This book categorizes current attacks in FL and the current defenses as well as future directions.

## Uncategorized 
### Random Client Selection on Contrastive Federated Learning for Tabular Data 
https://arxiv.org/abs/2505.10759v1
This paper presents a comprehensive experimental analysis of gradient-based attacks in CFL environments and evaluates random client selection as a defensive strategy.
### Personalized Federated Learning with Differential Privacy
DOI 10.1109/JIOT.2020.2991416, IEEE Internet of things Journal
This paper proposes a privacy-preserving approach for learning effective personalized models on distributed user data while guaranteeing the differential privacy of user data. Practical issues in a distributed learning system such as user heterogeneity are considered in the proposed approach.

## Library
### `FedML`: A Research Library and Benchmark for Federated Machine Learning (Library)
arXiv:2007.13518v4 [cs.LG] 8 Nov 2020
This paper introduce FedML, an open research library and bench mark to facilitate FL algorithm development and fair performance comparison. FedML supports three computing paradigms: on-device training for edge devices, distributed computing, and single-machine simulation. The source code, documents, and user community are at https://fedml.ai


### X. Cao, Z. Zhang, J. Jia, and N. Z. Gong, “Flcert: Provably secure federated learning against poisoning attacks,” in IEEE Transactions on Information Forensics and Security, 2022.
### Z. Sun, P. Kairouz, A. T. Suresh, and H. B. McMahan, “Can you really backdoor federated learning?” arXiv preprint arXiv:1911.07963, 2019.

### Virat Shejwalkar and Amir Houmansadr. Manipulating the byzantine: Opti mizing model poisoning attacks and defenses for federated learning. In Proceedings of the 28th Annual Network and Distributed System Security Symposium (NDSS’21), 2021.

### Qi Xia, Zeyi Tao, Zijiang Hao, and Qun Li. FABA:analgorithm for fast aggregation against byzantine attacks in distributed neural networks. In Proceedings of the Twenty-Eighth International Joint Conference on Arti f icial Intelligence (IJCAI’19), pages 4824–4830, 2019.

### Sai Praneeth Karimireddy, Lie He, and Martin Jaggi. Learning from history for byzan tine robust optimization. In Proceedings of the 38th In ternational Conference on Machine Learning (ICML’21), volume 139, pages 5311–5319, 2021.



# References
