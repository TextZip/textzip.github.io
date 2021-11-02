---
title: Gaming Wheel
date: 2019-01-10 19:13:20 +/-0530
categories: [Projects, DIY]
tags: [atmega328p,game,imu,touch]     # TAG names should always be lowercase
image_sliders:
  - gamingConsole
---
![Image1](/assets/img/GamingConsole/frontLaptop.jpg){: .shadow}

A cheap and affordable gaming console based on the Atmega-328p,
MPU6050 and capacitive touch sensors for an immersive gaming
experience.

## Hardware
- Arduino Uno
- ADXL345 (or any IMU or tilt sensor)
- Base Frame
- Large value resistors
- PVC or other material for structure

## Software
- Arduino IDE
- Processing/AHK
- Some game to test out the project

## Working
![Image1](/assets/img/GamingConsole/flowchart.png){: .shadow}


As the process description chart above states, the entire game plan takes user input and mimics keyboard or mouse activity. 

Keyboard, mouse, and other input devices are called HID Devices. While some microcontrollers like the Arduino Leonardo can be used as HID Devices, I decided to use an Arduino Uno to make things more challenging. 

I ended up using a 3-axis accelerometer and some make-shift capacitive touch sensors for taking user input. The Arduino Uno processes data from the sensors and sends commands via Serial Communication to Processing IDE, which conveniently can pull keyboard and mouse hooks, which help us interact with Softwares/games running on the system.   
## Build
### Prototype 
An early prototype to test the idea and feasibility was made using some spare cardboard lying around. 
<!-- Insert walk test video here -->
<iframe width="640" height="385" src="https://youtube.com/embed/OxRKLMEx7ZY" frameborder="0" allowfullscreen></iframe>

### Evolution Montage
{% include slider.html selector="gamingConsole" %}

## Results
### Rig Test
<iframe width="640" height="385" src="https://youtube.com/embed/8dTgWj4zzMw" frameborder="0" allowfullscreen></iframe>

### Vice City
<iframe width="640" height="385" src="https://youtube.com/embed/5FExViivi6Y" frameborder="0" allowfullscreen></iframe>

### Needs For Speed 
<iframe width="640" height="385" src="https://youtube.com/embed/6FIAbjZUHqM" frameborder="0" allowfullscreen></iframe>
