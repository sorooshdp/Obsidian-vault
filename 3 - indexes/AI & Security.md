---
created: 2026-11-10 13:54
updated: 2026-11-10 13:54
type: moc
status: mature
tags:
  - security
  - ai
aliases:
  - AI Security MOC
  - Federated Learning MOC
---

# AI Security & Federated Learning Hub

## 1. Federated Learning Foundations & Literature
- [[Federated learning]] — Core 4-step aggregation loop and parameter-sharing architecture
- [[A Survey of Security Strategies in Federated Learning]] — HFL/VFL/FTL taxonomy, Cross-Silo vs. Cross-Device, attack & defense phases
- [[FL Paper Abstracts Notes]] — Literature review abstracts on aggregation, personalization, and robustness
- [[Benign Overfitting]] — Overparameterization and generalization on noisy training data

## 2. Adversarial Attacks on ML & Federated Learning
- [[How To Backdoor Federated Learning]] — Single-shot model replacement, constrain-and-scale, and secure aggregation trade-offs
- [[Poison frogs!]] — Clean-label targeted data poisoning and feature-space decision boundary rotation
- [[Inference attack]] — Extracting training set privacy from model memory
- [[Machine Learning through Visual Analytics]] — Attacker taxonomy (Goal, Knowledge, Capability, Strategy) and diagnosing poisoning impact

## 3. Defenses & Application Security Principles
- [[Defenses against Model and Data Attacks]] — Server-side Detect-and-Remove (`LoMar`, `FederatedReverse`, `DDaBA`, `BAFFLE`, KPCA)
- [[Principal of least privilege]] — TCB minimization and privilege restriction
- [[Trust boundary]] — Execution vs. data trust boundaries and validation violations
- [[SQL Injection]] — Unsanitized input exploitation across database trust boundaries