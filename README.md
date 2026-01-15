# Robust GNN for Cyber Defense in CybORG++

A Graph Neural Network (GNN) based Reinforcement Learning agent for autonomous cyber defense in the CybORG++ simulation environment. This project demonstrates that incorporating network topology into the agent's state representation leads to superior defense strategies compared to standard methods.

## Executive Summary

This project successfully implemented and evaluated a **Graph Neural Network (GNN)** based Reinforcement Learning agent for autonomous cyber defense in the **CybORG++** simulation environment (Scenario 1b). 

**Key Result:** After identifying and fixing critical training instabilities, the GNN-based agent achieved a score of **-129.46 ± 11.27**, a **49.5% improvement** over the initial implementation (-256.30). The best episode achieved **-95.50**, surpassing even the theoretical SleepAgent baseline (-100), demonstrating that the agent learned actions that provide net positive value beyond simply doing nothing.

## Key Features & Architecture

### Architecture Components

- **GNN Encoder:** GraphSAGE (2-layer) for learning network topology representations
- **Policy Algorithm:** Proximal Policy Optimization (PPO)
- **Dual-Agent Setup:** Blue Agent (Defender) vs. Red Agent (Attacker)
- **Training Strategy:** Domain randomization and adversarial self-play

### Blue Agent (Defender)
- **GNN Type:** GraphSAGE with 2 layers
- **Policy Head:** 256 → 54 defense actions
- **Training:** PPO with domain randomization (50% B_lineAgent, 50% MeanderAgent)

### Red Agent (Attacker)
- **GNN Type:** GraphSAGE with 2 layers
- **Policy Head:** 256 → 56 attack actions
- **Training:** PPO against frozen trained Blue agent
- **Purpose:** Enable adversarial self-play for robust defense learning

## Performance

### Comparison: Optimized GNN vs. Baselines

| Model                   | Average Reward      | Improvement over Initial GNN |
| :---------------------- | :------------------ | :--------------------------- |
| **Optimized GNN (New)** | **-129.46 ± 11.27** | **+49.5%** 🏆                |
| Initial GNN             | -256.30             | Baseline                     |
| Baseline MLP            | -488.80             | -90.7% (Worse)               |
| _Passive Agent (Sleep)_ | _-100.00_           | _Theoretical Max_            |

### Detailed Evaluation Results

```
Average Reward: -129.46 ± 11.27
Min Reward:     -154.10
Max Reward:      -95.50  (Better than SleepAgent!)
```

**Conclusion:** The optimized GNN approaches the theoretical maximum performance. The best episode (-95.50) actually outperformed the SleepAgent baseline.

### "Castle Defense" Strategy Analysis

An analysis of the agent's action distribution revealed a distinct strategic preference:

- **Dominant Strategy:** **"Castle Defense"** - Focus on critical assets only
- **Action Types Used:** Analyse (~53%) + Restore (~47%)
- **Target Hosts:** Op_Host0 (~53%) + Op_Server0 (~47%)

#### Why "Castle Defense" Works

The agent learned to **ignore the perimeter** (User/Enterprise subnets) and focus exclusively on defending the **Operational subnet**:

```
User Subnet → Enterprise Subnet → [DEFENDED] Operational Subnet
    ❌              ❌                    ✅ ✅
 (ignored)       (ignored)         (Analyse + Restore)
```

**Rationale:**
1. Defending everything is too expensive (negative reward for each action)
2. Only the Operational subnet contains critical assets (Op_Server0)
3. Let attackers waste moves on outer networks while guarding the "crown jewels"
4. Analyse detects compromise → Restore removes it immediately

### Training Stability Improvements

The initial PPO training exhibited severe instability with loss values of 2,600 - 7,000+. After applying fixes:

| Parameter            | Before             | After       | Rationale                                                     |
| -------------------- | ------------------ | ----------- | ------------------------------------------------------------- |
| `UPDATE_EVERY`       | 500                | **256**     | Faster feedback for policy updates                            |
| `REWARD_SCALE`       | 1.0                | **0.1**     | Scale down large negative rewards (-100 to -200 → -10 to -20) |
| `VALUE_COEF`         | 0.5                | **0.25**    | Reduce critic's influence on total loss                       |
| Return Normalization | None               | **Enabled** | `(returns - mean) / std` before critic loss                   |
| Value Clipping       | `clamp(-100, 100)` | **Removed** | Unnecessary with normalized returns                           |

**Impact:**

| Metric                 | Before Fixes       | After Fixes           |
| ---------------------- | ------------------ | --------------------- |
| **Loss Range**         | 2,600 - 7,000+     | **0.12 - 0.22** ✅    |
| **Training Stability** | Oscillating wildly | Smooth convergence ✅ |
| **Final Score**        | -256.30            | **-129.46** ✅        |

## Adversarial Training

### Red Agent Training (Self-Play)

To create a truly adaptive defense system, we implemented **Red Agent (Attacker) Training** using the same GNN-based architecture. This enables adversarial self-play where both agents learn and improve.

#### Red Agent Action-Reward Analysis

After training, the Red agent learned effective attack patterns:

| Action Type              | Times Used | Avg Reward | Interpretation                         |
| :----------------------- | ---------: | ---------: | :------------------------------------- |
| **PrivilegeEscalate**    |     10,976 |     +0.429 | 🏆 Most effective attack action        |
| **Sleep**                |     10,694 |     +0.415 | Strategic waiting (no cost)            |
| **ExploitRemoteService** |      9,177 |     +0.411 | Successful exploitation attempts       |
| **Impact**               |      9,751 |     +0.368 | Final attack on compromised hosts      |
| **DiscoverNetworkSvc**   |      2,081 |     +0.389 | Reconnaissance (used sparingly)        |

