# Multi-Robot SLAM & Autonomous Navigation

**Synchronization and coordination of two TurtleBot3 mobile robots using ROS, SLAM, and autonomous navigation in a shared simulated environment.**

[![ROS](https://img.shields.io/badge/ROS-Noetic-brightgreen.svg)](https://wiki.ros.org/noetic)
[![Gazebo](https://img.shields.io/badge/Gazebo-11-orange.svg)](http://gazebosim.org/)
[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)](https://www.python.org/)
[![Ubuntu](https://img.shields.io/badge/Ubuntu-20.04-E95420.svg)](https://ubuntu.com/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 🎯 Project Goal

Develop and implement a system for coordinated, synchronized exploration using two autonomous TurtleBot3 robots in a simulated Gazebo environment.

**Key Question:** Can two robots collaboratively map an unknown environment faster and more completely than a single robot?

---

## 🏆 Key Results

**Single Robot Navigation (Successfully Implemented):**
- ✅ Autonomous navigation via AMCL + move_base with DWA local planner
- ✅ Accurate real-time obstacle avoidance and goal-reaching
- ✅ Full 2D occupancy grid map generated via Gmapping

**Dual Robot SLAM (Successfully Implemented):**
- ✅ Both robots spawned in the same Gazebo world without TF conflicts
- ✅ Independent SLAM pipelines isolated using ROS namespaces
- ✅ Individual maps generated and merged via `multirobot_map_merge`

**Critical Discovery:**
- TF frame conflicts (`base_footprint`, `odom`) caused `TF_REPEATED_DATA` errors when running two robots without namespaces
- Resolved through strict namespace isolation — each robot treated as a fully independent ROS entity
- Communication breakdown between `slam_gmapping` and `explore_lite` prevented fully autonomous frontier exploration

---

## 🖼️ Demonstrations

### SLAM + Navigation Visualization
<table>
  <tr>
    <td><b>SLAM & Costmap (RViz)</b></td>
    <td><b>Gazebo Simulation World</b></td>
  </tr>
  <tr>
    <td><img src="images/SLAM plus Nav.png" width="400"/></td>
    <td><img src="images/Nav_pic_2.png" width="400"/></td>
  </tr>
</table>

### Autonomous Path Planning & Navigation
<table>
  <tr>
    <td><b>Path Planning (move_base)</b></td>
    <td><b>Robot POV in Gazebo</b></td>
  </tr>
  <tr>
    <td><img src="images/Path planning.png" width="400"/></td>
    <td><img src="images/Nav_pic_1.png" width="400"/></td>
  </tr>
</table>

### Combined System View
<table>
  <tr>
    <td><b>RViz + Gazebo Combined</b></td>
    <td><b>Costmap & DWA Planner Detail</b></td>
  </tr>
  <tr>
    <td><img src="images/Combined view.png" width="400"/></td>
    <td><img src="images/Project overview shot.png" width="400"/></td>
  </tr>
</table>

---

## 📊 Results Summary

| Feature | Status | Notes |
|---|---|---|
| Single robot SLAM | ✅ Complete | Gmapping + RViz visualization |
| Single robot navigation | ✅ Complete | AMCL + move_base + DWA planner |
| Dual robot spawning | ✅ Complete | ROS namespace isolation |
| Dual robot SLAM | ✅ Complete | Independent gmapping per robot |
| Map merging | ✅ Partial | `multirobot_map_merge` implemented |
| Autonomous frontier exploration | ⚠️ Partial | TF comm. issue with `explore_lite` |

---

## 🔬 System Architecture

**Environment:** Custom Gazebo house world with furniture, walls, and obstacles  
**Robots:** TurtleBot3 Burger (differential drive, LiDAR equipped)  
**SLAM:** Gmapping (particle-filter based 2D occupancy grid)  
**Localization:** AMCL (Adaptive Monte Carlo Localization)  
**Navigation:** move_base with DWA local planner and costmaps

```
                ┌──────────────────────────────────────┐
                │          Gazebo Simulation            │
                │   ┌──────────┐    ┌──────────┐       │
                │   │ Robot 1  │    │ Robot 2  │       │
                │   │ (tb3_1)  │    │ (tb3_2)  │       │
                └───┴────┬─────┴────┴────┬─────┴───────┘
                         │                │
           ┌─────────────┘                └──────────────┐
           ▼                                              ▼
┌──────────────────────┐                    ┌──────────────────────┐
│  Namespace: /tb3_1   │                    │  Namespace: /tb3_2   │
│  - gmapping          │                    │  - gmapping          │
│  - amcl              │                    │  - amcl              │
│  - move_base         │                    │  - move_base         │
└──────────┬───────────┘                    └──────────┬───────────┘
           │  /tb3_1/map                               │  /tb3_2/map
           └───────────────────┬───────────────────────┘
                               ▼
               ┌───────────────────────────┐
               │   multirobot_map_merge    │
               │   publishes → /map        │
               └─────────────┬─────────────┘
                             ▼
               ┌───────────────────────────┐
               │           RViz            │
               └───────────────────────────┘
```

**Key Navigation Parameters:**
```
Costmap:      resolution=0.05, width=60, height=60
SLAM:         gmapping (particle filter, 2D occupancy grid)
Localization: AMCL (adaptive Monte Carlo, laser + odometry)
Planner:      DWA local planner, A* global planner
```

---

## 💡 Key Insights

**1. Namespace Isolation is Essential for Multi-Robot ROS**
- Without namespaces, identical TF frame names (`base_footprint`, `odom`) cause `TF_REPEATED_DATA` errors
- Each robot must be its own isolated entity with prefixed topics and frames

**2. Node Communication Must Be Explicitly Verified**
- `slam_gmapping` and `explore_lite` failed to communicate despite both running correctly
- The `map → base_footprint` transform was not being forwarded to the exploration node
- In multi-robot ROS systems, every inter-node dependency must be validated individually

**3. Multi-Robot Navigation Requires Per-Robot Stack Instances**
- Separate instances of `amcl`, `move_base`, and `slam` are needed per robot
- `.rviz` config must be updated for each robot's sensor, map, and TF parameters

**4. Map Merging Requires Known Initial Poses**
- `multirobot_map_merge` needs either known initial poses or sufficient map overlap for alignment
- Accurate initial pose setting (`init_pose.py`) is critical for successful merging

---

## 🛠️ Quick Start

### Prerequisites

- Ubuntu 20.04
- ROS Noetic (full desktop install)
- Gazebo 11

```bash
sudo apt install ros-noetic-turtlebot3 ros-noetic-turtlebot3-simulations
sudo apt install ros-noetic-slam-gmapping ros-noetic-navigation
sudo apt install ros-noetic-multirobot-map-merge ros-noetic-explore-lite
```

### Installation

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/multi-robot-slam.git
cd multi-robot-slam

# Build the workspace
catkin_make
source devel/setup.bash
```

### Running the Simulation

```bash
# 1. Set robot model and launch Gazebo world
export TURTLEBOT3_MODEL=burger
roslaunch multi_robot_nav multi_robot_gazebo.launch

# 2. Start SLAM for both robots
roslaunch multi_robot_nav multi_robot_slam.launch

# 3. Launch autonomous navigation
roslaunch multi_robot_nav multi_robot_navigation.launch

# 4. Set initial poses and open RViz
rosrun auto_nav init_pose.py
rviz -d config/turtlebot3_auto_nav.rviz
```

---

## 📂 Key Files

- `launch/multi_robot_gazebo.launch` — Spawns both TurtleBot3 robots with namespaces
- `launch/multi_robot_slam.launch` — Starts gmapping for each robot
- `launch/multi_robot_navigation.launch` — Starts AMCL + move_base per robot
- `scripts/init_pose.py` — Sets initial pose for AMCL localization
- `config/turtlebot3_auto_nav.rviz` — RViz config for multi-robot visualization
- `config/map_merge.launch` — multirobot_map_merge setup

---

## 🚀 Future Work

- **Autonomous Exploration:** Fix TF communication between `slam_gmapping` and `explore_lite` for full frontier-based exploration
- **More Robots:** Scale namespace architecture to 3+ robots with dynamic task assignment
- **Real Hardware:** Deploy on physical TurtleBot3 hardware using the same ROS stack
- **Coordination Strategy:** Implement bidding-based task allocation for efficient frontier assignment

---

## 👥 Team

**Ahilesh Vadivel** · **Siyu Liu** · **Jorge Ortega** · **Sachidanand Halhalli**

*EECE 5554 – Robotic Sensing & Navigation, Northeastern University, Fall 2024*

---

## 📚 References

- Quigley et al. (2009). *ROS: An open-source Robot Operating System*. ICRA Workshop.
- Koenig & Howard (2004). *Design and Use Paradigms for Gazebo*. IEEE/RSJ IROS.
- Thrun, Burgard & Fox (2005). *Probabilistic Robotics*. MIT Press.
- [TurtleBot3 Documentation](https://emanual.robotis.com/docs/en/platform/turtlebot3/)

---

## 📊 Technical Skills Demonstrated

- Multi-Robot Systems (ROS namespaces, TF frame management)
- Simultaneous Localization and Mapping (Gmapping, AMCL)
- Autonomous Navigation (move_base, DWA planner, costmaps)
- Physics Simulation (Gazebo, RViz)
- ROS Launch File Architecture & Node Communication
- Python / ROS scripting

---

*This project demonstrates end-to-end multi-robot ROS development: from simulation setup through SLAM and navigation to map merging, including debugging real-world TF frame conflicts in a multi-agent system.*
