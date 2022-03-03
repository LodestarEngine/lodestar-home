---
title: "Real-time Control Examples"
description: "Practical examples involving a robot arm and a thermal camera."
lead: "Practical examples involving a robot arm and a thermal camera."
date: 2022-03-01T00:00:00+00:00
lastmod: 2022-03-01T00:00:00+00:00
draft: false
weight: 50
images: ["panda.jpg"]
contributors: ["Hamza El-Kebir"]
---

![Franka Emika Panda robotic arm and dual Optris Xi 400 microscopic thermographer setup attached to the gripper.](panda.jpg "<a href=\"https://www.franka.de/robot-system/\">Franka Emika Panda</a> robotic arm and dual <a href=\"https://www.optris.com/optris-xi-400\">Optris Xi 400</a> microscopic thermographer setup attached to the gripper.")

In our paper, ["Lodestar: An Integrated Embedded Real-Time Control Engine"](https://arxiv.org/abs/2203.00649), we presented two applications of Lodestar: one involving real-time torque control of a 7-jointed robotic arm, and the other dealing with real-time autofocus of a thermal imaging camera. Both of these are challenging: the former requires fast execution time due to its strict real-time constraints, and the latter requires a custom computer vision algorithm to run efficiently between frames. In this post, we discuss both applications in detail, going through each line of code.

{{< alert icon="❓" context="info" text="For source code, see <a href=\"https://github.com/helkebir/Lodestar-Examples\">this GitHub repository</a>." />}}

## PID Controllers and Torque Control

### Designing a custom block

In order to tackle the challenge, the first order of the day was to let Lodestar interface with Franka Emika's `libfranka` drivers. These drivers were made with real-timability in mind: any control algorithm is implement using a callback function, the output of which gets processed by Franka Emika's proprietary control unit. Not only did we have to design a regular interface in Lodestar, we also had to make it run in parallel.

{{< alert icon="⚠️" context="info" text="This post is a work-in-progress." />}}

## Real-time Autofocus for Single-view Thermography