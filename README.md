# 🚦 Urban Mobility Intelligence & Multi-Agent Traffic Control

> **An end-to-end intelligent transportation systems prototype combining traffic data science, temporal machine learning, graph-based representations, SUMO simulation, and decentralized multi-agent traffic control.**

---

## 📌 Project Overview

Urban transportation networks are dynamic, interconnected systems where decisions made at one intersection can influence traffic conditions across an entire network.

This project explores an end-to-end data-driven framework for **urban traffic intelligence and decentralized traffic-signal control**.

Rather than treating traffic prediction, simulation, and control as isolated tasks, the project connects them into a single engineering pipeline:

```text
Traffic Observations
        ↓
Traffic State Estimation
        ↓
Feature Engineering
        ↓
Temporal Machine Learning
        ↓
Traffic Network Graph
        ↓
SUMO Traffic Simulation
        ↓
Signal Control
        ↓
Multi-Agent Coordination
        ↓
Experimentation
        ↓
Evaluation & Visualization
```

The project is designed as a **research-oriented engineering prototype** that can serve as a foundation for future work in:

- Intelligent Transportation Systems
- Smart Cities
- Graph Neural Networks
- Spatio-Temporal Learning
- Multi-Agent Reinforcement Learning
- Autonomous Systems
- Traffic Signal Optimization
- Data Science Engineering

---

# 🎯 Research Question

The central research question explored by the project is:

> **Can learned traffic-state representations and decentralized control improve traffic-flow objectives compared with fixed or heuristic control policies?**

The project investigates this question progressively rather than jumping directly into reinforcement learning.

The development strategy is:

```text
Statistical / Rule-Based Baselines
              ↓
Temporal Machine Learning
              ↓
Graph Representation
              ↓
Traffic Simulation
              ↓
Single-Agent Control
              ↓
Multi-Agent Control
              ↓
Future Learned MARL
```

This progression allows every stage of the system to be evaluated independently.

---

# 🏙️ Domain

**Smart Cities · Intelligent Transportation Systems · Autonomous Systems · Graph Learning · Multi-Agent AI · Reinforcement Learning · Data Science Engineering**

---

# 🧠 System Architecture

The complete system consists of eight major modules.

```text
┌───────────────────────────────────────────────┐
│          MODULE 01 — DATA ACQUISITION        │
│        METR-LA Traffic Observations          │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│       MODULE 02 — TRAFFIC INTELLIGENCE       │
│   State Estimation & Feature Engineering     │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│      MODULE 03 — SPATIO-TEMPORAL LEARNING    │
│       XGBoost + Traffic Graph Structure      │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│       MODULE 04 — TRAFFIC DIGITAL TWIN       │
│             SUMO + TraCI Simulation          │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│       MODULE 05 — DECISION & CONTROL         │
│          Traffic Signal Control              │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│       MODULE 06 — MULTI-AGENT INTELLIGENCE   │
│     Decentralized Intersection Agents        │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│      MODULE 07 — EXPERIMENTATION & EVALUATION│
│       Reproducible Policy Comparison         │
└──────────────────────┬────────────────────────┘
                       ↓
┌───────────────────────────────────────────────┐
│       MODULE 08 — INTELLIGENCE DELIVERY      │
│       Dashboards & Project Reporting         │
└───────────────────────────────────────────────┘
```

---

# 📊 Dataset

## METR-LA

The project uses the **METR-LA traffic dataset** for the traffic-intelligence portion of the system.

The processed project data contains:

| Property | Value |
|---|---:|
| Traffic observations | 34,272 |
| Traffic sensors | 207 |
| Sampling interval | 5 minutes |
| Time range | 2012-03-01 → 2012-06-27 |
| Missing values | 0 |
| Graph nodes | 207 |
| Graph edges | 1,722 |

The dataset provides a real-world traffic observation foundation for developing traffic-state representations and temporal prediction baselines.

---

# 🧩 Module 01 — Data Acquisition & Quality

The first module establishes the data foundation for the project.

### Responsibilities

