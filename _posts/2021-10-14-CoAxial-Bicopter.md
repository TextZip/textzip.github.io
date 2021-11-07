---
title: Co-Axial Bi-Copter
date: 2021-10-18 19:13:20 +/-0530
categories: [Projects, DIY]
tags: [atmega328p,game,imu,touch]     # TAG names should always be lowercase
# image_sliders:
#   - gamingConsole
image: /assets/img/CoaxialBicopter/nano.png
---
![Image1](/assets/img/CoaxialBicopter/nano.png){: .shadow}

A project to explore control strategies for thrust vector control of an
co-axial propeller with four control surfaces.

**Note: The project is in progress, further details/sections will be added as soon as the project is completed.**
# Hardware
- Arduino Nano 33 BLE Sense
- 2300kv BLDC 
- 8 inch props
- 2500 mAh Lipo 
- 45A ESC
- CT6B 
- MG90 Mini Servo 
# Design
## Initial Prototype
<!-- pic here -->
![Image1](/assets/img/CoaxialBicopter/iso_old.png){: .shadow}


The initial prototype was made using 1000kv BLDC motors along with 13inch props, as these were the cheapest materials that were available during the pandemic. But the length and width of the support frame was very large which increases the complexity of getting the frame 3D printed.  

## Final Prototype
<!-- pic here -->
![Image1](/assets/img/CoaxialBicopter/iso_final.png){: .shadow}

The final prototype had half the width of the initial prototype and each of the support frame elements were designed so that they can be finished in a single 3d print so that there are no strucutral issues during flight.


<div class="sketchfab-embed-wrapper"> <iframe title="Monocopter V2" frameborder="0" allowfullscreen mozallowfullscreen="true" webkitallowfullscreen="true" allow="autoplay; fullscreen; xr-spatial-tracking" xr-spatial-tracking execution-while-out-of-viewport execution-while-not-rendered web-share width="640" height="480" src="https://sketchfab.com/models/1a9efab045b24908a0da97baad3660d0/embed"> </iframe> <p style="font-size: 13px; font-weight: normal; margin: 5px; color: #4A4A4A;"> <a href="https://sketchfab.com/3d-models/monocopter-v2-1a9efab045b24908a0da97baad3660d0?utm_medium=embed&utm_campaign=share-popup&utm_content=1a9efab045b24908a0da97baad3660d0" target="_blank" style="font-weight: bold; color: #1CAAD9;"> Monocopter V2 </a> by <a href="https://sketchfab.com/textzip?utm_medium=embed&utm_campaign=share-popup&utm_content=1a9efab045b24908a0da97baad3660d0" target="_blank" style="font-weight: bold; color: #1CAAD9;"> textzip </a> on <a href="https://sketchfab.com?utm_medium=embed&utm_campaign=share-popup&utm_content=1a9efab045b24908a0da97baad3660d0" target="_blank" style="font-weight: bold; color: #1CAAD9;">Sketchfab</a></p></div>

# Current Work
- Simulation in Gazebo/Webots
- Writing code for the flight controller
- Implementing MPC and other control algorithms 