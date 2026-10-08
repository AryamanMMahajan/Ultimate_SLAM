# Ultimate SLAM

> From raw KITTI sensor streams to a globally consistent 3D map: classical SLAM built by hand, then capped with production-grade RTAB-Map.

![SLAM Demo](assets/Screencastfrom2026-10-0813-32-05-ezgif.com-video-to-gif-converter.gif)

---

## Overview

A ground-up **Simultaneous Localization and Mapping (SLAM)** project built in three stages on the
[KITTI](https://www.cvlibs.net/datasets/kitti/) autonomous-driving dataset. Each stage implements one
classical piece of a SLAM system: front-end odometry, then a graph-optimization back-end and the final stage runs the industrial **RTAB-Map** library to do the entire stack online at once.

Significance: Every autonomous system faces the same problem: to know where it is, it needs a map and to build a map, it needs to know where it is. SLAM (Simultaneous Localization and Mapping) solves both at once, estimating the robot's trajectory while reconstructing the world around it from nothing but onboard sensors. The result is an end-to-end, sensor-to-map pipeline that spans both the fundamentals and the real deployment: classical geometry, modern learned features, graph optimization, and a production SLAM system, all reproducible inside a single Docker image on ROS 2.

---

## Pipeline

```mermaid
flowchart LR
    SENS["KITTI sensors<br/>(stereo cameras + LiDAR)"] --> ODO
    subgraph ODO["1 · Odometry (front-end)"]
        LID["LiDAR: point-to-plane ICP"]
        VIS["Visual: SuperPoint → SuperGlue → PnP"]
    end
    ODO --> MAP
    subgraph MAP["2 · Graph SLAM (back-end)"]
        LC["loop closure<br/>(VisualHashMap)"] --> G2O["g2o pose-graph<br/>optimization (LM)"]
    end
    MAP --> RT
    subgraph RT["3 · RTAB-Map (capstone, online)"]
        VO["stereo VO"] --> BOW["bag-of-words<br/>loop closure"] --> OPT["graph opt"] --> DMAP["dense 3D map"]
    end
    RT --> OUT["globally consistent<br/>trajectory + 3D map (.db)"]
```

The system is composed of three sequential stages:

**1. Front-End Odometry** : `odometry/`
- Estimates relative frame-to-frame motion, two independent ways
- **LiDAR:** point-to-plane ICP on point clouds (Open3D), chained into a dead-reckoned pose
- **Visual:** SuperPoint → SuperGlue → (LiDAR-depth) → PnP for camera pose
- No loop closure - drift accumulates by design, motivating Stage 2

**2. Graph-SLAM Back-End** : `mapping/`
- Fuses camera + LiDAR observations into landmarks
- Detects loop closures with a spatial-grid landmark store (**VisualHashMap**)
- Optimizes a g2o pose graph with Levenberg–Marquardt → globally consistent trajectory

**3. RTAB-Map Capstone** : `rtabmap/`
- Feeds KITTI stereo to the production **RTAB-Map** SLAM system
- Performs stereo visual odometry, **appearance-based loop closure** (bag-of-words),
  pose-graph optimization, and dense 3D mapping online and together
- The production version of everything hand-built in Stages 1-2

---

## Key Results

| Stage | Technique | Loop closure | Output |
|---|---|:---:|---|
| 1a · LiDAR odometry | Point-to-plane ICP (Open3D) | ✗ | Dead-reckoned pose (drifts) |
| 1b · Visual odometry | SuperPoint + SuperGlue + PnP | ✗ | Camera pose (drifts) |
| 2 · Graph SLAM | VisualHashMap + g2o (Levenberg–Marquardt) | ✓ | Globally consistent trajectory |
| 3 · RTAB-Map | Stereo VO + bag-of-words + graph optimization | ✓ | Online dense 3D map (`.db`) |

<!-- Drag your RTAB-Map screenshots into the GitHub web editor here to embed them: -->
<!-- ![RTAB-Map 3D map](assets/rtabmap_map.png) -->
<!-- ![Trajectory + loop closures](assets/rtabmap_trajectory.png) -->

---

## Tech Stack

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![ROS 2](https://img.shields.io/badge/ROS_2_Humble-22314E?style=for-the-badge&logo=ros&logoColor=white)
![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=for-the-badge&logo=pytorch&logoColor=white)
![CUDA](https://img.shields.io/badge/CUDA-76B900?style=for-the-badge&logo=nvidia&logoColor=white)
![OpenCV](https://img.shields.io/badge/OpenCV-5C3EE8?style=for-the-badge&logo=opencv&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white)

**SLAM:** Point-to-plane ICP · SuperPoint + SuperGlue · PnP / RANSAC · g2o pose-graph optimization · Bag-of-words loop closure · RTAB-Map

**Point clouds / vision:** Open3D · OpenCV · PyTorch (SuperPoint/SuperGlue, LightGlue)

**Middleware:** ROS 2 Humble · `tf2` · `message_filters` · rosbag2 (sqlite3)

**Dataset:** KITTI (sequence `2011_10_03`)

**Visualization:** RViz2 · Plotly

---

## Author

**Aryaman Mahajan**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/aryaman-mahajan-23138a1b0/)
[![GitHub](https://img.shields.io/badge/GitHub-100000?style=for-the-badge&logo=github&logoColor=white)](https://github.com/AryamanMMahajan)