- Locate and load the provided dataset files
- Validate traffic observations
- Inspect sensor metadata
- Verify sensor mappings
- Load the traffic-network adjacency matrix
- Check missing values
- Verify temporal sampling frequency
- Establish the project data structure

### Results

```text
Traffic observations : 34,272
Traffic sensors      : 207
Missing values       : 0

Time range:
Start: 2012-03-01 00:00:00
End  : 2012-06-27 23:55:00

Sensor mapping       : 207
Adjacency matrix     : 207 × 207
Sampling interval    : 5 minutes
```

This module ensures that downstream modelling is based on a validated and structurally understood dataset.

---

# 🧠 Module 02 — Traffic Intelligence Engine

The second module transforms raw observations into higher-level traffic-state features.

### Features created

```text
network_mean
network_min
network_max
network_std
hour
day_of_week
is_weekend
congestion_indicator
```

These features provide both:

- **traffic-state information**
- **temporal context**

A congestion indicator was also created using a data-derived threshold.

The resulting representation provides a compact view of network-level traffic conditions and establishes the foundation for predictive modelling.

---

# 📈 Module 03 — Spatio-Temporal Learning

This module establishes the machine-learning and graph-learning foundation.

## Temporal ML Baseline

An **XGBoost** model was implemented as the initial predictive baseline.

### Results

| Metric | Value |
|---|---:|
| MAE | 1.6558 |
| RMSE | 6.3002 |

The XGBoost model serves as a conventional machine-learning benchmark before introducing more complex graph-based architectures.

---

## Traffic Network Graph

The traffic sensors are represented as nodes in a graph.

```text
Nodes  : 207
Edges  : 1,722
```

The adjacency structure captures spatial relationships between traffic sensors.

Conceptually:

```text
Sensor 1 ─── Sensor 2 ─── Sensor 3
    │             │
    │             │
Sensor 4 ─── Sensor 5 ─── Sensor 6
```

This graph representation provides the structural foundation required for future:

- Graph Neural Networks
- Spatio-Temporal GNNs
- Graph attention mechanisms
- Graph-based traffic forecasting
- GNN-enhanced MARL

The current implementation therefore establishes a **GNN-ready graph representation**, rather than claiming a fully trained GNN model.

---

# 🚗 Module 04 — Traffic Digital Twin

The fourth module introduces a simulation environment using:

**SUMO + TraCI**

A controllable **2×2 intersection grid** was constructed.

```text
Intersection 1 ───── Intersection 2
       │                     │
       │                     │
Intersection 3 ───── Intersection 4
```

### Simulation configuration

| Parameter | Value |
|---|---:|
| Network | 2×2 intersection grid |
| Intersections | 4 |
| Environment | SUMO |
| Control interface | TraCI |
| Simulation steps | 1,063 |

The simulation provides a controllable environment in which traffic-signal policies can be tested under reproducible conditions.

---

# 🎛️ Module 05 — Decision & Control Engine

This module introduces traffic-signal control.

The control environment represents traffic conditions using local lane queue information.

### Control design

```text
State
  ↓
Lane queue lengths
  ↓
Traffic signal decision
  ↓
SUMO / TraCI
  ↓
Updated traffic state
  ↓
Reward
```

### Environment characteristics

| Component | Implementation |
|---|---|
| Controlled intersection | 1 |
| State | Lane queue lengths |
| Action | Traffic signal phase |
| Reward | Negative total queue |
| Simulation | SUMO |
| Interface | TraCI |

The environment was designed to be compatible with future reinforcement-learning agents.

---

# 🤖 Module 06 — Multi-Agent Intelligence

The project then extends the control architecture from a single intersection toward decentralized multi-agent control.

Each traffic signal is treated as an independent agent.

```text
       Agent 1
          │
          │
Agent 3 ──┼── Agent 2
          │
          │
       Agent 4
```

### Multi-Agent Configuration

| Component | Value |
|---|---:|
| Traffic-signal agents | 4 |
| Network | 2×2 grid |
| Agent state | Local + neighboring traffic |
| Agent relationships | Spatial graph |
| Control | Decentralized |
| Simulation | SUMO + TraCI |

