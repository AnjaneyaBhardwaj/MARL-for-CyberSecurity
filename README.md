# Multi-Agent Reinforcement Learning for Autonomous Security & Intrusion Detection

This repository contains the research and implementation of an AI-driven cybersecurity defense system. Developed for **CSE 546: Reinforcement Learning** at the **University at Buffalo**, this project utilizes Multi-Agent Reinforcement Learning (MARL) and Graph Neural Networks (GNNs) to automate cyber defense in dynamic network environments.

## 📋 Project Overview

Enterprise networks are increasingly vulnerable to sophisticated, evolving adversarial attacks. This project addresses the lack of autonomous defense systems by modeling a digital network as a dynamic graph. Through adversarial self-play, we train agents to adapt in real-time while prioritizing the protection of critical assets.

### Key Features

* **Environment:** CybORG++ (Scenario 1b)
* **Architecture:** Multi-Agent Adversarial Framework
* **Algorithm:** Proximal Policy Optimization (PPO) with GraphSAGE integration
* **Training Strategy:** Adversarial Self-Play (Co-evolutionary training)

## 🚀 Methodology

### 1. Network Modeling (CybORG++)

We utilize the **Cyber Operations Research Gym (CybORG++)**, an open-source AI gym environment. The network is modeled as a dynamic graph, representing different zones (User, Enterprise, Operational) and assets.

### 2. State Representation

We integrate **GraphSAGE** for topology-aware state representation. This allows the Blue-Team (defenders) to understand the relational structure of the network and identify potential lateral movement patterns by the Red-Team (attackers).

### 3. Reinforcement Learning Algorithms

The primary training utilizes **PPO**, while **Double DQN (DDQN)** is used as a baseline for performance metrics.

#### Mathematical Framework

* **PPO Clipping Objective:** $$
  L^{clip}(\theta)=\mathbb{E}_{r}[min(r_{t}(\theta)\cdot A_{t},clip(r_{t}(\theta),1-\epsilon,1+\epsilon)\cdot A_{t})]
  $$

* **Double DQN Target:** $$
  Y_{t}^{DoubleDQN}\equiv R_{t+1}+\gamma Q(S_{t+1},arg\max_{a}Q(S_{t+1},a;\theta_{t}),\theta_{t}^{-})
  $$

## 📊 Results

The Blue agent (defender) was trained through three iterations of adversarial self-play.

### Performance Benchmarks

| **Algorithm** | **Avg Reward** | **Improvement** | 
| :--- | :--- | :--- |
| **Adversarial Self-Play (Iter 3)** | **-21.82** | **47% over baseline** | 
| DDQN | -47.5 | - | 
| PPO (Standard) | -129 | - | 

### Key Outcomes

* **Efficiency:** The Iter3 agent achieved a best reward of **-4.80**, placing it **78.2% closer** to the theoretical maximum than the SleepAgent baseline.
* **Emergent Behavior:** The system discovered emergent **"castle defense"** strategies without explicit reward shaping.
* **Adaptability:** Red agents co-evolved, achieving 49.80 attack rewards through learned privilege escalation, forcing the Blue agent to develop more robust defenses.

## 📝 Conclusion

This project validates that **topology-aware GNNs** combined with **co-evolutionary training** can create autonomous defenders capable of adapting to sophisticated, evolving cyber threats. The results show a significant 47% improvement over baseline agents, proving the efficacy of adversarial self-play in cybersecurity contexts.

## 👥 Authors

* **Anjaneya Bhardwaj** - University at Buffalo ([anjaneya@buffalo.edu](mailto:anjaneya@buffalo.edu))
* **Syed Mohammed Faham** - University at Buffalo ([sfaham@buffalo.edu](mailto:sfaham@buffalo.edu))
* **Instructor:** Alina Vereshchaka

## 📚 References

1. Kenneth Limmen. *Awesome Reinforcement Learning for Cybersecurity*. GitHub Repository, 2024.
2. Shaukat, K., et al. *Cyber Threat Detection Using Machine Learning Techniques: A Comprehensive Survey*, IEEE Access, 2020.
3. Yu, C., et al. *Multi-Agent Proximal Policy Optimization (MAPPO)*. arXiv:2103.01955, 2021.
4. Harold Booth et al. *CybORG++: A Cyber Operations Research Gym Environment*. Defence Science and Technology Group, 2023.
