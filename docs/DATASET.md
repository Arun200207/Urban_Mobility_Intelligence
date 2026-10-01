# Dataset Documentation

## 1. Dataset Overview

This project uses the **METR-LA traffic dataset** as the primary real-world traffic observation source for traffic-state analysis, temporal forecasting, and traffic-network graph construction.

METR-LA is a widely used benchmark dataset for traffic forecasting research. It contains traffic measurements collected from loop detectors deployed across the Los Angeles highway network.

In this project, METR-LA is used primarily for:

- Traffic observation analysis
- Traffic-state feature engineering
- Temporal forecasting
- Sensor-level traffic representation
- Traffic-network graph construction
- Preparation of graph-based learning experiments

The dataset is **not directly used as a traffic-signal-control dataset**. Signal control and reinforcement-learning experiments are conducted separately in a SUMO simulation environment.

---

## 2. Dataset Role in the Project

The dataset forms the observation and intelligence layer of the project.

The overall relationship is:

```text
METR-LA Traffic Observations
            ↓
Traffic State Estimation
            ↓
Temporal Features
            ↓
XGBoost Forecasting Baseline
            ↓
Sensor Network Graph
            ↓
GNN-Ready Representation
            ↓
SUMO Digital-Twin Prototype
            ↓
Signal Control Experiments
```

This separation is intentional.

METR-LA provides **real-world traffic observations and spatial relationships**, while SUMO provides a **controllable traffic simulation environment** for signal-control experimentation.

The two environments are therefore complementary rather than being a calibrated one-to-one digital twin.

---

# 3. Dataset Files

The downloaded METR-LA package used in this project contains two primary files:

```text
*.h5
*.pkl
```

These files were retained in their original form rather than unnecessarily converting them into another dataset format.

The project stores the raw dataset under:

```text
data/
└── raw/
    ├── <METR-LA HDF5 file>
    └── <METR-LA metadata / graph PKL file>
```

The exact filenames may depend on the source package used to obtain the dataset.

---

# 4. HDF5 Traffic Data

The HDF5 file contains the traffic observations used throughout the traffic-intelligence pipeline.

The dataset was inspected programmatically before modeling.

The resulting traffic observation matrix contains:

| Property | Value |
|---|---:|
| Traffic observations | 34,272 |
| Traffic sensors | 207 |
| Time interval | 5 minutes |
| Number of sensors | 207 |
| Missing values after loading | 0 |
| Start timestamp | 2012-03-01 00:00:00 |
| End timestamp | 2012-06-27 23:55:00 |

The traffic observations are organized by timestamp and sensor.

Conceptually:

```text
                 Sensor 1   Sensor 2   Sensor 3   ...   Sensor 207
Timestamp 1        value      value      value            value
Timestamp 2        value      value      value            value
Timestamp 3        value      value      value            value
...
Timestamp N        value      value      value            value
```

This structure allows the project to analyze both:

1. **Temporal behavior** — how traffic changes over time.
2. **Spatial behavior** — how traffic conditions are distributed across connected sensors.

---

# 5. Temporal Resolution

The dataset uses a **5-minute observation interval**.

The project verified that the timestamp spacing is consistent:

```text
Time difference between observations = 5 minutes
```

Therefore:

```text
12 observations = 1 hour
288 observations = 1 day
```

This temporal structure is used when creating lagged features and forecasting the next traffic state.

---

# 6. Dataset Time Range

The traffic observations used in the project span:

```text
Start:
2012-03-01 00:00:00

End:
2012-06-27 23:55:00
```

The dataset therefore provides several months of continuous traffic observations suitable for temporal analysis.

The project preserves the chronological ordering of the observations.

No random shuffling is used when creating the forecasting train/test split.

---

# 7. Sensor Information

The dataset contains:

```text
207 traffic sensors
```

Each sensor represents a traffic measurement location within the underlying road network.

The project extracts the sensor identifiers from the metadata file and uses them consistently throughout the pipeline.

The sensor metadata includes:

```text
Sensor IDs
Sensor-to-index mapping
Spatial adjacency / graph information
```

The sensor-to-index mapping allows the traffic observations and graph representation to use a consistent node ordering.

---

# 8. Metadata PKL File

The accompanying PKL file contains metadata required to interpret the traffic network.

After loading with:

```python
pickle.load(file, encoding="latin1")
```

