---
title: POV - Lightsaber
date: 2021-12-10 19:13:20 +/-0530
categories: [Projects, DIY]
tags: [ws2812b,POV,imu]     # TAG names should always be lowercase
# image_sliders:
#   - gamingConsole
image: /assets/img/POV-lightsaber/arc.jpg
---
![Image1](/assets/img/POV-lightsaber/propic.jpg){: .shadow}

A persistance of vision lightsaber heavly inspired by [bitluni](https://www.electromaker.io/project/view/pov-light-saber)(original build). 


# Introduction
A LED Lightsaber equipped with an onboard IMU and ESP32 to detect the current angle of the lightsaber and accordinly light up the LED's to display images in the air as the saber is swung around.

[![TextZip/POV-lightsaber - GitHub](https://gh-card.dev/repos/TextZip/POV-lightsaber.svg)](https://github.com/TextZip/POV-lightsaber)
# Hardware
- ESP32
- 1 Meter WS2812B Strip (144 LEDS per meter density)
- Buck Converter
- 3S 35C 2200mAh Lipo
- MPU6050
- PVC Pipe
- Wooden strip
- Basic power tools  
# Software
Given that the lightsaber had 144LEDs and each LED is supposed to represent the color of 1 pixel, we needed to scale images to 128x128 pixels before they could be used in the light saber

Since the pivot point of the light saber will be at the bottom center of the image, the side length of the image given its diagonal length is fixed at 144 turns out to be 128 as shown in the figure below.

![Image1](/assets/img/POV-lightsaber/image_size.png){: .shadow}

The current angle of the lightsaber is caliculated using the MPU6050, the corosponding values of the LED colors needed are picked by overlaying a straight line over the image array and extracting the pixels that conincide with the above mentioned imaginary line. 
![Image1](/assets/img/POV-lightsaber/image_overlay.png){: .shadow}


# Results

<iframe width="640" height="385" src="https://youtube.com/embed/wkdZ0ptcQeU" frameborder="0" allowfullscreen></iframe>


<iframe width="640" height="385" src="https://youtube.com/embed/_GdPxGN2Kls" frameborder="0" allowfullscreen></iframe>


