# System Architecture

## Urban Mobility Intelligence & Multi-Agent Traffic Control

This document describes the technical architecture of the **Urban Mobility Intelligence & Multi-Agent Traffic Control** project.

The system is designed as an end-to-end intelligent transportation pipeline connecting:

- Real-world traffic observations
- Traffic-state engineering
- Temporal machine learning
- Traffic-network graph representation
- Traffic simulation
- Traffic-signal control
- Decentralized multi-agent coordination
- Experimental evaluation
- Result visualization and delivery

The architecture follows a progressive design in which each layer provides the foundation for the next.

---

# 1. High-Level Architecture

The overall system can be represented as:

```text
┌─────────────────────────────────────────────────────────────┐
│                    REAL-WORLD TRAFFIC DATA                 │
│                         METR-LA                             │
│                                                             │
│              34,272 observations · 207 sensors             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│              MODULE 01 — DATA ACQUISITION                  │
│                                                             │
│   Loading · Validation · Quality Checks · Metadata          │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│           MODULE 02 — TRAFFIC INTELLIGENCE ENGINE          │
│                                                             │
│   Traffic-State Features · Temporal Patterns · Indicators  │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│          MODULE 03 — SPATIO-TEMPORAL LEARNING              │
│                                                             │
│       XGBoost Temporal Baseline + Traffic Graph            │
│                                                             │
│       207 Nodes · 1,722 Graph Connections                  │
└──────────────────────────────┬──────────────────────────────┘
                               │
                 ┌─────────────┴──────────────┐
                 │                            │
                 ▼                            ▼
        ┌─────────────────┐          ┌────────────────────┐
        │ Temporal ML     │          │ Traffic Graph      │
        │ XGBoost         │          │ Representation     │
        └────────┬────────┘          └─────────┬──────────┘
                 │                             │
                 └──────────────┬──────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────┐
│             MODULE 04 — TRAFFIC DIGITAL TWIN               │
│                                                             │
│                    SUMO + TraCI                            │
│                                                             │
│                 2×2 Intersection Grid                      │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│           MODULE 05 — DECISION & CONTROL ENGINE             │
│                                                             │
│              Traffic-Signal Control Loop                   │
│                                                             │
│      Observe → Decide → Act → Simulate → Reward             │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│            MODULE 06 — MULTI-AGENT INTELLIGENCE             │
│                                                             │
│       4 Decentralized Traffic-Signal Agents                │
│                                                             │
│       Local State + Neighboring Traffic State              │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│         MODULE 07 — EXPERIMENTATION & EVALUATION            │
│                                                             │
│ Fixed-Time · Cyclic · Multi-Agent Heuristic                 │
│                                                             │
│ Queue · Waiting Time · Vehicle Counts                       │
└──────────────────────────────┬──────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────┐
│             MODULE 08 — INTELLIGENCE DELIVERY               │
│                                                             │
│        Metrics · CSV Outputs · Visualizations               │
│        Dashboard · Project Reports                          │
└─────────────────────────────────────────────────────────────┘
```

---

# 2. Architectural Principles

The system follows several core principles.

## 2.1 Progressive Complexity

The project does not begin with reinforcement learning.

Instead, complexity increases progressively:

```text
Data
  ↓
Features
  ↓
Prediction
  ↓
Graph Representation
  ↓
Simulation
  ↓
Control
  ↓
Multi-Agent Control
  ↓
Evaluation
```

This makes it possible to establish meaningful baselines before introducing more complex learning methods.

---

## 2.2 Separation of Intelligence and Control

The project separates:

### Traffic Intelligence

Understanding and predicting traffic conditions.

```text
Traffic observations
        ↓
Feature engineering
        ↓
Temporal prediction
        ↓
Graph representation
```

from:

### Traffic Control

Acting on the simulated environment.

```text
Traffic state
      ↓
Decision
      ↓
Signal action
      ↓
SUMO
      ↓
Updated traffic state
```

This separation allows future learned traffic representations to be connected to different control strategies.

---

## 2.3 Simulation as a Controlled Environment

Real-world traffic data is used for traffic intelligence and representation learning.

SUMO is used as the controllable environment for traffic-signal experimentation.

The two components therefore serve complementary purposes.

```text
METR-LA
   │
   ├── Traffic-state understanding
   ├── Temporal modelling
   └── Graph representation
           
SUMO
   │
   ├── Traffic simulation
   ├── Signal control
   ├── Multi-agent decisions
   └── Policy evaluation
```

The current architecture does **not** claim that the SUMO network is a calibrated replica of the METR-LA road network.

---

# 3. Module Architecture

## Module 01 — Data Acquisition & Quality

