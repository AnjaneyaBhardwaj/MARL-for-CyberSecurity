# Project Findings: Robust GNN for Cyber Defense

## 1. Executive Summary

This project successfully implemented and evaluated a **Graph Neural Network (GNN)** based Reinforcement Learning agent for autonomous cyber defense in the **CybORG++** simulation environment (Scenario 1b). The primary goal was to demonstrate that incorporating network topology into the agent's state representation leads to superior defense strategies compared to standard methods.

**Key Result:** The GNN-based agent achieved a score of **-256.30**, significantly outperforming a baseline Multi-Layer Perceptron (MLP) agent which scored **-488.80**. This validates the hypothesis that topology awareness is critical for effective cyber defense.

## 2. Performance Comparison: GNN vs. MLP

To validate the architecture, we compared the trained Robust GNN against a standard MLP trained with the same algorithm (REINFORCE/PPO) but without access to the graph structure.

| Model                   | Average Reward (Higher is Better) | Improvement                    |
| :---------------------- | :-------------------------------- | :----------------------------- |
| **Robust GNN**          | **-256.30**                       | **Baseline**                   |
| **Baseline MLP**        | -488.80                           | -90.7% (Worse)                 |
| _Passive Agent (Sleep)_ | _-100.00_                         | _Theoretical Max (No Attacks)_ |

**Conclusion:** The MLP failed to learn a coherent strategy, performing near the level of a random agent. The GNN, by leveraging the connections between hosts (User Subnet $\leftrightarrow$ Enterprise $\leftrightarrow$ Operational), learned to prioritize critical nodes and mitigate threats effectively.

## 3. Generalization & Robustness

We tested the agent against three different types of adversaries to evaluate its robustness. The agent was trained solely against `B_lineAgent`.

| Adversary        | Type             | Score       | Observation                                                                                                                          |
| :--------------- | :--------------- | :---------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| **B_lineAgent**  | Known / Training | **-256.30** | **Strong Performance.** The agent effectively counters this specific threat.                                                         |
| **MeanderAgent** | Unknown / Random | **-378.32** | **Generalization Gap.** The agent struggled against unpredictable, random attacks, indicating overfitting to the training adversary. |
| **SleepAgent**   | Passive          | **-100.00** | **Perfect Efficiency.** The agent correctly identified the lack of threat and minimized unnecessary actions.                         |

## 4. Behavioral Analysis

An analysis of the agent's action distribution revealed a distinct strategic preference:

- **Dominant Strategy:** **"Misinform" (Decoy)**.
- **Behavior:** Instead of purely relying on "Restore" (which clears a compromise but doesn't prevent re-infection) or "Remove", the agent frequently used "Misinform" actions.
- **Interpretation:** The agent learned that deploying decoys is a more cost-effective way to waste the attacker's moves and protect critical assets than constantly fighting for control of already compromised nodes.

## 5. Future Work

To address the generalization gap observed against the `MeanderAgent`, future development should focus on:

1.  **Curriculum Learning:** Training against a mixed pool of adversaries (B_line, Meander, RedPPO) simultaneously.
2.  **Hierarchical RL:** Separating high-level strategic goals (e.g., "Defend Enterprise Server") from low-level execution.
3.  **Graph Attention (GAT):** Replacing GraphSAGE with GAT to allow the model to dynamically weigh the importance of specific neighbors (e.g., the Gateway) during an attack.
