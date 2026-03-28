# UR5e Reach Task — Isaac Lab

A custom reinforcement learning environment for the **Universal Robots UR5e** robotic arm, built with [Isaac Lab](https://isaac-sim.github.io/IsaacLab). The agent learns to reach target end-effector poses using **Proximal Policy Optimization (PPO)** via the [skrl](https://skrl.readthedocs.io) library.

---

## Overview

The task is a 6-DOF end-effector pose tracking problem. The robot receives a randomly sampled target pose within its reachable workspace and must move its `wrist_3_link` to match the target position and orientation. The environment is fully parallelized using Isaac Lab's Manager-Based RL framework.

**Key details:**

- Robot: Universal Robots UR5e
- Task: End-effector reach (position + orientation tracking)
- RL algorithm: PPO (skrl)
- Environment: 4096 parallel envs (configurable)
- Action space: Joint position control (scaled, with default offset)
- Observation space: Joint positions, joint velocities, pose command, last action
- Episode length: 12 seconds

---

## Requirements

| Dependency | Version |
|---|---|
| Isaac Sim | 5.0.0 |
| Isaac Lab | 0.46 |
| skrl | latest |
| Python | 3.11 |
| CUDA | 11.8 |

---

## Installation

**1. Install Isaac Lab** by following the [official guide](https://isaac-sim.github.io/IsaacLab/main/source/setup/installation/index.html).

**2. Clone this repository** outside the IsaacLab directory:

```bash
git clone https://github.com/FarkasBalintKaroly/IsaacLab_UR5e_Reach_Task.git
cd IsaacLab_UR5e_Reach_Task
```

**3. Install the package** in editable mode:

```bash
# Replace PATH_TO with the actual path to your IsaacLab installation
PATH_TO/isaaclab.sh -p -m pip install -e source/UR5e_reach_task
PATH_TO/isaaclab.sh -p -m pip install -e source/isaaclab_assets
```

**4. Place the UR5e USD asset** in the repository root:

```
IsaacLab_UR5e_Reach_Task/
└── ur5e.usd      ← required
```

---

## Training

```bash
../../IsaacLab/isaaclab.sh -p scripts/skrl/train.py --task=Isaac-Reach-UR5E-v0 --headless
```

To reduce the number of parallel environments (e.g. for less VRAM):

```bash
../../IsaacLab/isaaclab.sh -p scripts/skrl/train.py --task=Isaac-Reach-UR5E-v0 --headless --num_envs 512
```

Training logs are saved to:

```
logs/skrl/reach_ur5/
```

Monitor training with TensorBoard (in a separate terminal):

```bash
../../IsaacLab/isaaclab.sh -p -m tensorboard.main --logdir logs/skrl/reach_ur5
```

---

## Evaluation / Play

```bash
../../IsaacLab/isaaclab.sh -p scripts/skrl/play.py --task=Isaac-Reach-UR5E-v0 --num_envs 50
```

---

## Project Structure

```
IsaacLab_UR5e_Reach_Task/
├── scripts/
│   ├── skrl/
│   │   ├── train.py              # PPO training entry point
│   │   └── play.py               # Evaluation / playback
│   ├── zero_agent.py             # Zero-action baseline
│   └── random_agent.py           # Random-action baseline
├── source/
│   ├── isaaclab_assets/
│   │   └── isaaclab_assets/
│   │       └── robots/
│   │           └── universal_robots.py   # UR5E_CFG definition
│   └── UR5e_reach_task/
│       └── UR5e_reach_task/
│           └── tasks/
│               └── manager_based/
│                   └── ur5e_reach_task/
│                       ├── ur5e_reach_task_env_cfg.py  # Full env config
│                       ├── mdp/                        # MDP components
│                       ├── agents/
│                       │   └── skrl_ppo_cfg.yaml       # PPO hyperparameters
│                       └── __init__.py                 # Task registration
├── ur5e.usd                      # UR5e robot asset (required)
└── README.md
```

---


## Known Issues

- **Warp CUDA warning** on startup (`Failed to get driver entry point 'cuDeviceGetUuid'`) — cosmetic, does not affect training.
- **CPU powersave mode** warning — switch to performance mode for faster training: `sudo cpupower frequency-set -g performance`
- The `isaaclab_assets` package in this repo takes precedence over the Isaac Lab built-in version — this is intentional, as it provides the custom `UR5E_CFG`.

---

## License
 
This project is licensed under the [MIT License](LICENSE).

It builds upon [Isaac Lab](https://github.com/isaac-sim/IsaacLab), 
which is distributed under the Apache 2.0 License.

© 2025 Farkas Bálint