the metadata structure contains three primary components:

```text
metadata[0] → sensor IDs
metadata[1] → sensor ID-to-index mapping
metadata[2] → spatial adjacency matrix
```

The resulting structures are:

```text
Sensor IDs:
207

Sensor mapping:
207 entries

Adjacency matrix:
207 × 207
```

The adjacency matrix provides the spatial relationship used to construct the project's traffic network graph.

---

# 9. Traffic Network Graph

The sensor metadata is transformed into a graph representation.

Each traffic sensor becomes a graph node:

```text
Node = Traffic Sensor
```

Connections between sensors are represented as graph edges:

```text
Edge = Spatial relationship between sensors
```

The resulting graph contains:

```text
Nodes: 207
Non-zero connections: 1,722
```

The graph can therefore be represented as:

```text
G = (V, E)
```

where:

```text
|V| = 207
|E| = 1,722
```

The graph is implemented using PyTorch Geometric.

---

# 10. PyTorch Geometric Representation

The graph is converted into a PyTorch Geometric `Data` object.

Conceptually:

```python
graph = Data(
    edge_index=edge_index,
    edge_weight=edge_weight,
    num_nodes=207
)
```

The graph contains:

- Node count
- Edge connectivity
- Edge weights
- Sensor relationships

The edge representation is constructed from the non-zero entries of the adjacency matrix.

The graph is designed to support future graph neural network experiments.

---

# 11. Node Features

The project constructs node-level temporal features from the most recent traffic observations.

For the graph representation, the latest four time points are transposed so that each sensor becomes one graph node with four temporal observations.

The resulting node feature matrix has shape:

```text
(207, 4)
```

Conceptually:

```text
               t-3     t-2     t-1      t
Sensor 1        x       x       x       x
Sensor 2        x       x       x       x
Sensor 3        x       x       x       x
...
Sensor 207      x       x       x       x
```

This representation is intentionally simple.

It establishes the data structure required for future GNN-based traffic representation learning without claiming that a trained GNN is already part of the project.

---

# 12. Traffic-State Features

The raw sensor observations are transformed into network-level traffic-state features.

The project generates:

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

These features provide a compact representation of the overall traffic state.

---

## 12.1 Network Mean

The network mean represents the average observed traffic value across all sensors at a given timestamp.

Conceptually:

```text
network_mean(t)
=
mean(sensor_1(t), sensor_2(t), ..., sensor_207(t))
```

This feature provides a simple global view of network traffic conditions.

---

## 12.2 Network Minimum

The network minimum captures the lowest observed sensor value at a given timestamp.

It can help identify spatial variation within the network.

---

## 12.3 Network Maximum

The network maximum captures the highest observed sensor value.

This helps identify periods where one or more locations experience substantially higher traffic values than the network average.

---

## 12.4 Network Standard Deviation

The network standard deviation captures the spatial variability of traffic observations across the sensors.

Higher values indicate greater differences between sensor measurements.

---

# 13. Temporal Features

Several calendar-based features are derived from the timestamps.

### Hour

The hour of the day:

```text
0–23
```

This captures recurring daily traffic patterns.

### Day of Week

The day of the week is encoded numerically:

```text
0–6
```

This allows weekday and weekend patterns to be distinguished.

### Weekend Indicator

A binary feature identifies weekend observations:

```text
0 = weekday
1 = weekend
```

These temporal features provide basic context for the forecasting baseline.

---

# 14. Congestion Indicator

The project also creates a simple congestion indicator based on a statistical threshold.

The threshold used was:

```text
52.73631958822176
```

The resulting distribution was:

```text
Congestion indicator = 0
25,704 observations

Congestion indicator = 1
8,568 observations
```

The indicator is calculated from the project's traffic-state representation.

### Important Interpretation

This indicator is a **relative statistical traffic-state indicator**.

It should **not** be interpreted as:

- A ground-truth congestion label
- An externally validated congestion classification
- A manually annotated traffic condition
- A traffic incident label

It is used as an exploratory feature within the project.

---

# 15. Forecasting Dataset

For temporal prediction, lagged traffic features are created.

The project uses:

```text
Lag 1
Lag 2
Lag 3
```

along with the current network traffic state.

The target is shifted one time step into the future.

Conceptually:

