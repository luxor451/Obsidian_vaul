**Tags:** #Robotics #DistanceMap #PathPlanning #Localization #Algorithms
![[rp_07_dmap.pdf]]
![[rp_08b_dmap_localizaton.pdf]]
![[rp_09_dmap_planning.pdf]]

---

## 1. The Neighbor Search Problem

Many problems in robotics (localization, mapping, planning) boil down to finding the closest point in a dataset to a query point.

**Problem Statement:**
Given a set of points $\mathcal{P} = \{p_1, \dots, p_N\}$ and a query point $p_q$:
1.  **Nearest Neighbor:** Find $p_i \in \mathcal{P}$ that minimizes distance $d(p_q, p_i)$.
2.  **Radius Search:** Find all $p_i \in \mathcal{P}$ such that $d(p_q, p_i) < \epsilon$.

### Distance Metrics
The choice of "closest" depends on the metric:
* **Squared Euclidean:** $||p_i - p_j||^2 = (p_i - p_j)^T (p_i - p_j)$
* **Omega Norm (Mahalanobis):** $||p_i - p_j||_\Omega^2 = (p_i - p_j)^T \Omega (p_i - p_j)$ (Accounts for uncertainty/covariance).
* **Hamming Distance:** For binary descriptors (number of different bits).

---

## 2. Distance Maps (DMaps)

A **Distance Map** is a grid where every cell stores the distance to the nearest obstacle (or target). It essentially pre-computes the answer to the "Nearest Neighbor" problem for every possible location in the grid.



### Definition
For a grid cell $x$, the value $D(x)$ is:
$$D(x) = \min_{o \in \text{Obstacles}} ||x - o||$$

### Computation Algorithm (Wavefront/Dijkstra)
Computing the DMap naively (checking every obstacle for every cell) is slow. An efficient approach uses a wavefront expansion (similar to Breadth-First Search or Dijkstra).

**Algorithm:**
1.  **Initialization:**
    * Set $D(x) = 0$ for all obstacle cells.
    * Set $D(x) = \infty$ for all free cells.
    * Add all obstacle cells to a priority queue (or open list) $Q$.
2.  **Expansion:**
    * While $Q$ is not empty:
        * Extract cell $x$ with the smallest distance.
        * For each neighbor $n$ of $x$:
            * Calculate potential new distance: $d_{new} = D(x) + \text{step\_cost}(x, n)$.
            * If $d_{new} < D(n)$:
                * $D(n) = d_{new}$
                * Add $n$ to $Q$.



```cpp
// Pseudo-code for Distance Map Expansion
void computeDMap(Grid& grid) {
    Queue q;
    
    // 1. Initialize
    for (auto& cell : grid.cells) {
        if (cell.isObstacle()) {
            cell.distance = 0;
            q.push(cell);
        } else {
            cell.distance = INFINITY;
        }
    }

    // 2. Propagate
    while (!q.empty()) {
        Cell current = q.pop();
        
        for (Cell neighbor : getNeighbors(current)) {
            float new_dist = current.distance + dist(current, neighbor);
            if (new_dist < neighbor.distance) {
                neighbor.distance = new_dist;
                q.push(neighbor);
            }
        }
    }
}
```

---

## 3. Applications

### A. Obstacle Avoidance (Force Fields)
The Distance Map defines a scalar field. We can compute the **gradient** of this field to determine the direction that moves away from obstacles most quickly.

* **Repulsive Force:** $\vec{F}_{rep} = -\nabla D(x)$ (or proportional to the gradient).
* **Control Law:** $\vec{v}_{cmd} = \vec{v}_{goal} + k \cdot \vec{F}_{rep}$.
* The robot "surfs" down the gradient away from walls.



### B. Path Planning
We can use the grid as a graph (8-connected) to find a path from Start to Goal.
* **Cost Function:** The cost of traversing a cell is not just distance, but a function of the **clearance** (distance to obstacles).
    * Cost $\propto 1 / D(x)$ (High cost near walls, low cost in open space).
* **Algorithm:** Use A* or Dijkstra on this weighted graph.
* **Result:** Paths that naturally stay in the middle of corridors (Voronoi-like behavior) rather than hugging corners.

### C. Localization (Scan Matching)
When aligning a laser scan to a map, we want to find the robot pose $X$ that minimizes the error between scan points and map obstacles.