### Purpose

Establish a validated traffic-data foundation.

### Inputs

- METR-LA traffic observations
- Sensor metadata
- Sensor-to-index mapping
- Adjacency matrix

### Processing

```text
Raw Files
   ↓
Load
   ↓
Validate
   ↓
Inspect
   ↓
Quality Checks
   ↓
Validated Traffic Dataset
```

### Outputs

- Traffic observations
- Sensor identifiers
- Sensor mapping
- Adjacency matrix
- Data-quality statistics

### Key characteristics

```text
Observations : 34,272
Sensors      : 207
Missing      : 0
Interval     : 5 minutes
Graph        : 207 × 207
```

---

# 4. Module 02 — Traffic Intelligence Engine

## Purpose

Transform raw sensor measurements into meaningful traffic-state information.

### Feature Pipeline

```text
Sensor Measurements
        ↓
Network Aggregation
        ↓
Statistical Features
        ↓
Temporal Features
        ↓
Traffic-State Representation
```

### Generated Features

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

These features provide both traffic-state information and temporal context.

---

# 5. Module 03 — Spatio-Temporal Learning

This module contains two complementary components:

```text
                 Traffic State
                      │
             ┌────────┴────────┐
             │                 │
             ▼                 ▼
        Temporal ML       Spatial Graph
             │                 │
             ▼                 ▼
          XGBoost          PyG Data
```

---

## 5.1 Temporal Prediction

An XGBoost regressor is used as the initial temporal machine-learning baseline.

The model uses lagged traffic information to predict future network traffic conditions.

### Conceptual flow

```text
Traffic(t-3)
     +
Traffic(t-2)
     +
Traffic(t-1)
     +
Traffic(t)
     ↓
XGBoost
     ↓
Traffic(t+1)
```

### Evaluation

```text
MAE  = 1.6558
RMSE = 6.3002
```

The XGBoost model provides a conventional machine-learning baseline against which future neural forecasting architectures can be compared.

---

# 6. Traffic Network Graph

The project uses the available traffic-network adjacency information to construct a graph.

### Graph Definition

```text
G = (V, E)
```

where:

- `V` represents traffic sensors
- `E` represents spatial relationships between sensors

### Graph Statistics

```text
Nodes : 207
Edges : 1,722
```

The graph is represented using **PyTorch Geometric** data structures.

Conceptually:

```text
Sensor A ───── Sensor B
   │              │
   │              │
Sensor C ───── Sensor D
```

The graph representation is designed to support future spatial and spatio-temporal learning architectures.

---

# 7. Module 04 — Traffic Digital Twin

## Purpose

Provide a controllable simulation environment for traffic-control experiments.

### Technology

```text
SUMO
 +
TraCI
```

### Network

```text
2 × 2 intersection grid
```

with:

```text
4 traffic-signal intersections
```

### Simulation flow

```text
SUMO
 │
 ├── Road network
 ├── Vehicles
 ├── Traffic signals
 └── Traffic dynamics
       │
       ▼
     TraCI
       │
       ▼
 External Python Controller
```

TraCI provides the interface through which Python can observe and modify the simulation.

---

# 8. Module 05 — Decision & Control Engine

The control environment follows a closed-loop decision process.

```text
┌──────────────┐
│ Traffic State│
└──────┬───────┘
       ↓
┌──────────────┐
│   Decision   │
└──────┬───────┘
       ↓
┌──────────────┐
│ Signal Phase │
└──────┬───────┘
       ↓
┌──────────────┐
│     SUMO     │
└──────┬───────┘
       ↓
┌──────────────┐
│ New Traffic  │
│    State     │
└──────┬───────┘
       ↓
┌──────────────┐
│    Reward    │
└──────┬───────┘
       │
       └──────────→ Next Decision
```

---

# 9. State Representation

The current control prototype uses lane-level queue information as its primary state representation.

Conceptually:

```text
Intersection
      │
      ├── Incoming Lane 1 → Queue
      ├── Incoming Lane 2 → Queue
      ├── Incoming Lane 3 → Queue
      └── Incoming Lane 4 → Queue
```

This provides a compact representation of immediate traffic pressure around an intersection.

---

# 10. Action Representation

The traffic-signal controller selects among available signal phases.

```text
State
  ↓
Phase Selection
  ↓
TraCI
  ↓
SUMO Traffic Signal
```

The current multi-agent controller uses decentralized phase decisions.

---

# 11. Reward Design

The control prototype uses:

```text
Reward = - Total Queue
```

Therefore:

```text
Lower Queue → Higher Reward
Higher Queue → Lower Reward
```

For example:

```text
Queue = 0
Reward = 0

Queue = 5
Reward = -5
```