```text
Current state:
X(t)

Historical states:
X(t-1)
X(t-2)
X(t-3)

Prediction target:
X(t+1)
```

This produces a supervised learning problem:

```text
Past traffic states
        ↓
Temporal ML model
        ↓
Next traffic state
```

---

# 16. Train/Test Split

The forecasting experiment uses a chronological split.

The data is divided approximately as:

```text
80% → Training
20% → Testing
```

The split is performed chronologically rather than randomly.

This is important for time-series modeling because randomly mixing future observations into the training set could introduce temporal leakage.

The project therefore follows:

```text
Past observations
       ↓
Training

Later observations
       ↓
Testing
```

No shuffling is used for the forecasting split.

---

# 17. XGBoost Baseline

The project uses XGBoost as a temporal machine-learning baseline.

The model receives engineered historical traffic features and predicts the next traffic-state value.

The baseline achieved:

| Metric | Result |
|---|---:|
| MAE | 1.6558 |
| RMSE | 6.3002 |

These results provide a reference point for future modeling experiments.

---

# 18. Why XGBoost Is Used

XGBoost provides a strong and computationally practical baseline for structured temporal features.

It is useful for establishing whether basic engineered traffic features already contain predictive information before introducing more complex architectures.

In this project, XGBoost should be interpreted as:

```text
Temporal ML Baseline
```

rather than a full spatio-temporal deep-learning model.

It does not directly perform message passing over the traffic graph.

---

# 19. Relationship Between Dataset and GNN Research

The dataset's sensor topology makes it suitable for graph-based traffic modeling.

The project therefore prepares:

```text
Traffic observations
        +
Sensor graph
        +
Temporal features
        ↓
GNN-ready representation
```

The current implementation stops at graph construction and representation preparation.

Future work could introduce:

```text
Graph Convolution
Graph Attention
Temporal GNN
Spatio-Temporal GNN
Graph-based forecasting
Graph-enhanced MARL
```

These are future research directions rather than completed components of the current implementation.

---

# 20. Relationship Between METR-LA and SUMO

A major architectural distinction in this project is the separation between the real-world dataset and the simulation environment.

### METR-LA

Used for:

- Traffic observations
- Traffic-state analysis
- Temporal forecasting
- Sensor relationships
- Graph construction
- Real-world traffic intelligence

### SUMO

Used for:

- Traffic simulation
- Signal control
- TraCI interaction
- Reinforcement-learning environment design
- Multi-agent control experiments
- Policy comparison

The relationship is therefore:

```text
METR-LA
Real-world traffic intelligence
        ↓
Traffic representation / learning
        ↓
Research architecture
        ↓
SUMO
Controllable simulation environment
        ↓
Signal-control experimentation
```

---

# 21. Digital-Twin Scope

The SUMO environment should be described as a:

> **Digital-twin-style traffic simulation prototype**

rather than a calibrated replica of the METR-LA road network.

The current SUMO experiment uses a generated:

```text
2 × 2 intersection grid
```

with:

```text
4 controlled intersections
```

The network is intentionally lightweight so that the control architecture can be tested efficiently.

The SUMO environment is therefore not a one-to-one reconstruction of the Los Angeles sensor network represented by METR-LA.

---

# 22. Data-to-Control Boundary

METR-LA and SUMO are connected conceptually but are not directly merged into a single calibrated simulation.

The current architecture is:

```text
                 ┌──────────────────────┐
                 │      METR-LA         │
                 │ Real-world traffic   │
                 │ observations         │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Traffic Intelligence │
                 │ Features + Forecast  │
                 │ + Graph              │
                 └──────────────────────┘


                 ┌──────────────────────┐
                 │        SUMO          │
                 │ Traffic simulation   │
                 │ + signal control     │
                 └──────────┬───────────┘
                            │
                            ▼
                 ┌──────────────────────┐
                 │ Decision & Control   │
                 │ Multi-agent control  │
                 └──────────────────────┘
```

This separation prevents the project from making unsupported claims about direct real-world deployment.

---

# 23. Data Quality Checks

The project performs basic dataset validation before downstream processing.

The checks include:

- File discovery
- File loading
- HDF5 inspection
- Metadata inspection
- Sensor count verification
- Timestamp inspection
- Missing-value checking
- Adjacency matrix shape verification
- Sensor mapping verification
- Temporal interval verification

The final dataset checks confirmed:

