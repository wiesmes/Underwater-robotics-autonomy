# Autonomous Underwater Vehicle — Perception & Navigation Stack

ROS2-based autonomy stack for an autonomous underwater vehicle, running on an NVIDIA Jetson Nano under Ubuntu Linux.

**Competition:** RoboSub — international student autonomous underwater vehicle competition
**Role:** Software Lead — perception and navigation stack
**Team:** 9 engineers
**Timeline:** Fall 2025 – Present
**Organization:** Lehigh Underwater Robotics

> **About this repository:** This is a design and architecture writeup. The implementation lives in the team's repository; this documents the system design and the work I own as software lead.

---

## Overview

Lehigh Underwater Robotics builds an autonomous underwater vehicle for **RoboSub**, an international competition in which student teams run a submerged obstacle course with no human control once the vehicle is in the water — no tether, no operator, no intervention. Everything the vehicle does, it decides onboard.

I own the perception and navigation stack: everything between the sensors and the heading command.

---

## The problem

Underwater autonomy loses most of the assumptions a ground robot gets for free. There is no GPS, so global position is unavailable. Visibility is limited and inconsistent, and the vehicle drifts. Once the vehicle is in the water there is no operator to correct a bad decision — the run either works or it doesn't. The system has to determine where the next gate is and hold a heading toward it from onboard sensing alone.

Testing is also expensive. Pool access comes in three-hour sessions that are hard to schedule, against a fixed competition date, so any bug that could have been caught on land wastes irreplaceable water time.

---

## Approach

### Map-free reactive navigation

Rather than building a global map and localizing within it, the stack uses a **gap-follow** controller. It identifies the traversable opening ahead from the depth field and steers toward it, correcting heading continuously as the vehicle advances.

**Why this instead of SLAM:**

- The mission is a sequence of gates, not an environment to explore. A global map would solve a problem this course doesn't pose.
- Underwater scenes are feature-poor and visually inconsistent — conditions that make visual SLAM unreliable.
- The Jetson Nano has a fixed compute budget. Spending it on perception and a fast control loop beats spending it on map maintenance the mission never uses.

**The trade-off:** there is no persistent world model. The vehicle reacts to what is in front of it and cannot reason about the course globally or recover a gate it has already passed. For this mission profile that is an acceptable cost; for a course requiring revisits or global planning it would not be.

### Perception

Two sensing paths feed the navigation controller:

- **Intel RealSense RGB-D** — depth field used to identify the traversable gap ahead.
- **AprilTag fiducial detection** — gate identification and relative pose from the color stream.

Together these give the controller both *where the opening is* and *which gate it belongs to*, without requiring any prior map.

---

## Architecture

The stack is built as modular ROS2 nodes communicating over publish/subscribe topics, written in C++ and Python. Decoupling perception, navigation, and control this way lets each run concurrently under real-time constraints and lets individual components be swapped or tested in isolation — which matters on a nine-person team where subsystems are developed in parallel.

**Platform:** NVIDIA Jetson Nano · Ubuntu Linux · ROS2

---

## Simulation-first validation

Pool sessions run three hours and are scarce, so the full autonomy stack is validated in simulation before anything reaches the water.

The effect is that water time is spent testing pre-validated changes rather than debugging basics — every session exercises code that has already survived a full run in simulation. On a team this size, it also means integration problems surface on land, where they are cheap to fix.

---

## What I own

- Architecture of the perception and navigation stack
- Gap-follow navigation controller
- Integration of RGB-D perception with AprilTag detection
- Simulation-first validation workflow for the team

---

## Author

**Wiesmes Antwi** — B.S. Electrical Engineering, Lehigh University
[LinkedIn](https://linkedin.com/in/wiesmes) · wwa228@lehigh.edu
