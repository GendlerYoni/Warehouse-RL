# Warehouse-RL

**Multi-agent reinforcement learning for package collection and delivery in Unity ML-Agents.**

Warehouse-RL is a Unity ML-Agents environment in which multiple agents learn to locate colored packages, navigate around obstacles, and deliver each package to the matching delivery zone using **Proximal Policy Optimization (PPO)**.

The project focuses on reinforcement-learning environment design: observations, continuous control, reward shaping, multi-agent interaction, randomized episode layouts, Ray Perception, and training analysis.

This repository is a **portfolio-focused extract** containing the core C# environment logic, PPO configuration, and recorded training visuals from the original Unity project.

---

## Demo

| Before training | After training |
|---|---|
| ![Warehouse agents before training](assets/Before%20training.gif) | ![Warehouse agents after training](assets/After%20training.gif) |

---

## Environment

The default environment contains:

- **4 learning agents**
- **16 packages, randomly assigned red or yellow**
- **2 color-matched delivery zones**
- **4 obstacles/walls**

Each episode randomizes key parts of the environment:

- agent positions and starting rotations
- package positions and colors
- delivery-zone positions
- wall positions

Wall and package spawn positions are resampled when they are too close to either delivery zone.

Agents operate simultaneously in the same warehouse. Rewards are assigned individually to each agent; the implementation does not use a shared team reward or an explicit competitive objective.

### Package-delivery loop

1. An agent searches for a nearby package.
2. Colliding with a red or yellow package collects it.
3. The package object is removed from the environment and the agent stores its color as internal state.
4. The agent changes color to provide visual feedback that it is carrying a package.
5. Its navigation objective switches to the matching red or yellow delivery zone.
6. Entering the correct zone completes the delivery and returns the agent to its default state.

The environment resets when all packages have been delivered or after **1,500 physics steps**.

---

## Observation Space

Each agent collects **10 vector observations in code**.

| Observation | Encoding | Dimensions |
|---|---|---:|
| Agent yaw | `sin(yaw)`, `cos(yaw)` | 2 |
| Agent position | normalized X/Z coordinates | 2 |
| Agent velocity | local-space X/Y/Z velocity | 3 |
| Relative objective position | normalized ΔX / ΔZ | 2 |
| Objective distance | normalized distance | 1 |
| **Total** | | **10** |

The current objective depends on the agent's state:

- **Not carrying a package:** the nearest package detected within the target-search radius
- **Carrying red:** the red delivery zone
- **Carrying yellow:** the yellow delivery zone

### Ray Perception

The original Unity setup also used **ML-Agents Ray Perception Sensors 3D** to provide additional spatial information about the surrounding environment.

<p align="center">
  <img src="assets/Ray%20sensors%20settings.png" alt="Ray Perception sensor settings" width="45%">
  <img src="assets/Ray%20sensors%20in%20simulation.png" alt="Ray Perception sensors in the warehouse simulation" width="45%">
</p>

The Unity scene and prefab configuration are not included in this portfolio extract, so the exact sensor configuration is documented through the saved screenshots rather than reconstructed from the checked-in C# scripts.

---

## Action Space

Each agent controls **3 continuous action channels**.

| Action | Purpose |
|---|---|
| Move X | Strafe left / right |
| Move Z | Move forward / backward |
| Turn Y | Rotate around the vertical axis |

Movement is applied through the agent's `Rigidbody`, while package collection and delivery are handled through collisions and trigger zones rather than separate interaction actions.

---

## Reward Design

The reward function combines sparse task rewards with continuous shaping signals that encourage progress and discourage collisions or unnecessarily long episodes.

### Event rewards

| Event | Reward |
|---|---:|
| Collect a package | **+2.0** |
| Deliver to the matching zone | **+4.0** |
| Collide with a wall or another agent | **−5.0** |

### Progress shaping

Every action step applies:

```text
-0.002
```

to discourage unnecessarily long trajectories.

The agent also compares its current distance to its active objective with the previous step:

| Progress | Reward |
|---|---:|
| Distance decreased | **+0.01** |
| Distance did not decrease | **−0.005** |

These shaping rewards are applied in addition to the per-step penalty.

When the agent collects a package or completes a delivery, the stored distance is reset before shaping begins toward the next objective.

---

## PPO Configuration

The checked-in ML-Agents configuration defines one PPO behavior named `WareHouse1`.

| Parameter | Value |
|---|---:|
| Trainer | PPO |
| Batch size | 1,024 |
| Buffer size | 10,240 |
| Learning rate | 0.0003 |
| Learning-rate schedule | Linear |
| PPO epsilon | 0.2 |
| GAE lambda | 0.95 |
| Epochs per update | 3 |
| Entropy coefficient (`beta`) | 0.01 |
| Hidden units | 256 |
| Hidden layers | 2 |
| Observation normalization | Enabled |
| Discount factor (`gamma`) | 0.99 |
| Time horizon | 128 |
| Max training steps | 100,000,000 |
| Summary frequency | 10,000 |

The configuration uses a **256 × 2 fully connected policy network** with normalized observations and an extrinsic reward signal.

---

## Training Results

The recorded before/after demonstrations show the transition from untrained behavior to agents performing the package-collection and color-matched delivery task.

TensorBoard was used to monitor:

- **Cumulative reward** — the reward accumulated by the learned policy during training
- **Episode length** — the duration of episodes as agent behavior changed

The saved training plot is included as evidence of the training process rather than as a formal success-rate, optimal-policy, or benchmark result.

<p align="center">
  <img src="assets/50m%20training%20graph.png" alt="Warehouse PPO TensorBoard training curves" width="75%">
</p>

---

## Key Repository Structure

```text
Warehouse-RL/
├── WareHouse1Agent.cs
├── WareHouse1Manager.cs
├── WareHouse1.yaml
├── assets/
│   ├── Before training.gif
│   ├── After training.gif
│   ├── 50m training graph.png
│   ├── Ray sensors settings.png
│   └── Ray sensors in simulation.png
└── LICENSE
```

### Main files

- **`WareHouse1Agent.cs`** — observations, continuous actions, movement, package state, collisions, delivery handling, and reward shaping
- **`WareHouse1Manager.cs`** — agent creation, environment randomization, target/wall spawning, episode lifecycle, and delivery tracking
- **`WareHouse1.yaml`** — PPO hyperparameters and neural-network configuration
- **`assets/`** — recorded demonstrations, Ray Perception screenshots, and TensorBoard training results

---

## Technologies

- **Unity**
- **C#**
- **Unity ML-Agents**
- **Proximal Policy Optimization (PPO)**
- **TensorBoard**
- **Deep Reinforcement Learning**
- **Multi-Agent Reinforcement Learning**

---

## Project Scope

Warehouse-RL is preserved as a compact portfolio project rather than a complete Unity distribution.

The repository contains the core learning-agent logic, environment manager, PPO training configuration, and recorded visual results. Unity scenes, prefabs, generated project files, training checkpoints, and engine-generated directories are intentionally not included.

The project demonstrates practical experience with designing an RL environment, defining observation and action spaces, shaping rewards, training multiple learning agents in a shared simulation, randomizing training conditions, and analyzing PPO training behavior.

---

## Contact

**Yoni Gendler**

- 💼 [LinkedIn](https://www.linkedin.com/in/yoni-gendler/)
- 💻 [GitHub](https://github.com/GendlerYoni)
- 📧 [gendler.yoni.dev@gmail.com](mailto:gendler.yoni.dev@gmail.com)