```text
Observations: 34,272
Sensors: 207
Missing values: 0
Timestamp interval: 5 minutes
Adjacency matrix: 207 × 207
Sensor IDs: 207
Sensor mapping entries: 207
```

---

# 24. Data Leakage Considerations

The forecasting pipeline preserves chronological ordering.

The project avoids randomly shuffling the time series before the train/test split.

The temporal forecasting setup is therefore:

```text
Earlier observations → Training
Later observations   → Testing
```

This is more appropriate for evaluating a forecasting system than a conventional random train/test split.

Future experiments should continue to consider:

- Temporal leakage
- Feature leakage
- Future information accidentally entering features
- Sensor availability differences
- Distribution shifts over time

---

# 25. Data Preprocessing Philosophy

The project intentionally keeps preprocessing relatively transparent.

Rather than applying a large number of opaque transformations, the pipeline focuses on:

```text
Load
 ↓
Validate
 ↓
Inspect
 ↓
Engineer traffic-state features
 ↓
Create temporal features
 ↓
Create lagged features
 ↓
Build graph representation
```

This makes the research prototype easier to understand and reproduce.

---

# 26. Dataset Statistics Used in the Final Project

The key dataset-related statistics are:

| Statistic | Value |
|---|---:|
| Dataset | METR-LA |
| Traffic observations | 34,272 |
| Traffic sensors | 207 |
| Temporal resolution | 5 minutes |
| Missing values | 0 |
| Graph nodes | 207 |
| Graph connections | 1,722 |
| Adjacency matrix | 207 × 207 |
| Forecasting split | 80 / 20 chronological |
| XGBoost MAE | 1.6558 |
| XGBoost RMSE | 6.3002 |

---

# 27. Reproducibility

The dataset-processing workflow was implemented in Google Colab.

The project stores its working data under:

```text
Urban_Mobility_Project/
└── data/
    └── raw/
```

The raw dataset files are copied into the project workspace while preserving the original downloaded files.

This approach provides two benefits:

1. The original dataset remains untouched.
2. The project has a consistent raw-data location.

The processing pipeline can therefore be rerun without manually reconstructing the directory structure.

---

# 28. Data Directory Structure

The expected project structure is:

```text
urban-mobility-intelligence/
│
├── data/
│   └── raw/
│       ├── METR-LA HDF5 file
│       └── METR-LA metadata PKL file
│
├── notebooks/
│   ├── module_01_data_acquisition.ipynb
│   ├── module_02_traffic_intelligence.ipynb
│   ├── module_03_spatiotemporal_learning.ipynb
│   ├── module_04_digital_twin.ipynb
│   ├── module_05_decision_control.ipynb
│   ├── module_06_multi_agent.ipynb
│   ├── module_07_experimentation.ipynb
│   └── module_08_intelligence_delivery.ipynb
│
├── results/
│
├── docs/
│
└── README.md
```

---

# 29. Data Privacy and Licensing Considerations

The dataset should be obtained and redistributed according to the terms of its original source and associated dataset license.

This repository documentation describes the dataset and the processing performed by the project.

If the raw dataset is not permitted to be redistributed through the repository, it should **not** be committed to GitHub.

Instead, users should obtain the dataset from the appropriate original source and place the required files into:

```text
data/raw/
```

The repository should therefore contain code and documentation describing how the data is used without unnecessarily committing large or externally licensed raw files.

---

# 30. What This Dataset Does Not Contain for This Project

The METR-LA data used here should not be described as containing:

- Traffic-signal action labels
- Reinforcement-learning actions
- PPO training trajectories
- Multi-agent reward signals
- Ground-truth signal-control policies
- SUMO simulation states
- Intersection phase decisions
- Direct traffic-light optimization targets

Those components are generated or evaluated separately within the SUMO control environment.

---

# 31. Current Dataset Limitations

Several limitations should be considered when interpreting the project.

### 31.1 Historical Dataset

The traffic observations represent a historical period rather than live traffic conditions.

### 31.2 Sensor Coverage

The dataset represents a fixed sensor network and does not capture every road or intersection in a city.

### 31.3 Sensor-Level Observations

Traffic observations are measurements from sensors rather than complete vehicle trajectories.

### 31.4 No Direct Signal-Control Labels

The dataset does not provide the signal-control actions required to directly train the project's traffic-light agents.