Each agent has access to local traffic information together with neighboring traffic-state information.

This provides the structural foundation for future multi-agent reinforcement-learning experiments.

---

# ⚠️ Important MARL Scope

The current project **does not claim to have trained a PPO, SAC, MAPPO, or other learned MARL policy**.

The implemented multi-agent controller is a:

> **Decentralized heuristic control policy**

The project instead establishes:

- multi-agent state representation
- spatial agent relationships
- decentralized control structure
- simulation interface
- reward formulation
- evaluation framework

This makes the environment suitable for future experiments with:

```text
PPO
 ↓
Multi-Agent PPO
 ↓
MAPPO-style coordination
 ↓
Graph-enhanced MARL
 ↓
Spatio-Temporal GNN + MARL
```

This distinction is intentional so that the current experimental claims remain reproducible and technically accurate.

---

# 🧪 Module 07 — Experimentation & Evaluation

The experimentation module evaluates multiple traffic-control policies under the same simulation configuration.

### Policies

```text
1. Fixed-Time
2. Cyclic
3. Multi-Agent Heuristic
```

### Experiment configuration

| Parameter | Value |
|---|---:|
| Network | 2×2 SUMO grid |
| Simulation steps | 1,000 |
| Random seed | 42 |
| Intersections | 4 |
| Decision interval | 10 steps |
| Policies | 3 |

---

# 📏 Evaluation Metrics

The system evaluates:

### Average Queue

Average number of queued vehicles observed during the simulation.

### Maximum Queue

Maximum observed queue length.

### Average Waiting Time

Average vehicle waiting time during the simulation.

### Maximum Waiting Time

Maximum observed waiting time.

### Average Vehicles

Average number of vehicles present in the simulation.

### Peak Vehicles

Maximum number of vehicles present simultaneously.

---

# 📊 Control Results

The completed experiment produced the following results:

| Policy | Avg Queue | Max Queue | Avg Waiting | Max Waiting | Avg Vehicles | Peak Vehicles |
|---|---:|---:|---:|---:|---:|---:|
| Fixed-Time | 0.104 | 1 | 0.007 | 3.0 | 6.286 | 11 |
| Cyclic | 0.115 | 2 | 0.025 | 3.0 | 6.324 | 11 |
| Multi-Agent Heuristic | 0.100 | 1 | 0.000 | 0.0 | 6.271 | 11 |

The multi-agent heuristic produced a lower observed average queue than the fixed-time baseline in this particular simulation configuration.

However, these results come from a **single controlled simulation configuration with seed 42**.

Therefore, they should be interpreted as **prototype benchmark results**, not as evidence of generalized superiority.

Future evaluation should include:

- multiple random seeds
- longer simulation horizons
- different traffic-demand scenarios
- larger networks
- statistical confidence intervals
- sensitivity analysis
- trained MARL policies

---

# 📦 Module 08 — Intelligence Delivery

The final module packages the results into reusable project artifacts.

Generated outputs include:

```text
results/
│
├── module_07_control_comparison.csv
├── module_07_control_comparison_with_baseline.csv
├── module_07_experiment_metadata.csv
│
├── module_08_control_performance.png
├── module_08_project_kpi_dashboard.png
├── module_08_traffic_intelligence_summary.png
├── module_08_end_to_end_architecture.png
├── module_08_control_strategy_comparison.png
│
├── module_08_final_metrics_summary.csv
├── module_08_final_project_summary.csv
├── module_08_dashboard_registry.csv
├── module_08_visual_assets.csv
│
└── module_08_final_project_report.txt
```

These artifacts provide both machine-readable experiment outputs and presentation-ready visualizations.

---

# 🖥️ Project Dashboard

A lightweight dashboard was also developed to expose the key project metrics and control-policy results.

The dashboard provides a compact view of:

- traffic observations
- sensor count
- graph size
- intersection count
- XGBoost performance
- control-policy metrics
- overall system pipeline

The dashboard is intended as a **project demonstration layer**, while the underlying CSV and simulation outputs remain the primary experimental artifacts.