Negative rewards are therefore intentional and reflect the minimization objective.

---

# 12. Module 06 — Multi-Agent Architecture

The multi-agent system models each traffic intersection as an independent control agent.

For the current 2×2 network:

```text
Agent 1 ───── Agent 2
   │              │
   │              │
Agent 3 ───── Agent 4
```

The agent relationships follow the spatial topology of the intersection network.

---

# 13. Agent State

Each agent receives:

```text
Local Traffic State
        +
Neighbor Traffic State
        ↓
Combined Agent Observation
```

Conceptually:

```text
              Neighbor
                 │
                 ▼
        ┌─────────────────┐
        │                 │
Neighbor│  Local Agent    │Neighbor
        │                 │
        └─────────────────┘
                 │
                 ▼
          Signal Decision
```

This design allows an agent to make decisions using both its own traffic conditions and information from neighboring intersections.

---

# 14. Decentralized Control

The current multi-agent architecture is decentralized.

Instead of a single centralized controller deciding the phase for every intersection:

```text
             Central Controller
              /      |      \
             /       |       \
         Agent 1   Agent 2   Agent 3
```

the architecture allows:

```text
Agent 1 → Decision 1
Agent 2 → Decision 2
Agent 3 → Decision 3
Agent 4 → Decision 4
```

with each agent using its own local and neighboring traffic information.

This structure is intended to provide a foundation for future decentralized MARL research.

---

# 15. Current Multi-Agent Policy

The current implementation uses a **decentralized heuristic policy**.

The heuristic observes traffic conditions and selects the signal phase associated with the largest current queue.

This is important architecturally because it allows the multi-agent control loop to be evaluated before introducing a learned policy.

The current architecture therefore separates:

```text
Multi-Agent Environment
        ≠
Trained MARL Policy
```

The environment and coordination structure are implemented, while learned MARL remains a future extension.

---

# 16. Module 07 — Experimentation Architecture

The evaluation layer runs multiple policies under a common simulation configuration.

```text
                 SUMO Environment
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
     Fixed-Time      Cyclic      Multi-Agent
                                  Heuristic
          │            │            │
          └────────────┼────────────┘
                       ▼
                 Metrics Engine
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      Queue        Waiting Time    Vehicles
                       │
                       ▼
                Results / CSV
```

---

# 17. Evaluation Metrics

The evaluation layer measures:

```text
Average Queue
Maximum Queue

Average Waiting Time
Maximum Waiting Time

Average Vehicles
Peak Vehicles
```

These metrics provide complementary views of traffic-system behavior.

---

# 18. Experiment Configuration

The primary control experiment uses:

```text
Network              : 2×2 SUMO grid
Intersections        : 4
Simulation steps     : 1,000
Random seed           : 42
Decision interval    : 10 steps
Policies              : 3
```

The evaluated policies are:

```text
1. Fixed-Time
2. Cyclic
3. Multi-Agent Heuristic
```

---

# 19. Module 08 — Intelligence Delivery

The final layer packages experimental outputs into reusable artifacts.

### Output types

```text
CSV
 ↓
Metrics
 ↓
Visualizations
 ↓
Dashboard
 ↓
Project Report
```

### Generated visual assets include

```text
module_08_control_performance.png
module_08_project_kpi_dashboard.png
module_08_traffic_intelligence_summary.png
module_08_end_to_end_architecture.png
module_08_control_strategy_comparison.png
```

These outputs provide a presentation layer over the underlying experimental results.

---

# 20. End-to-End Data Flow

The complete data and decision flow is:

```text
                    ┌──────────────┐
                    │   METR-LA    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Data Quality │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Traffic State│
                    └──────┬───────┘
                           │
                ┌──────────┴──────────┐
                │                     │
                ▼                     ▼
        ┌──────────────┐      ┌──────────────┐
        │ XGBoost      │      │ Graph        │
        │ Prediction   │      │ Structure    │
        └──────┬───────┘      └──────┬───────┘
               │                     │
               └──────────┬──────────┘
                          │
                          ▼
                   ┌──────────────┐
                   │ SUMO + TraCI │
                   └──────┬───────┘
                          │
                          ▼
                   ┌──────────────┐
                   │ Traffic State│
                   └──────┬───────┘
                          │
                          ▼
                   ┌──────────────┐
                   │ Signal Agent │
                   └──────┬───────┘
                          │
                          ▼
                  ┌─────────────────┐
                  │ Neighbor Agents │
                  └────────┬────────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Signal Action│
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ SUMO Update  │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │   Metrics    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ Results      │
                    │ & Dashboard  │
                    └──────────────┘
```

---

# 21. Feedback Loop

