# Robot Programming Course Overview
**Tags:** #RobotProgramming #Robotics a #ProjectOverview #CourseStructure 
![[rp_00_intro.pdf]]
---

## 1. Core Paradigm: Sense-Plan-Act
The fundamental cycle of robot operation involves three stages:
1.  **Sense:** The robot perceives the environment through sensors.
2.  **Think (Plan):** The robot processes sensor data to decide on a reaction.
3.  **Act:** The robot executes the action via actuators to alter the environment.


---

## 2. Course Topics Overview

The course covers the full stack required to build robotic software, from the language level to the middleware and application level.

### C++ Programming
[[C++|C++]] is the primary language used because it compiles to machine code and supports multiple paradigms:
* **Imperative Programming**
* **Object-Oriented Programming (OOP)**
* **Generic Programming** (Templates)
* **Meta Programming**

### Build Systems
Tools for compiling and managing C++ projects:
* [[Compiler and build system#Makefiles|Make]]
* [[Compiler and build system#4. CMake (Cross-Platform Make)|CMake]]

### ROS (Robot Operating System)
[[ROS introduction#1. What is ROS?|ROS]] serves as the middleware for robotic systems.
* Provides infrastructure to design applications with multiple processing [[ROS introduction#Nodes|nodes]].
* Offers core functionalities and utilities for robot system design.
* Integrates [[Transform and sensors#3. The TF Library (Transforms)|transforms]] and driver handling.

---

## 3. Robotics Fundamentals

### Hardware Components
* **Sensors:** Devices that measure physical quantities in the environment and report them in digital form to the software (e.g., [[Transform and sensors#1. Accessing Devices in ROS|LIDAR]], Cameras).
* **Actuators:** Devices that alter the environment by applying perturbations based on program input (e.g., Motors).


### Navigation Stack
[[Navigation#1. Navigation Concepts|Navigation]] is the ability to move from point A to point B within a map. It requires a pipeline of specific tasks:
1.  **[[Navigation#3. Building a Map (SLAM)|Mapping]]:** Constructing a representation of the environment.
2.  **[[Navigation#4. Localizing in a Map (AMCL)|Localization]]:** Determining the robot's position within that map.
3.  **[[Navigation#5. The Navigation Stack|Path Planning]]:** Calculating a feasible path to the destination.
4.  **Execution:** Following the path via actuators.


---

## 4. Course Logistics

### Resources & Requirements
* **Repository:** `https://gitlab.com/grisetti/robotprogramming_2025_26`
* **OS Requirement:** Native installation of **Ubuntu 24.04**.
* **ROS Version:** ROS2 Rolling.

### Structure
* **Thursdays:** "Lecture" sessions introducing new concepts and exercises.
* **Wednesdays:** "Exercise" sessions presenting solutions (requires a Linux computer).

### Evaluation
The course uses a binary **Pass/No Pass** system based on:
1.  **Questionnaire:** To be done in class.
2.  **Coding Requirement:** Either a **Final Project** OR **2 out of 3 Homeworks**.

**Important Notes on Evaluation:**
* Work is tracked via **Git history** to verify development progress.
* Instructors will check correctness and ask questions about specific parts of the code.
* Plagiarism or improper use of AI tools results in invalidation of the work.

### Contacts
* Email subject line must start with: `[RP2025]`
* **Office Hours:** Thursdays 12:30 - 13:00 (in the lesson room).