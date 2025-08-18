2025/05/01  -  17:39
status: 

Tags: [[AI]] [[Federated Learning]]

Federated learning is generically vulnerable to model poisoning attack. neither defenses against data poisoning, nor anomaly detection can be used during federated learning because they require access to, respectively, the participants’ training data or their submitted model updates.

**Federated learning**: enables thousands of participants to construct a deep learning model without sharing their private training data with each other. For example, multiple smartphones can jointly train a next-word predictor for keyboards without revealing what individual users type.

**Federated models**: created by aggregating model updates submitted by practitioners. the aggregator by design has no visibility into how these updates are generated and that makes them vulnerable to models poisoning attacks that are more powerful then data poisoning attacks. A framework for the massively distributed training of deep learning models with thousands or even millions of participants. Motivating applications include training image classifiers and next-word predictors on users’ smartphones.

The attacker does not control the aggregation algorithm used to combine participants’ updates into the joint model, nor any aspects of the benign participants’ training.

**Model replacement**: takes advantage of the observation that a participant in federated learning can:
1. directly influence the weights of the joint model, and
2. train in any way that benefits the attack, e.g., arbitrarily modify the weights of its local model and/or incorporate the evasion of potential defenses into its loss function during training.
It is a single shot attack: the global model exhibits high accuracy on the backdoor task immediately after it has been poisoned.

Even a single-shot attack, where a single attacker is selected in a single round of training, causes the joint model to achieve 100% accuracy on the backdoor task. when training with millions of participants, it is impossible to ensure that none of them are malicious.

neither defenses against data poisoning, nor anomaly detection can be used during federated learning because they require access to, respectively, the participants’ training data or their submitted model updates.

The aggregation server cannot observe either the training data, or model updates based on these data [45, 48] without breaking participants’ privacy, which is the key motivation for federated learning.

boosted learning rate cases catastrophic forgetting!

**Secure ML**: Secure multi-party computation can help train models while protecting privacy of the training data [47], but it does not protect model integrity. Specialized solutions, such as training secret models on encrypted, vertically partitioned data [26], are not applicable to federated learning.

Secure aggregation makes our attack easier because it prevents the central server from detecting anomalous updates and tracing them to a specific participant.

**Pixel pattern backdoor**: These backdoors require the attacker to modify the pixels of the digital image in a special way at test time in order for the model to misclassify the modified image.

**Adversarial transformation**: exploit the boundaries between the model’s representations of different classes to produce inputs that are misclassified by the model.

**Backdoor attacks**: backdoor attacks intentionally shift these boundaries so that certain inputs are misclassified.

**Semantic backdoors**: cause the model to misclassify even the inputs that are not changed by the attacker.

Federated learning gives the attacker full control over one or several participants, e.g., smartphones whose learning software has been compromised by malware. 
1. The attacker controls the local training data of any compromised participant; 
2.  it controls the local training procedure and the hyperparameters such as the number of epochs and learning rate; 
3.  it can modify the weights of the resulting model before submitting it for aggregation; and, 
4.  it can adaptively change its local training from round to round.

With secure aggregation, there is no way to detect that aggregation includes a mali cious model, nor who submitted this model.

Constrain-and-scale: Intuitively, we incorporate the evasion of anomaly detection into the training by using an objective function that (1) rewards the model for accuracy and (2) penalizes it for deviating from what the aggregator considers “normal”.

**Kerckhoffs's Principle** is a fundamental concept in cryptography, but its core idea is broadly applicable to security system design. It states that a **cryptosystem (or any security system) should be secure even if everything about the system, except for the secret key, is public knowledge.**

Train-and-scale: The attacker trains the backdoored model until it converges and then scales up the model weights byγ up to the bound S permitted by the anomaly detector.

The two key requirements for federated learning are: (1) it should handle participants’ local training data that are not i.i.d., and (2) these data should remain confidential and private.

Backdoor S attacks do not target privacy, but two key steps of differentially private training may limit their efficacy. First, each participant’s parameters are clipped, i.e., multiplied by min(1, | |Lt+1 i −Gt ||2 ) to bound the sensitivity of model updates. Second, Gaussian noise N(0,σ) is added to the weighted average of updates. The attacker always knows this bound because it is sent to all participants.

Critically, the low clipping bounds and high noise variance that render the backdoor attack ineffective also greatly de crease the accuracy of the global model on its main task.
# References

[48] M. Nasr, R. Shokri, and A. Houmansadr. Comprehensive privacy analysis of deep learning: Stand-alone and federated learning under passive and active white-box inference attacks. In S&P, 2019
[45] L. Melis, C. Song, E. De Cristofaro, and V. Shmatikov. Exploiting unintended feature leakage in collaborative learning. In S&P, 2019.
