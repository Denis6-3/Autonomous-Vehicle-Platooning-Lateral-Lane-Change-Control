# Autonomous-Vehicle-Platooning-Lateral-Lane-Change-Control

![MATLAB](https://img.shields.io/badge/MATLAB-R202x-orange.svg)
![Simulink](https://img.shields.io/badge/Simulink-Control_System-blue.svg)
![Status](https://img.shields.io/badge/Status-Educational_Project-green.svg)

> **Development of a Vehicle Platooning System with lateral planning capabilities. The follower vehicle autonomously adapts its lateral reference to mimic the leader's maneuvers while ensuring safety constraints, powered by Adaptive LQR. Built on SCANeR Studio and MATLAB/Simulink.**

---

### Visual Demo of the platooning concept

<table width="100%">
  <tr>
    <td colspan="2" align="center">
      <img src="https://github.com/user-attachments/assets/678d9b85-642a-4584-b3fd-7f445699550c" width="400" alt="Project Simulation GIF"/>
      <br><br>
      <b>Fig 1.</b> Simulation of cooperative lateral maneuver in vehicle platooning.
    </td>
  </tr>
</table>

<table width="100%">
  <tr>
    <td align="center" width="50%" valign="top">
      <img src="https://github.com/user-attachments/assets/c041f243-47db-4fca-8e26-9cc3147a0d0f" width="300" alt="Truck Platooning Concept"/>
      <br><br>
      <b>Fig 2.</b> Illustration of truck platooning.
      <br>
      <sub>Source: <a href="https://www.rapp.ch/de/stories/truck-platooning">rapp.ch</a></sub>
    </td>
    <td align="center" width="50%" valign="top">
      <img src="https://github.com/user-attachments/assets/8df1a407-4ac3-4c68-adc7-51ef187335ed" width="250" alt="NAHSC Platoon 1997"/>
      <br><br>
      <b>Fig 3.</b> NAHSC platoon demo (San Diego, 1997).
      <br>
      <sub>Source: <a href="https://www.tech-faq.com/vehicle-platooning.html">tech-faq.com</a></sub>
    </td>
  </tr>
</table>


---


### Disclaimer
**This project was developed and validated using the SCANeR Studio co-simulation middleware.**
Since the simulation environment is proprietary, the `.slx` files provided here (if still present) are for **algorithmic review and architectural demonstration**. They cannot be executed without the specific SCANeR Studio licenses and interfaces.

---

### Project Overview

This project implements a **collaborative Vehicle Platooning system** focusing on autonomous lateral planning and control. The goal here is to dynamically adapt the lateral reference to track a **Leader Vehicle** and execute cooperative overtaking maneuvers. The control strategy relies on the robust **Adaptive Linear Quadratic Regulator (LQR)** developed in my [previous work](https://github.com/Denis6-3/Adaptive-LQR-Control-for-Lane-Keeping-Systems-LKS-), applied to a dynamic bicycle model.

The system was validated on a **dynamic driving simulator** to assess performance and realism. The setup includes:
* **Immersive Display:** A triple-screen configuration for a wide field of view.
* **Motion Platform:** A dynamic seat actuated by cylinders to simulate longitudinal forces (acceleration/braking) and physical feedback.
* **Physical hardware setup** : real steering wheel, pedals, and gearbox.

<p align="center">
  <img src="https://github.com/user-attachments/assets/f966ef87-6ad1-4e92-9690-f7d771caafce" width="225" />
  &nbsp; &nbsp; &nbsp;
  <img src="https://github.com/user-attachments/assets/3b7ab8bc-53dc-404e-ab14-d1bb0189257a" width="500" />
</p>

---

### Technical Approach

#### 1. Road Geometry & Planning Strategy

In the SCANeR Studio simulation environment, the road geometry is standardized with the central median at `0`. From there, we map the exact coordinates of the two usable lanes:
* **Left Lane:** Ends at `-1.25` / `-4.25` (Center: **-2.5**)
* **Right Lane:** Ends at `-4.25` / `-7.25` (Center: **-6.0**)

<img width="400" alt="Lane Geometry" src="https://github.com/user-attachments/assets/c772152b-8e36-4990-94bf-cfd07641411d" />

**Why does this positioning matter?**

You might be wondering why I map the lanes so precisely instead of just recording the leader's trajectory. That is a great question, and the answer lies in stability.

My goal was to adapt the Lateral Control system I previously developed to handle platooning. However, simply mimicking the leader's exact path is risky. If the leader swerves slightly to avoid a pothole or a rabbit, we don't want the follower to blindly copy that erratic movement.

**The Solution: "Snap-to-Lane" Logic**

Instead of following the leader's exact coordinates, the system acts smarter:
1.  It detects the leader's position at every sampling step.
2.  It determines which lane the leader is currently in.
3.  It **forces the follower's lateral setpoint** to the center of that specific lane (either `-2.5` or `-6.0`).

This ensures the follower stays perfectly centered, even if the leader drives a bit off-center.

The figure below illustrates the actual Simulink implementation of this logic:

<p align="left">
  <img src="https://github.com/user-attachments/assets/aa2b658b-a006-4a42-8c57-f024836330d5" width="500" alt="Simulink Decision Logic"/>
  <br>
  <b>Fig 4.</b> Simulink implementation of the lane decision logic.
</p>

---
#### 2. Signal Smoothing & Passenger Comfort

**The Problem: Abrupt Dynamics**

During the initial testing phase, I noticed that feeding the raw decision output (a sudden step signal) directly to the controller was problematic. The command was too brutal: the vehicle tried to "jump" instantly from one lane center to the other, causing **significant oscillations** and jerky steering behavior.

**The Solution: Rate Limiter**

To solve this, I integrated a **Rate Limiter** block to filter the signal.

<p align="left">
  <img src="https://github.com/user-attachments/assets/d7902d1d-e21a-4194-8055-78e6a905e368" width="500" alt="Simulink Decision Logic"/>
  <br>
  <b>Fig 5.</b> Simulink implementation: Integration of the Rate Limiter block.
</p>

* **How it works:** It acts similarly to a 2nd-order discrete filter, transforming the sharp "step" command into a progressive "ramp."
* **The Result:** This simple addition ensures the lateral trajectory is smooth and realistic, guaranteeing **passenger comfort** and keeping the vehicle stable during maneuvers.


To validate this approach, I simulated a stress test where a **virtual leader** (calculated in code but invisible in the rendering) switches lanes every 4 seconds. The video below demonstrates the **stabilizing effect of the Rate Limiter**: the follower executes seamless transitions without any overshoot or oscillation.

<table align="left">
  <tr>
    <td width="400" align="center">
      <video src="https://github.com/user-attachments/assets/93439038-9174-4ec8-b49f-4450835c3d4b" controls muted autoplay></video>
      <b>Video 1.</b> "Overtaking" test scenario: Smooth lane changes simulating a passing maneuver.
    </td>
  </tr>
</table>

<br> <br>

<img width="400" alt="courbe smooth" src="https://github.com/user-attachments/assets/6329bb37-287a-472f-b358-15829c493fdc" />
<br>
<b>Fig 6.</b> Evolution of vehicle lateral position relative to the input.

<br clear="all" />












---

#### 3. Handling Edge Cases (Safety Check)
At this stage, you might think the job is done. But in reality, we have to deal with some annoying (and dangerous) scenarios.

For example:
* **The "Cut-in" Risk:** The leader (V1) moves to the left lane and barely squeezes in. However, a fast car is approaching from behind, making it impossible for the follower (V2) to join safely.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/c4ff94f6-89da-4005-884b-62f2531bec36" />

* **The "Blocked Merge" Risk:** The leader (V1) merges back to the right after overtaking, but another vehicle is right next to the follower, blocking the merge.

<img width="400" alt="image" src="https://github.com/user-attachments/assets/96ac0c1c-54a1-4795-b066-3cc56b9ea83b" />

&nbsp;

To handle these situations, I leveraged SCANeR Studio’s data (positions and speeds of surrounding traffic) to build a **Safety Check Algorithm**. The follower is only allowed to change lanes if the specific lane is free and safe, regardless of what the leader does.

*(See the logic diagram below)*




