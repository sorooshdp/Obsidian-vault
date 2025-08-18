2025/05/14  -  02:26
status: 

Tags: [[AI]] [[Data Poisoning]] 

**Data poisoning**: is an attack on machine learning models wherein the attacker adds examples to the training set to manipulate the behavior of the model at test time. These attacks happen at training time; they aim to manipulate the performance of a system by inserting carefully constructed poison instances into the training data. Classical poisoning attacks indiscriminately degrade test accuracy rather than targeting specific examples, making them easy to detect.

**Evasion attack**: test time – a clean target instance is modified to avoid detection by a classifier, or spur misclassification.

The paper introduces a **clean-label targeted poisoning attack**, where **correctly labeled** data is added to the training set to cause **specific misclassifications** at test time—without degrading overall model accuracy or requiring control over the labeling process. Traditional **evasion attacks** modify test-time inputs, which isn’t always practical. 

Our strategy assumes that the attacker has no knowledge of the training data but has knowledge of the model and its parameters.

Poisoning in the end-to-end training scenario is difficult because the network learns feature embeddings that optimally distinguish the target from the poison.

Transfer learning reacts to poisons by rotating the decision boundary to encompass the target, while end-to-end training reacts by pulling the target into the base distribution (in feature space).

We can increase the success rate of this attack by targeting data outliers. These targets lie far from other training samples in their class, and so it should be easier to flip their class label. We target the 50 “airplanes” with the lowest classification confidence (but still correctly classified), and attack them using 50 poison frogs per attack. The success rate for this attack is 70% (Fig. 5a), which is 17% higher than for randomly chosen targets.


# References