The control system is fundamentally a feedback system.

```text
          ┌─────────────────────────────┐
          │                             │
          ▼                             │
     Traffic State                      │
          │                             │
          ▼                             │
    Agent Decision                      │
          │                             │
          ▼                             │
     Signal Action                     │
          │                             │
          ▼                             │
        SUMO                            │
          │                             │
          ▼                             │
   New Traffic State ──────────────────┘
```

The controller continuously observes the simulated environment and responds to the resulting traffic conditions.

This closed-loop structure is important for future reinforcement-learning experiments because the environment naturally provides:

```text
Observation → Action → Transition → Reward
```

---

# 22. Current Architecture vs Future Architecture

## Current

```text
Traffic Data
     ↓
Feature Engineering
     ↓
XGBoost Baseline
     +
Traffic Graph
     ↓
SUMO
     ↓
Heuristic Control
     ↓
Decentralized Multi-Agent Control
     ↓
Evaluation
```

## Future

```text
Traffic Data
     ↓
Spatio-Temporal Representation
     ↓
GNN / Graph Transformer
     ↓
SUMO
     ↓
Learned Policy
     ↓
MAPPO / Multi-Agent RL
     ↓
Graph-Enhanced Coordination
     ↓
Robust Multi-Scenario Evaluation
```

The architecture is therefore designed to evolve without replacing the entire pipeline.

---

# 23. Scalability Path

The current network is intentionally small.

The architecture can progressively scale:

```text
2×2 Network
    ↓
3×3 Network
    ↓
4×4 Network
    ↓
Larger Urban Network
    ↓
Realistic Road Network
```

As the network grows, the graph-based representation becomes increasingly relevant because each intersection can be represented as an agent connected to neighboring agents.

---

# 24. Research Extension Architecture

A future graph-enhanced MARL system could follow:

```text
Traffic Sensors
      │
      ▼
Traffic-State Encoder
      │
      ▼
Spatio-Temporal GNN
      │
      ▼
Node Embeddings
      │
      ▼
Multi-Agent Policy
      │
      ▼
Traffic Signal Actions
      │
      ▼
SUMO
      │
      ▼
Traffic Feedback
      │
      └───────────────┐
                      ▼
                  Reward
                      │
                      ▼
              Policy Update
```

This would connect the project's existing graph representation with learned decentralized control.

---

# 25. Reproducibility

The architecture is designed around explicit experiment configuration.

Important parameters include:

```text
Random seed
Simulation duration
Decision interval
Network topology
Traffic demand
Control policy
Evaluation metrics
```

The current primary experiment uses:

```text
Seed             : 42
Simulation steps : 1,000
Decision interval: 10
Network          : 2×2
Agents           : 4
Policies         : 3
```

Experiment metadata is persisted alongside the evaluation results.

---

# 26. Design Limitations

The current architecture has several deliberate limitations.

### Small Simulation Network

The current SUMO network contains four controlled intersections.

### Heuristic Multi-Agent Controller

The current decentralized policy is not a learned MARL policy.

### Limited Experimental Diversity

The primary benchmark uses a single random seed and controlled demand configuration.

### No Real-Time Deployment

The current system is a research/simulation prototype rather than a live traffic-management platform.

### No Calibrated Digital Twin

The SUMO network is not claimed to be a calibrated representation of the METR-LA physical road network.

These limitations define the boundary between the current implementation and future research.

---

# 27. Summary

The architecture integrates multiple layers of intelligent transportation engineering:

```text
┌──────────────────────────────────────────┐
│              DATA LAYER                  │
│          METR-LA Traffic Data            │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│          INTELLIGENCE LAYER              │
│     Feature Engineering + XGBoost        │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│             GRAPH LAYER                  │
│       207 Nodes + 1,722 Edges            │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│           SIMULATION LAYER               │
│             SUMO + TraCI                │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│            CONTROL LAYER                 │
│       Signal Decision & Feedback         │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│         MULTI-AGENT LAYER                │
│      4 Decentralized Agents              │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│          EVALUATION LAYER                │
│ Queue · Waiting · Vehicles · Throughput   │
└────────────────────┬─────────────────────┘
                     ↓
┌──────────────────────────────────────────┐
│           DELIVERY LAYER                 │
│   Results · Visualizations · Dashboard   │
└──────────────────────────────────────────┘
```

The resulting system provides a complete foundation for progressing from **traffic data intelligence** toward **graph-based multi-agent traffic control**.

The current implementation establishes the complete engineering pipeline while leaving trained spatio-temporal GNNs, PPO/MAPPO policies, larger-scale networks, and robust multi-seed evaluation as clearly defined future extensions.
