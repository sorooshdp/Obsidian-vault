2025/05/01  -  15:30
status: 

Tags: [[AI]] [[Data Poisoning]]

Model Explain ability: The users want to understand what is happening under the black box of machine learning models and there are different approaches for this goal.

Model Performance Diagnosis: also it is crucial for users to understand the performance of the model. one of the new ways is confusion wheel. For example, the INFUSE system supports the interactive ranking of features based on feature selection algorithms and cross-validation performances.

beside this features mentioned, it is also important to visualize the model weaknesses. 
four main features of an attacker: Goal, Knowledge, Capability, and Strategy. 

- Goal: what do we want from the attack?
- Knowledge: How much do we know about the model?
- Capability: In which way and to what extend can we manipulate the training?
- Strategy: How can we achieve the goal?

**Poisoning attacks**: take place during the training-stage, and the attacker attempts to manipulate the training dataset. Typical operations in data poisoning attacks include adding noise instances and flipping labels of existing instances.

**evasion attack**: takes place during the testing stage. Such an attack is intended to manipulate unlabeled data in order to avoid detection in the testing stage without touching the training process.

an attacker’s goal can be separated into two major categories: targeted attacks and reliability attacks. In a targeted attack, the attacker seeks to insert specific malicious instances or regions in the input feature space and prevent these insertions from being detected . In a reliability attack, the goal of the attacker is to maximize the overall prediction error of the model and make the model unusable for making predictions.

tasks for analyzing poisoning attack strategies: 1- Summarize the attack space 2- Summarize the attack results : How many poisoning data instances are inserted? What is their distribution? Has the attack goal been achieved yet? - What is the performance of the model before and after the attack and is there a significant difference? How many instances in the training dataset are misclassified by the poisoned model? 3- Diagnose the impact of data poisoning: At the feature-level, what is the impact of data poisoning on the feature distributions?

Design requirements: 1- visualize attack the attack space 2- visualize results:
- Model Overview - summarize prediction performance for the victim model as well as the poisoned model 
- Data Instances - present the labels of the original and poisoned data instances
- Data Features - visualize the statistical distributions of data along each feature  
- Local Impacts - depict the relationships between target data instances and their nearest neighbors 



# References
