```markdown
#  Dual-Brain Edge Intelligence for Energy-Efficient Home Automation

[![Python 3.11](https://img.shields.io/badge/Python-3.11-blue.svg)](https://www.python.org/downloads/release/python-3110/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.2.0-ee4c2c.svg)](https://pytorch.org/)
[![Home Assistant](https://img.shields.io/badge/Home_Assistant-Integration-41BDF5.svg)](https://www.home-assistant.io/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

This repository contains the complete implementation of the **Dual-Brain Edge Intelligence Architecture**, the software engineering thesis project by David Kimani Mungai (L202126100203). 

This system resolves the "Comfort Penalty," cloud latency, and data fragmentation issues common in commercial smart homes. It achieves this by bridging predictive load forecasting (**1D-DCNN + BLSTM**) with adaptive reinforcement learning (**Deep Q-Network**) on a localized, offline-first edge computing hub.

---

## System Architecture: The Dual-Hub Approach

To overcome the computational limits of edge hardware, this codebase is strictly physically decoupled into two operational environments:

1. **`offline_training_hub/` (High-Compute / GPU)**
   * Handles massive data ingestion (e.g., the 6GB UK-DALE corpus & EnergyPlus).
   * Executes heavy PyTorch tensor operations to train the forecasting and scheduling models.

2. **`edge_deployment_hub/` (Intel NUC / Home Assistant)**
   * Contains the `ai_docker_addon` that operates as a stateless microservice running inside Home Assistant.
   * Passively ingests normalized sensor data via a local Mosquitto MQTT broker.
   * Executes sub-100ms actuation commands directly to the Home Assistant OS via secure REST APIs.
   * Enforces absolute physical safety guardrails via deterministic `ha_core_config` YAML logic (Hardware Freeze Protection & Human-in-the-Loop overrides).

---

##Repository File Tree

```text
low-cost-energy-efficient-home-automation-system/
├── edge_deployment_hub/                 # LIVE EDGE EXECUTION ENVIRONMENT
│   ├── ai_docker_addon/                 # Containerized HA Microservice
│   │   ├── models/                      # Deployment-ready .pth weights
│   │   │   ├── dqn_smart_home.pth
│   │   │   └── hybrid_forecaster.pth
│   │   ├── Dockerfile
│   │   ├── config.yaml
│   │   ├── forecasting_pipeline.py
│   │   ├── live_deployment.py           # Main inference loop
│   │   ├── networks.py
│   │   └── requirements.txt
│   └── ha_core_config/                  # HA Safety Guardrails
│       ├── automations.yaml
│       └── configuration.yaml
│
├── offline_training_hub/                # GPU MODEL TRAINING ENVIRONMENT
│   ├── src/
│   │   ├── rl_agent/                    # Core RL/MDP Logic
│   │   ├── forecasting_pipeline.py
│   │   ├── ha_bridge.py
│   │   └── virtual_house_daemon.py      # MQTT Telemetry Simulator
│   ├── config.yaml
│   ├── dataset_generator.py             # Synthetic data generation
│   ├── offline_h5_training.py           # UK-DALE .h5 parser & trainer
│   ├── requirements.txt
│   ├── test_env.py                      # Gym environment sanity check
│   └── train_agent.py                   # DQN Training Script
│
├── .gitignore
└── README.md

```

---

## Environment Setup

```bash
git clone [https://github.com/Tannedcroissant/low-cost-energy-efficient-home-automation-system.git](https://github.com/Tannedcroissant/low-cost-energy-efficient-home-automation-system.git)
cd low-cost-energy-efficient-home-automation-system

# Create and activate virtual environment inside the offline hub
cd offline_training_hub
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

```



## Phase 1: Offline Training (Generating the Models)

This phase trains the predictive and scheduling models. To reproduce the results, ensure you are in the `offline_training_hub/` directory.

**1. Data Provisioning (UK-DALE vs. Synthetic):**
To replicate the thesis results (8.2% MAPE), download the 6GB `ukdale.h5` dataset and run the offline parser. For rapid reviewer testing, you can generate a lightweight synthetic dataset instead:

```bash
# Option A: Train on raw UK-DALE data
python3 offline_h5_training.py

# Option B: Generate synthetic test data
python3 dataset_generator.py

```

**2. Verify the Gymnasium Environment:**
Runs a quick deterministic test of the custom MDP environment to ensure the state tensor and reward functions are initialized correctly.

```bash
python3 test_env.py

```

**3. Train the Adaptive Scheduler (Deep Q-Network):**
Trains the reinforcement learning agent over 500 simulated episodes to balance peak-load shaving against the quadratically scaled thermal discomfort penalty ($U_{t}$).

```bash
python3 train_agent.py

```

*(Once training is complete, the optimal `.pth` weights must be copied into the `edge_deployment_hub/ai_docker_addon/models/` directory for deployment).*



## Phase 2: Edge Deployment (Home Assistant SIL Simulation)

With the `.pth` weights generated and placed in the edge hub, the system shifts to the lightweight, localized execution loop. To demonstrate the real-time, sub-100ms actuation loop without requiring physical IoT sensors, this system uses a decoupled **Software-in-the-Loop (SIL)** simulation.

**Step A (Start the Brain inside Home Assistant):**

1. Copy the contents of `edge_deployment_hub/ai_docker_addon/` into your Home Assistant `/addons/` directory.
2. Install and start the **Energy Efficient RL Agent** Add-on from the Home Assistant Supervisor.
3. Check the logs: It will load the `.pth` weights and idle, listening to the local Mosquitto MQTT broker.

**Step B (Launch the Digital Twin):** Open a local terminal, ensure your virtual environment is active, and execute the environmental daemon. This script acts as the "physical house," streaming the synchronized sensor telemetry over your local network to the Home Assistant instance.

```bash
cd offline_training_hub
python3 src/virtual_house_daemon.py

```

**Step C (Observe Actuation):** As the daemon broadcasts data, watch the Home Assistant dashboards and Add-on logs. The AI will instantly process the 8-Dimensional continuous state tensor, forecast the load, and fire discrete integer commands back via the REST API to physically toggle the virtual HVAC and smart devices in Home Assistant.

*Note: If the agent overcools the house below 17°C, the safety circuit breakers defined in `ha_core_config/automations.yaml` will forcefully override the AI and shut down the system.*


*Developed as part of the 2026 ZJUT Software Engineering Graduation Project. Supervised by Mr. Chenbo.*

```

```
