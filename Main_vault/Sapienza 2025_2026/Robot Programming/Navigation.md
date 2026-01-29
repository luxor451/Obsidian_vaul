**Tags:** #Robotics #Navigation #ROS #Mapping #Localization #PathPlanning #SLAM #GMapping #AMCL
![[rp_08a_navigation.pdf]]

---

## 1. Navigation Concepts

Navigation is the ability of a robot to move from its current position to a goal position within an environment.

### Core Components
* **Map:** A representation of the environment.
* **Pose:** The robot's position ($x, y$) and orientation ($\theta$) within the map.
* **Path:** A sequence of poses or actions to reach the goal.

### The Problem
To navigate autonomously, a robot must answer three questions:
1.  **"Where am I?"** (Localization)
2.  **"Where am I going?"** (Goal Selection)
3.  **"How do I get there?"** (Path Planning)



---

## 2. Taxonomy of Navigation

Navigation systems can be classified based on how much information is known a priori.

1.  **Map-Based Navigation:**
    * The robot has a complete map of the environment.
    * **Localization:** Determine pose in the known map.
    * **Planning:** Compute path to goal avoiding static obstacles.

2.  **Map-Building (SLAM):**
    * The robot starts in an unknown environment.
    * **Mapping:** Build a map from sensor data while exploring.
    * **SLAM (Simultaneous Localization and Mapping):** The robot must localize itself relative to the map it is currently building.

3.  **Mapless Navigation (Reactive):**
    * No map is built or stored.
    * The robot reacts to sensor data in real-time (e.g., wall following, obstacle avoidance).

---

## 3. Building a Map (SLAM)

In ROS, the standard approach for 2D mapping is **GMapping**.

### GMapping
* Implements a **Particle Filter** based SLAM approach (Rao-Blackwellized Particle Filter).
* It estimates the robot's trajectory and the map simultaneously.

### How to Build a Map in ROS
1.  **Record Data:** Drive the robot around the environment and record sensor data (Laser scans `/scan`, Odometry `/odom`, Transforms `/tf`) into a **bag file**.
2.  **Run GMapping:** Play back the bag file while running the `gmapping` node.
3.  **Save Map:** Use `map_server` to save the generated map to disk.

### Map Representation
The output consists of two files:
* **Image (`.pgm`):** A grayscale image representing the occupancy grid (White = Free, Black = Obstacle, Gray = Unknown).
* **Metadata (`.yaml`):** Describes map properties like resolution (meters/pixel) and origin coordinates.



---

## 4. Localizing in a Map (AMCL)

Once a map exists, the robot needs to localize itself within it. The standard ROS package is **AMCL (Adaptive Monte Carlo Localization)**.

### How AMCL Works
* Uses a **Particle Filter** to track the posterior distribution of the robot's pose.
* **Particles:** Each particle represents a hypothesis of the robot's pose $(x, y, \theta)$.
* **Update Steps:**
    1.  **Motion Update:** When the robot moves (odometry), particles are shifted accordingly. Uncertainty increases (particles spread out).
    2.  **Sensor Update:** When a laser scan is received, particles that match the scan well (high likelihood) are kept; bad matches are discarded (resampling). Uncertainty decreases (particles converge).

### ROS Integration
* AMCL publishes a transform from the `map` frame to the `odom` frame.
* This "correction" transform fixes the drift inherent in the robot's odometry.
* The true robot pose in the map is the transform chain: `map` $\to$ `odom` $\to$ `base_link`.



---

## 5. The Navigation Stack

The ROS Navigation Stack combines these components to move the robot.

### Layers
1.  **Global Planner:** Computes a high-level path from Start to Goal on the static map (e.g., using A* or Dijkstra).
2.  **Local Planner:** Computes velocity commands (`cmd_vel`) to follow the global path while avoiding dynamic obstacles (e.g., people walking).
3.  **Costmaps:** Grid-based maps used by planners.
    * **Global Costmap:** Used by Global Planner (Static map + inflation).
    * **Local Costmap:** Used by Local Planner (Rolling window of recent sensor data).

### Setup in ROS
To set up navigation, you need to configure:
* `move_base`: The main node coordinating planners.
* `costmap_common_params`: Parameters common to both costmaps (robot footprint, obstacle layers).
* `global_costmap_params`: Configuration for the global map.
* `local_costmap_params`: Configuration for the local window.
* `base_local_planner_params`: Tuning for the local controller (max velocity, acceleration limits).