**Optimization Problem:**
$$X^* = \text{argmin}_X \sum_{j} || d(X \cdot z_j) ||^2$$
* $X$: Robot Pose.
* $z_j$: Laser point in robot frame.
* $X \cdot z_j$: Laser point transformed to map frame.
* $d(\cdot)$: Distance to nearest obstacle.

**Role of DMap:**
Instead of searching for the nearest obstacle for every point $z_j$ at every iteration (which is expensive), we look up the value in the Distance Map. The DMap provides the distance (and its gradient) in $O(1)$ time. This is often called **Chamfer Matching** or **Likelihood Fields**.

---

# Distance Map Planning


## 1. Concept: Distance Map for Planning

A **Distance Map (DMap)** is an efficient data structure for path planning. Instead of checking for collisions with polygons or complex shapes at every step, we pre-calculate the distance to the nearest obstacle for every cell in the grid.

**Advantages:**
* **Efficiency:** Collision checking becomes an $O(1)$ lookup.
* **Clearance:** We can easily determine how "safe" a cell is based on its distance value.
* **Gradient:** The gradient of the distance map guides the robot away from obstacles (repulsive force).



---

## 2. Cost Function

To use standard graph search algorithms (like Dijkstra or A*), we need to define a **Cost Function** for traversing a cell. This cost depends on the distance from obstacles.

**Cost Formula:**
We typically define a critical radius $r_{min}$ (robot radius) and an influence range $r_{max}$.
* If $d < r_{min}$: Cost is Infinite (Collision).
* If $r_{min} < d < r_{max}$: Cost decreases linearly (or exponentially) as distance increases.
* If $d > r_{max}$: Cost is minimal (Free space).

$$Cost(x) = \begin{cases} \infty & \text{if } d(x) < r_{min} \\ C_{max} \cdot \frac{r_{max} - d(x)}{r_{max} - r_{min}} & \text{if } r_{min} \le d(x) < r_{max} \\ 0 & \text{if } d(x) \ge r_{max} \end{cases}$$



---

## 3. Path Planning Algorithm

We model the environment as an **8-connected graph** (grid).
We can use **A*** (A-Star) or **Dijkstra** to find the optimal path.

**Traversal Cost:**
The cost to move from cell $u$ to neighbor $v$ is:
$$Cost(u, v) = \text{Dist}(u, v) + \alpha \cdot \text{MapCost}(v)$$
* $\text{Dist}(u, v)$: Euclidean distance (1 for cardinal, $\sqrt{2}$ for diagonal).
* $\text{MapCost}(v)$: The obstacle proximity cost calculated above.

**Policy:**
If the environment is static, running Dijkstra from the Goal to all other cells generates a **Navigation Policy** (or Vector Field).
* For any cell, the "parent" pointer tells you the optimal next step to reach the goal.
* This allows re-planning the next move in $O(1)$ time if the robot deviates slightly.



---

## 4. ROS Implementation Strategy

A Distance Map Planner node integrates with the ROS ecosystem.

### Node Structure
* **State:** Current Robot Pose estimate ($X$).
* **Subscribers:**
    * `/map`: Occupancy grid to compute/update the Distance Map.
    * `/tf`: To track the robot's current position.
    * `/goal_pose`: The destination point.

### Logic Flow
1.  **Map Update:** When a new map arrives, recompute the Distance Map (Wavefront expansion).
2.  **Goal Received:**
    * Check if Goal is valid (free space).
    * Compute the path using A* on the Distance Map.
3.  **Path Execution (Control Loop):**
    * Determine the next waypoint on the path.
    * Compute velocity commands ($v, \omega$) to drive towards the waypoint while respecting the map gradient (staying centered).
    * Publish `/cmd_vel`.

### Code Snippet (Conceptual A*)
```cpp
struct Node {
    int x, y;
    float g_score; // Cost from start
    float f_score; // g_score + heuristic
    Node* parent;
};

std::vector<Point> computePath(Point start, Point goal, const DMap& dmap) {
    PriorityQueue open_set;
    open_set.push(start);
    
    while (!open_set.empty()) {
        Node current = open_set.pop();
        
        if (current == goal) return reconstructPath(current);
        
        for (Node neighbor : getNeighbors(current)) {
            float tentative_g = current.g_score + dist(current, neighbor) + dmap.getCost(neighbor);
            
            if (tentative_g < neighbor.g_score) {
                neighbor.parent = current;
                neighbor.g_score = tentative_g;
                neighbor.f_score = tentative_g + heuristic(neighbor, goal);
                open_set.push(neighbor);
            }
        }
    }
    return {}; // No path found
}
```