---

# 🏗️ Engineering Architecture

The project was intentionally developed as more than a single notebook experiment.

The architecture separates the major responsibilities:

```text
                    ┌───────────────────┐
                    │   Traffic Data    │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Data Validation   │
                    │ & Quality Checks   │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Traffic State     │
                    │ Engineering       │
                    └─────────┬─────────┘
                              │
                    ┌─────────┴──────────┐
                    ▼                    ▼
          ┌─────────────────┐   ┌─────────────────┐
          │ Temporal ML     │   │ Traffic Graph   │
          │ XGBoost         │   │ 207 Nodes       │
          └────────┬────────┘   └────────┬────────┘
                   │                     │
                   └──────────┬──────────┘
                              ▼
                    ┌───────────────────┐
                    │ SUMO Digital Twin │
                    │ + TraCI           │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Signal Control    │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Multi-Agent       │
                    │ Coordination      │
                    └─────────┬─────────┘
                              │
                              ▼
                    ┌───────────────────┐
                    │ Evaluation        │
                    │ & Reporting       │
                    └───────────────────┘
```

---

# 🧰 Technology Stack

## Programming

- Python

## Data Science

- NumPy
- Pandas
- Scikit-learn
- XGBoost

## Deep Learning / Graph Learning

- PyTorch
- PyTorch Geometric
- Graph-based representations

## Reinforcement Learning Foundation

- RL environment design
- PPO-ready state/action interface
- Multi-agent control architecture

## Traffic Simulation

- SUMO
- TraCI

## Development Environment

- Google Colab
- Google Drive

## Visualization

- Matplotlib
- Pandas visualization utilities

---

# 📁 Project Structure

A simplified representation of the project structure is:

```text
Urban_Mobility_Project/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── metadata/
│
├── notebooks/
│   ├── module_01_data_acquisition.ipynb
│   ├── module_02_traffic_intelligence.ipynb
│   ├── module_03_spatiotemporal_learning.ipynb
│   ├── module_04_digital_twin.ipynb
│   ├── module_05_decision_control.ipynb
│   ├── module_06_multi_agent.ipynb
│   ├── module_07_evaluation.ipynb
│   └── module_08_delivery.ipynb
│
├── simulation/
│   ├── network/
│   ├── routes/
│   └── configuration/
│
├── models/
│   ├── baselines/
│   ├── graph/
│   └── reinforcement_learning/
│
├── results/
│   ├── *.csv
│   ├── *.png
│   └── *.txt
│
├── dashboard/
│
├── README.md
└── requirements.txt
```

The exact notebook and directory organization may vary depending on the local/Colab execution environment.

---

# 🔬 Research & Engineering Philosophy

The project follows a progressive modelling strategy.

Instead of immediately applying complex reinforcement learning, each layer is established and evaluated independently.

### Stage 1 — Understand the Data

Traffic observations are validated and converted into meaningful traffic-state features.

### Stage 2 — Establish a Baseline

XGBoost provides a conventional machine-learning benchmark.

### Stage 3 — Introduce Spatial Structure

Traffic sensors are represented as nodes in a graph using the available adjacency structure.

### Stage 4 — Introduce Simulation

SUMO provides a controllable environment for traffic-signal experiments.

### Stage 5 — Introduce Decision Making

Traffic signals become controllable decision points.

### Stage 6 — Introduce Multi-Agent Structure

Multiple intersections become decentralized agents connected through spatial relationships.

### Stage 7 — Evaluate Policies

Control policies are compared under reproducible simulation conditions.

### Stage 8 — Deliver Intelligence

The resulting metrics, figures, summaries, and dashboards are packaged for analysis and communication.

---

# 🔮 Future Research Directions

The current implementation provides several natural directions for further research.

## 1. Spatio-Temporal GNN

Replace or extend the XGBoost baseline with architectures such as:

```text
GCN
GAT
GraphSAGE
ST-GCN
DCRNN
Graph WaveNet
```

The objective would be to jointly model:

- temporal dependencies
- spatial dependencies
- traffic-network structure

---