### 31.5 Simulation Separation

The SUMO network is not calibrated directly from the METR-LA road geometry.

### 31.6 Limited Control Experiment

The current control experiment uses a small 2×2 network and a lightweight traffic demand configuration.

### 31.7 Congestion Indicator

The congestion indicator is a project-defined statistical feature rather than a validated ground-truth congestion label.

---

# 32. Future Dataset Extensions

Future versions of the project could integrate additional data sources such as:

```text
Traffic signal timing data
Vehicle trajectory data
Road geometry
Incident data
Weather data
Public transit data
GPS probe data
Real-time traffic APIs
Intersection-level detector data
Connected-vehicle observations
```

This would enable richer relationships between:

```text
Traffic state
+
Road topology
+
Signal state
+
External conditions
+
Vehicle movement
```

Such integration would support more realistic spatio-temporal modeling and traffic-control research.

---

# 33. Recommended Future Data Pipeline

A more advanced version of the system could follow:

```text
Real-World Sensors
        │
        ├── Traffic Speed
        ├── Flow
        ├── Occupancy
        └── Travel Time
        │
        ▼
Road Network Graph
        │
        ├── Road topology
        ├── Sensor relationships
        ├── Intersection relationships
        └── Signal relationships
        │
        ▼
Spatio-Temporal Representation
        │
        ▼
GNN / Temporal GNN
        │
        ▼
Traffic Forecast
        │
        ▼
Multi-Agent Controller
        │
        ▼
Traffic Simulation
        │
        ▼
Evaluation
```

This architecture is a natural extension of the current dataset and graph representation.

---

# 34. Dataset Usage Summary

The dataset contributes three major capabilities to the project:

### 1. Traffic Intelligence

The raw observations are converted into interpretable traffic-state features.

### 2. Temporal Prediction

Lagged observations are used to train and evaluate an XGBoost forecasting baseline.

### 3. Spatial Representation

The sensor metadata is converted into a graph representation containing:

```text
207 nodes
1,722 connections
```

This provides the foundation for future graph neural network research.

---

# 35. Final Dataset Positioning

The role of METR-LA in this project can be summarized as:

```text
METR-LA
   │
   ├── Real-world traffic observations
   │
   ├── 207 traffic sensors
   │
   ├── Temporal traffic intelligence
   │
   ├── Forecasting baseline
   │
   └── Traffic sensor graph
             │
             ▼
       GNN-ready representation
```

The dataset therefore forms the **real-world observation and representation layer** of the Urban Mobility Intelligence system.

The project then extends beyond the dataset into a separate SUMO-based simulation layer for controllable traffic-signal experiments.

This separation keeps the architecture reproducible and prevents the project from overstating what the METR-LA dataset itself provides.

---

## 36. Dataset-to-Project Mapping

| Project Component | Dataset Contribution |
|---|---|
| Module 01 — Data Acquisition | Raw HDF5 + PKL files |
| Module 02 — Traffic Intelligence | Sensor observations + timestamps |
| Module 03 — Spatio-Temporal Learning | Temporal features + sensor graph |
| Module 04 — Digital Twin | Dataset provides traffic-intelligence context; SUMO provides simulation |
| Module 05 — Decision & Control | SUMO-generated control environment |
| Module 06 — Multi-Agent Intelligence | SUMO network + traffic states |
| Module 07 — Experimentation | SUMO simulation results |
| Module 08 — Intelligence Delivery | Aggregates dataset, ML, graph, simulation, and evaluation outputs |

---

## 37. Final Dataset Statistics

```text
Dataset:
METR-LA

Traffic observations:
34,272

Traffic sensors:
207

Temporal resolution:
5 minutes

Time range:
2012-03-01 00:00:00
to
2012-06-27 23:55:00

Missing values:
0

Sensor metadata:
207 IDs

Sensor mapping:
207 entries

Adjacency matrix:
207 × 207

Graph nodes:
207

Graph connections:
1,722

Forecasting model:
XGBoost

Chronological split:
80% train / 20% test

XGBoost MAE:
1.6558

XGBoost RMSE:
6.3002
```

---

## 38. One-Line Summary

> **METR-LA provides the project's real-world traffic observations, temporal forecasting foundation, and sensor-network graph, while SUMO provides the separate controllable simulation environment used for traffic-signal and multi-agent control experiments.**