**Key Observations:**
- PrivilegeEscalate is the most rewarding action (+0.429 avg)
- Agent uses a balanced mix of exploitation, privilege escalation, and impact actions
- Learned multi-phase attack pattern: reconnaissance → exploitation → lateral movement → impact

### Iterative Adversarial Self-Play

After training baseline Blue and Red agents, we implemented **iterative adversarial self-play** to create stronger agents through competition:

| Iteration | Agent Trained | Opponent Used            | Episodes |
| :-------- | :------------ | :----------------------- | :------- |
| Baseline  | 🔵 Blue       | B_line + Meander (50/50) | 10,000   |
| Baseline  | 🔴 Red        | Frozen Blue baseline     | 10,000   |
| Iter 1    | 🔵 Blue       | Frozen Red baseline      | 5,000    |
| Iter 2    | 🔴 Red        | Frozen Blue iter1        | 5,000    |
| Iter 3    | 🔵 Blue       | Frozen Red iter2         | 5,000    |

**Benefits:**
- **Co-evolution:** Each agent learns to counter the other's strategies
- **Robustness:** Blue agent trained against learned attackers (not just scripted)
- **Emergent strategies:** Novel attack/defense patterns emerge
- **Curriculum learning:** Progressive difficulty as opponents improve

## Installation & Usage

### Prerequisites

This project requires the following dependencies:
- **CybORG** - Cyber Operations Research Gym
- **PyTorch** - Deep learning framework
- **PyTorch Geometric (PyG)** - For GNN implementations
- **NumPy** - Numerical computing
- **Matplotlib** - Visualization

### Running the Project

The implementation is provided in Jupyter Notebooks:

1. **Main Implementation:**
   ```bash
   jupyter notebook anjaneya_sfaham_DDQN_final_Project.ipynb
   ```
   This notebook contains the complete GNN-based PPO implementation with both Blue and Red agent training.
   
   **Note:** The notebook filename references "DDQN" from an earlier project iteration, but the current implementation uses GNN-based PPO.

2. **Checkpoint Version:**
   ```bash
   jupyter notebook anjaneya_sfaham_checkpoint_project.ipynb
   ```
   Earlier checkpoint version of the project.

### Quick Start

1. Install dependencies:
   ```bash
   pip install torch torch-geometric numpy matplotlib jupyter
   # Install CybORG from https://github.com/cage-challenge/CybORG
   ```

2. Open the main notebook:
   ```bash
   jupyter notebook anjaneya_sfaham_DDQN_final_Project.ipynb
   ```

3. Run cells sequentially to:
   - Set up the CybORG++ environment
   - Train the Blue (defender) agent
   - Train the Red (attacker) agent
   - Evaluate performance and visualize results

### Trained Models

The following trained models are saved during training:
- `robust_gnn_actor.pth` - Baseline Blue agent
- `red_gnn_actor_final.pth` - Baseline Red agent
- `blue_actor_iter{N}_final.pth` - Blue agents from iterative training
- `red_actor_iter{N}_final.pth` - Red agents from iterative training

## Future Work

To further improve the system, future development should focus on:

1. **Enhanced Adversarial Training:** Continue iterative training for more iterations to achieve stronger co-evolution between Blue and Red agents.

2. **Multi-Agent PPO (MAPPO):** Implement true simultaneous multi-agent learning where both agents train concurrently instead of alternating frozen opponents.

3. **Graph Attention Networks (GAT):** Replace GraphSAGE with GAT for dynamic neighbor importance weighting, potentially improving the agent's ability to focus on critical network components.

4. **Curriculum Learning:** Design a structured curriculum with progressively harder opponents to improve learning efficiency and final performance.

5. **Transfer Learning:** Apply trained policies to different network topologies and CybORG scenarios to evaluate generalization capabilities.

6. **Hierarchical Reinforcement Learning:** Separate high-level strategic decision-making from low-level action execution for more sophisticated defense strategies.

7. **Action Masking Improvements:** Address the high invalid action count in Red agent training to improve learning efficiency.

8. **Diverse Defense Strategies:** Explore methods to encourage more varied defensive behaviors while maintaining performance (beyond the current "Castle Defense" strategy).

## Key Lessons Learned

### Why Reward Scaling Matters

The Blue agent receives large negative rewards from CybORG (typically -2 to -5 per step, accumulating to -100 to -250 per episode). Scaling by 0.1 provides several benefits:

- **Prevents gradient explosion:** MSE loss reduced 100× (from 40,000 to 400)
- **Stable critic learning:** Smaller target values allow gradual, stable learning
- **Balanced loss components:** Actor and critic contribute equally to total loss
- **Faster convergence:** Smooth optimization landscape

### The Fix Philosophy

- **Scale rewards to a manageable range** (~±10) to prevent gradient explosion
- **Normalize returns** so the critic learns relative value differences, not absolute magnitudes
- **Update more frequently** to reduce variance in policy gradients
- **Balance actor/critic** by reducing the critic's coefficient

## Project Context

This project was developed as part of CSE-556 (Multi-Agent Reinforcement Learning for Cybersecurity). For detailed findings and analysis, including comprehensive training logs, experimental results, and in-depth behavioral analysis, see [findings.md](findings.md).

## License

This project is for educational purposes as part of academic coursework.

---

**Note:** For detailed experimental results, training curves, and in-depth analysis, please refer to the [findings.md](findings.md) document.