## 2. Transformer-Based Traffic Forecasting

Investigate temporal attention mechanisms for traffic forecasting.

Potential architectures include:

```text
Temporal Transformer
Spatial-Temporal Transformer
Graph Transformer
```

---

## 3. Trained Reinforcement Learning

The current control environment can be extended toward:

```text
PPO
SAC
MAPPO
Multi-Agent PPO
```

The learned policy would replace the current heuristic controller.

---

## 4. Graph-Enhanced MARL

A particularly interesting research direction is combining:

```text
Traffic Graph
      +
GNN
      +
Multi-Agent RL
      +
SUMO
```

This could allow each traffic-signal agent to incorporate both local observations and learned spatial representations.

---

## 5. Larger Networks

The current 2×2 network is intentionally lightweight.

Future experiments could investigate:

- 3×3 networks
- 4×4 networks
- real road-network topology
- larger intersection counts
- heterogeneous traffic demand

---

## 6. Robust Evaluation

A stronger experimental study would include:

```text
Multiple random seeds
        +
Multiple traffic-demand scenarios
        +
Longer simulations
        +
Confidence intervals
        +
Statistical testing
```

This would make conclusions substantially more robust.

---

# ⚠️ Limitations

This project is a prototype and has several important limitations.

### Simulation Scale

The current SUMO environment is a relatively small 2×2 intersection grid.

### Control Policy

The multi-agent controller is currently heuristic rather than a trained MARL policy.

### Experimental Scope

The control comparison uses a controlled simulation configuration and a fixed random seed.

### Dataset / Simulation Relationship

The METR-LA traffic observations and the SUMO simulation are used as complementary components of the pipeline.

The SUMO environment is **not claimed to be a calibrated digital replica of the METR-LA network**.

### Generalization

The observed policy differences should not be interpreted as universally applicable conclusions about traffic-control strategies.

### GNN Scope

The project establishes a graph representation and GNN-ready pipeline, but does not claim a completed state-of-the-art GNN forecasting model.

---

# 📈 Key Project Statistics

| Category | Value |
|---|---:|
| Traffic observations | 34,272 |
| Traffic sensors | 207 |
| Graph nodes | 207 |
| Graph edges | 1,722 |
| XGBoost MAE | 1.6558 |
| XGBoost RMSE | 6.3002 |
| SUMO intersections | 4 |
| SUMO network | 2×2 |
| Simulation horizon | 1,000 steps |
| Signal agents | 4 |
| Control policies | 3 |
| Random seed | 42 |
| Decision interval | 10 steps |
| Completed modules | 8 |

---

# 🧪 Reproducibility

The primary experimentation uses:

```text
Random seed: 42
Simulation horizon: 1000 steps
Decision interval: 10 steps
Network: 2×2 SUMO grid
Controlled intersections: 4
```

Results and experiment metadata are stored in the `results/` directory.

The project is designed to be reproducible within the same simulation configuration and software environment.

Exact numerical results may vary if simulation configuration, SUMO version, dependencies, random seeds, or traffic demand are changed.

---

# 🚀 Running the Project

The project was developed primarily using **Google Colab**.

The recommended workflow is:

```text
1. Open the project notebook
2. Mount Google Drive
3. Point the project root to Urban_Mobility_Project
4. Load the traffic data
5. Execute Modules 01–03
6. Initialize the SUMO environment
7. Execute Modules 04–06
8. Run Module 07 experiments
9. Generate Module 08 outputs
```

The project does not require Docker for its current implementation.

---

# 📊 Project Outputs

The `results/` directory contains the primary generated artifacts.

### Data outputs

```text
module_07_control_comparison.csv
module_07_control_comparison_with_baseline.csv
module_07_experiment_metadata.csv
module_08_final_metrics_summary.csv
module_08_final_project_summary.csv
```

### Visual outputs

```text
module_08_control_performance.png
module_08_project_kpi_dashboard.png
module_08_traffic_intelligence_summary.png
module_08_end_to_end_architecture.png
module_08_control_strategy_comparison.png
```

### Documentation outputs

