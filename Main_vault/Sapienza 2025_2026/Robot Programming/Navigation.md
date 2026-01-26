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

---

# Distance Map Localization (Scan Matching)


## 1. Problem Formulation

Localization can be framed as an optimization problem where we want to find the robot pose $X$ that aligns the current sensor readings with the map.

Using a **Distance Map (DMap)**, we can formulate this as minimizing the distance between the transformed laser points and the nearest obstacles in the map.

**Objective Function:**
$$X^* = \text{argmin}_{X} \sum_{j} || d(X \oplus z_j) ||^2$$

* $X$: The robot pose (what we want to estimate).
* $z_j$: A laser scan point (endpoint) in the robot's local frame.
* $X \oplus z_j$: The scan point transformed into the world frame.
* $d(\cdot)$: The distance value from the Distance Map at that world coordinate.

Essentially, we want to move the robot pose $X$ such that all laser points "land" on cells with zero distance (obstacles).



---

## 2. The Optimization Algorithm: Gauss-Newton

Since the distance function is non-linear, we use an iterative Least Squares solver like **Gauss-Newton**.

### The Concept
We want to minimize the squared error term:
$$X^* = \text{argmin}_{x} \sum_{i} ||e_i(X)||^2$$
where the error $e_i(X) = d(X \oplus z_i)$.

We approximate the error function around the current estimate using a first-order Taylor expansion and iteratively compute a correction $\Delta X$.

### The Jacobian
The core of Gauss-Newton is the Jacobian Matrix $J$, which tells us how the error changes as the pose changes.
By applying the Chain Rule:
$$\frac{\partial d(X \oplus z_j)}{\partial X} = \underbrace{\frac{\partial d(p)}{\partial p}}_{\text{Map Gradient}} \cdot \underbrace{\frac{\partial (X \oplus z_j)}{\partial X}}_{\text{Geometric Jacobian}}$$

1.  **Map Gradient ($\nabla D$):** How the distance changes as we move in $x, y$ on the map. This is pre-computed in the Distance Map.
2.  **Geometric Jacobian:** How the point's world coordinates change as the robot moves/rotates.

### The Algorithm Step
1.  **Initialize:** $H = 0$, $b = 0$.
2.  **Accumulate:** For each scan point $z_j$:
    * Compute world point $p = X \oplus z_j$.
    * Lookup distance error $e = D(p)$.
    * Lookup gradient $J_{map} = \nabla D(p)$.
    * Compute full Jacobian $J = J_{map} \cdot J_{geom}$.
    * Update Information Matrix: $H += J^T J$.
    * Update Error Vector: $b += J^T e$.
3.  **Solve:** $\Delta X = -H^{-1} b$.
4.  **Update:** $X \leftarrow X + \Delta X$.
5.  **Repeat:** Until convergence.

---

## 3. ROS Implementation Strategy

A robust localizer integrates Odometry (prediction) and Scan Matching (correction).

### Workflow
1.  **Initialization:** The algorithm starts with an initial guess (e.g., from a message on `/initial_pose`).
2.  **Odometry Update (Prediction):**
    * When an `/odom` message arrives, calculate the relative motion $U$ since the last step.
    * $U = Odom_{t-1}^{-1} \cdot Odom_t$.
    * Update estimate: $X \leftarrow X \oplus U$.
3.  **Scan Update (Correction):**
    * When a `/scan` message arrives, compute endpoints in the robot frame.
    * $x_i = r_i \cos(\theta_i), \quad y_i = r_i \sin(\theta_i)$.
    * Run Gauss-Newton registration to refine $X$.

### TF Publishing (The Transform Tree)
Standard ROS navigation requires a single connected TF tree.
* **Static:** `Map` frame.
* **Drifting:** `Odom` frame (continuous but inaccurate).
* **Robot:** `Base_Link` frame.

**The Rule:** The robot hardware/drivers publish `Odom -> Base_Link`. The Localizer **must** publish `Map -> Odom` to correct the drift. It should **not** publish `Map -> Base_Link` directly, or the tree will break.

**Calculation:**
We have the estimated pose $T_{localizer}$ (Map $\to$ Base) and the raw odometry $T_{odom}$ (Odom $\to$ Base).
We need to find the correction transform $T_{correction}$ (Map $\to$ Odom).

$$T_{localizer} = T_{correction} \cdot T_{odom}$$
$$\Rightarrow T_{correction} = T_{localizer} \cdot T_{odom}^{-1}$$