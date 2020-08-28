---
title: 'Spot Mini Mini'
subtitle: 'Quadruped Locomotion, Bezier Gait, Reinforcement Learning, Mechanical, Electronics, Microcontroller'
date: 2020-06-04 10:05:55 +0300
description: Developed Pybullet Spot Environment and deployed 12-point Bezier-curve gait as baseline for RL task. Validated on real robot designed and built for under $600.
featured_image: '/images/Projects/spot-mini-mini/spot_hello.gif'
---

## Project Overview
The goal of this project was to create a **quadruped platform for reinforcement learning tasks under $600**. First, I built a **Pybullet environment** using the open-source Spot Micro CAD models (and later my own). Then, I deployed a 12-point Bezier gait scheme and used it as a baseline for Reinforcement Learning tasks on various terrain environments. After building this original version, I collaborated with a friend to mechanically redesign Spot for higher fidelity, and more optimal weight distribution. Check out the package on [Github](https://github.com/moribots/spot_mini_mini)!

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_hello.gif" style="width: 80%">
</div>

## Sim2Real: Gait Modulation with Bezier Curves
My experience with locomotive RL is that it is mostly limited to demonstrative tasks (walk forward, stand up, etc) with little real-world use. Using this platform, I propose a **novel reinforcement learning method** that seeks to deliver a robust and universally controllable gait. It builds on an existing gait scheme using **12-point Bezier curves** which allow for any **combination** of forward, lateral, and yaw **commands** at user-defined step heights, lengths, and speeds. The method wraps a learning agent around this scheme to **modulate gait parameters** such as step and body height, and to add significant **residuals** to the resultant foot coordinates. The only sensor used here is an IMU. After simulating training for 1 hour on a CPU (\~500 epochs), I was able to achieve the following real-world results, where the **agent** is shown on the **right**.

<div class="gallery" data-columns="2">
	<img src="/images/Projects/spot-mini-mini/V2_3.gif" style="width: 100%">
	<img src="/images/Projects/spot-mini-mini/T2_1.gif" style="width: 100%">
</div>

The best part is that even though the agent was only **trained** to walk **forward**, it responds to previously **unseen commands** such as **yaw** and **lateral** motion! This means that you can finally use RL on a real robot! Keep in mind that this is all done on a $600 platform!

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_lateral_agent.gif" style="width: 70%">
</div>

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_new_universal.gif" style="width: 70%">
</div>

## Inverse Kinematics
After deriving the [Inverse Kinematics](https://www.researchgate.net/publication/320307716_Inverse_Kinematic_Analysis_Of_A_Quadruped_Robot) for each leg, the next step was to describe the IK for the body itself. The approach used here considers a **world frame** $w$, which is the robot centroid's base position, and a **body frame** $b$, describing the robot's pose relative to the world frame. In addition, we have $T_{wh}$, which is a transform from the world frame to the robot's hip: this describes the base transform between the robot centroid and the hip. Finally, we have our inputs: $T_{wb}$, which describes the desired transform from **world** to **body** (RPY and Translation), and $T_{bf}$, the desired **foot** position relative to the transformed **body** - this is useful for gait generation. The output of our process is $T_{hf}$, the transform between each **hip** and its respective **foot** required to achieve this motion. The gallery below shows our inputs and outputs, as well as the actual process for getting $T_{hf}$. Note that this diagram is facing the robot, so the example shown is for **body roll**.

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/io.jpg" style="width: 70%">
	<img src="/images/Projects/spot-mini-mini/thf.jpg" style="width: 70%">
</div>

Here's a gif of the body IK in action:

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_rpy.gif" style="width: 60%">
</div>


## Bezier Gait
The [Bezier Gait](https://dspace.mit.edu/handle/1721.1/98270) deployed in this project uses a closed-loop trajectory generator, which resets whenever the reference foot (front left) hits the ground after a swing. The Bezier curve for the Swing period is made up of 12 points, where some of them overlap to induce zero vertical velocities or changes in foot direction. The Table and image below summarize the gait:


<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/bezier.jpg" style="width: 60%">
	<img src="/images/Projects/spot-mini-mini/bezier_points.png" style="width: 120%">
</div>

For the Stance portion of the gait, we simply deploy a sinusoidal curve whose z-amplitude is the desired penetration depth and whose x-amplitude is the half the Stride Length. Note, for y-coordinate foot motion, we apply the same Bezier and Sinusoidal curves as for x-coordinate motions.

Here's what this looks like on Spot:
<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_bezier.gif" style="width: 50%">
</div>

I implemented Yaw control based on [this paper](http://www.inase.org/library/2014/santorini/bypaper/ROBCIRC/ROBCIRC-54.pdf) which treats the quadruped as a four-wheel steering car. The intuition here is that to turn clockwise, both front feet should move towards the rear-left, and both back feet should move towards the rear-right of the robot during the stance phase. To adequately trace a circular path while doing this, the directional vector of each foot is modulated at each iteration by $\theta_{mod}$ to remain tangent to said circle, as shown in the second image below:

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/yaw_overview.jpg" style="width: 25%">
	<img src="/images/Projects/spot-mini-mini/yaw_mod.jpg" style="width: 50%">
</div>

Here's what this looks like on Spot:
<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_yaw.gif" style="width: 50%">
</div>

## Gym Environment and Terrain

The environment provided here is largely derived from Pybullet's **minitaur** example. In fact, it is nearly identical aside from accounting for the differences in the robots themselves. Another difference is the terrain used in the environment, which is an optional programmatically generated heightfield triggered at the command-line. You should experiment with the meshscale argument as well, as this will change the characteristics of your terrain. This environment is great for locomotive reinforcement learning tasks! Notice that if we increase the mesh size, and hence the terrain's roughness, the robot loses the ability to traverse it:

<div class="gallery" data-columns="2">
	<img src="/images/Projects/spot-mini-mini/spot_new_demo.gif" style="width: 100%">
	<img src="/images/Projects/spot-mini-mini/spot_rough_falls.gif" style="width: 100%">
</div>

### Reinforcement Learning Task

To allow for stable terrain traversal, I trained an [Augmented Random Search](https://arxiv.org/pdf/1803.07055.pdf) agent with a 12-dimensional observation space [**IMU Inputs** (8), **Leg Phases** (4)] and a 14-dimensional action space [**Clearance Height** (1), **Body Height** (1), and **Foot XYZ Residual** modulations (12)] processed through an exponential filter with **alpha = 0.7**, the agent was able to traverse the light terrain in as little as 150 epochs, and on terrain of twice the height (shown above) in 500 epochs.

<div class="gallery" data-columns="1">
	<img src="/images/Projects/spot-mini-mini/spot_rough_ARS.gif" style="width: 70%">
</div>

### Real World Validation

Here are some additional takes where the agent is on the right!

<div class="gallery" data-columns="2">
	<img src="/images/Projects/spot-mini-mini/spot_vanilla_fall.gif" style="width: 100%">
	<img src="/images/Projects/spot-mini-mini/spot_agent_walk.gif" style="width: 100%">
</div>

### Mechanical Redesign

Together with [Adham Elarabawy](https://github.com/adham-elarabawy/OpenQuadruped), I have a completed a total mechanical redesign of SpotMicro, the robot that inspired this project. We call it [Open Quadruped](https://cad.onshape.com/documents/9d0f96878c54300abf1157ac/w/c9cdf8daa98d8a0d7d50c8d3/e/fa0d7caf0ed2ef46834ecc24)!

<div class="gallery" data-columns="2">
	<img src="/images/Projects/spot-mini-mini/openquad_orange.png" style="width: 100%">
	<img src="/images/Projects/spot-mini-mini/openquad_black.png" style="width: 100%">
</div>

##### Main improvements:
* Shortened the body by 40mm while making more room for our electronics with adapter plates.
* Moved all the servos to the hip to save 60g on the lower legs, which are now belt-drive actuated with tunable belt tightness.
* Added support bridge on hip joint for added longevity.
* Added flush slots for hall effect sensors on the feet.

I also went created a new URDF with proper inertial values on each link, making the simulation much more reliable.

### Power Distribution Board

<figure style= "text-align: center; float: right; width: 30%; margin-right: 0%; margin-left: 10%; font-style: italic">
    <img src="/images/Projects/spot-mini-mini/pdb.png" style="width: 100%;" class="img-fluid rounded">
  </figure>

We also designed this Power Distribution Board with a **1.5mm Track Width** to support up to **6A** at a **10C** temperature increase (conservative estimate). There are copper grounding planes on both sides of the board to help with heat dissipation, and parallel tracks for the power lines are provided for the same reason. The PDB also includes shunt electrolytic capactiors for each servo motor to smooth out the power input. The board interfaces with a sensor array (optionally used for foot sensors) and contains two  I2C terminals and a regulated 5V power rail. At the center of the board is a **Teensy 4.0** which communicates with a **Raspberry Pi** over ROSSerial to control the 12 servo motors and read analogue sensors.