```text
module_08_dashboard_registry.csv
module_08_visual_assets.csv
module_08_final_project_report.txt
```

---

# 🎓 Research Positioning

This project is intended as a **portfolio and research-oriented engineering project** demonstrating the ability to connect multiple technical disciplines into a single working system.

The emphasis is not solely on obtaining the most sophisticated model.

Instead, the project demonstrates the engineering progression:

```text
Data
 ↓
Understanding
 ↓
Representation
 ↓
Prediction
 ↓
Simulation
 ↓
Decision Making
 ↓
Coordination
 ↓
Evaluation
 ↓
Delivery
```

This architecture reflects the broader challenge of building intelligent autonomous systems: machine learning models must ultimately interact with environments, make decisions, and be evaluated against measurable objectives.

---

# 💼 Skills Demonstrated

### Data Science

- Data validation
- Feature engineering
- Time-series analysis
- Baseline modelling
- Model evaluation

### Machine Learning

- XGBoost
- Temporal prediction
- Model benchmarking

### Graph Learning

- Graph construction
- Adjacency-based representations
- Spatial relationships
- GNN-ready data structures

### Reinforcement Learning

- State design
- Action design
- Reward design
- Environment construction
- Single-agent control
- Multi-agent control architecture

### Autonomous Systems

- Simulation environments
- Decision loops
- Agent coordination
- Feedback systems

### Data Science Engineering

- Modular pipeline design
- Reproducible experimentation
- Result persistence
- Experiment metadata
- Visualization
- Dashboard delivery
- Simulation integration

---

# 🧭 Overall Project Roadmap

```text
                         CURRENT PROJECT
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Traffic Intelligence│
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Graph Representation│
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ SUMO Digital Twin   │
                   └──────────┬──────────┘
                              │
                              ▼
                   ┌─────────────────────┐
                   │ Heuristic Multi-Agent│
                   │      Control         │
                   └──────────┬──────────┘
                              │
                              ▼
                       EVALUATION
                              │
                              │
                  ────────────┼────────────
                              │
                              ▼
                     FUTURE RESEARCH
                              │
             ┌────────────────┼────────────────┐
             ▼                ▼                ▼
          ST-GNN            MAPPO        Graph-enhanced
                                          MARL
```

---

# 🏁 Final Status

```text
╔══════════════════════════════════════════════════╗
║                                                  ║
║     URBAN MOBILITY INTELLIGENCE PROJECT         ║
║                                                  ║
║     Modules Completed              8 / 8        ║
║     Traffic Sensors                  207         ║
║     Graph Nodes                      207         ║
║     Graph Edges                    1,722         ║
║     SUMO Intersections                 4         ║
║     Control Policies                   3         ║
║     Simulation Steps                1000         ║
║                                                  ║
║            IMPLEMENTATION COMPLETE               ║
║                                                  ║
╚══════════════════════════════════════════════════╝
```

---

# 📜 Disclaimer

This repository represents an independent portfolio and research-oriented engineering project.

The current implementation should be understood as a **simulation-based prototype** rather than a production traffic-management system.

Experimental results are specific to the implemented simulation configuration and should not be interpreted as generalized evidence that one traffic-control strategy is universally superior to another.

The project intentionally distinguishes between implemented functionality and future research directions. In particular, trained PPO/MARL performance and calibrated real-world digital-twin performance are future extensions rather than claims of the current implementation.

---

# 👨‍💻 Project Focus

**Urban Mobility Intelligence & Multi-Agent Traffic Control**

**Core themes:**

`Smart Cities` · `Intelligent Transportation Systems` · `Graph Learning` · `Traffic Forecasting` · `SUMO` · `TraCI` · `Multi-Agent AI` · `Reinforcement Learning` · `Autonomous Systems` · `Data Science Engineering`

---

## ⭐ If You Find This Project Interesting

The project is designed to demonstrate how **data science, graph learning, simulation, and autonomous decision-making can be connected into one engineering pipeline**.

The current implementation provides the foundation for progressively moving from heuristic traffic control toward learned spatio-temporal multi-agent policies.
