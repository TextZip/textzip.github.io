---
title: Dino Game Arduino Edition
date: 2021-06-14 01:10:20 +/-0530
categories: [Projects,DIY]
tags: [TFT,Dino,Arduino,game]     # TAG names should always be lowercase
math: true
#image_sliders:
#  - workspace
---

## Introduction
I always had a thing for wearable electronics, a few months ago I got a cheap TFT display to play around with and the ultimate goal was to get familiar with all the electronics so that I can try to scale everything down and make a (hopefully portable) smartwatch.

I stumbled upon the flappy bird game for arduino by [mrt-prodz](https://www.mrt-prodz.com/blog/view/2015/03/flappy-bird-clone-on-the-atmega328-arduino-uno) while looking for some good projects with the TFT display, after playing the game for a couple of times, I decided to build my own version of the classic "Dino Game" from chromium.

All the code files and assets for the project can be found in the following repo:

[![TextZip/Dino-Game-Arduino-Edition - GitHub](https://gh-card.dev/repos/TextZip/Dino-Game-Arduino-Edition.svg)](https://github.com/TextZip/Dino-Game-Arduino-Edition)

## Hardware & Schematics 
### Components 
1. Arduino Uno
2. Push Button
3. TFT Display 1.8 Inches (128x160)
4. Buzzer
5. A few jumpers

### Schematic
The TFT display I got is based on the ST7735 Driver IC while it also had an SD card slot I ended up saving all the required data in flash as I didn't have an SD card at hand. The wiring for the TFT was a bit of a pain but thanks to help from [Tweaking4all](https://www.tweaking4all.com/hardware/arduino/sainsmart-arduino-color-display/) I finally got it running. 

The other connections for the buzzer and the push button are quite easy and require no explination. 
I did notice that the flappy bird game didn't actually use an interrupt based trigger but rather had a normal digitalRead function. I decided to use the internal pull and the external interrupt triggered ISR for servicing the user input via the digital pin 2.

The Display uses SPI communication, so I ended up using the SPI hardware pins of the ardunio uno for communicating with the display. Tinkercad doesn't have a model for the TFT display, so I just labeled the connections and dropped them onto a breadboard.
![image1](/assets/img/DinoGame/schematic_tinkercad.png){: .shadow}

My final setup with everything jam-packed into a tiny breadboard, as I didn't have anything bigger than this with me. 
![image1](/assets/img/DinoGame/schematic_real.png){: .shadow}


Another view of my TFT display for anyone's reference
![image1](/assets/img/DinoGame/TFTDisplay_side.png){: .shadow}


## Software 
The software stack can be broken down into the following sub-tasks
### Make a box that jumps
This step was supposed to be fairly easy and people actually use the euler's integration method to give an object some force (accleration) and then compute the velocity and displacement numericially using a very small dt. 

$$acceleration = force / mass$$

$$Delta_{position} = velocity * dt$$

$$Delta_{velocity} = acceleration * dt$$


I for some reason ended up using the equation of motion of a projectile under constant accleration. 

$$y = ut + \dfrac{1}{2}at^2$$

Everytime the button was pushed the box ended up with an initial velocity of $30 m/s$, I played around with the value of accleration due to gravity and the initial velocity to get the right jump responce. 

![Image1](/assets/img/DinoGame/jumping_box.gif){: .center}
*Things ended up looking like this (PS: don't mind the moving cactus in the background)*
### Draw cactus and make it move around
I used some basic shapes (rounded rectangles) to make a cactus like object and gave it a constant veloctiy in the -ve x direction to make it look like the dino is moving forward.

I used the brush method that was discussed in the [mrt-prodz](https://www.mrt-prodz.com/blog/view/2015/03/flappy-bird-clone-on-the-atmega328-arduino-uno) post for animations, which basically meant I erased the old pixels by drawing the background color over them and then drew the required colors over the new pixels, therefore I had to change a few selected pixels in each frame rather than the whole 128x160 of them.

### Replace box with a dino
I next tried replacing the box with a dino(bitmap), it looked like the dino was skating over the floor, and was quite artificial. 

![Image1](/assets/img/DinoGame/dinoSkating.gif){: .shadow}

I decided to make the dino walk as the original game does that as well, after snipping the two walking frames from the sprite sheet, I tried a simple code where I switched between the frames to make it look like the dino is walking. 

### Walking vs Skating Dino
The above method was a partial success as the entire dino image was now having a weird flicker due to the update speed and I realised that I was updating the whole dino bitmap but most of the body was statonary and only the legs were supposed to move.


![image1](/assets/img/DinoGame/dinoFlicker.gif){: .shadow}


I ended up fixing this by breaking the 2 dino frames into 2 parts each (4 in total), where the top part of each frame was the body and the bottom part was the leg, luckily the animation was setup in such a way that the body of the dino never actually changed between the two frames and only the leg positions were changed, so I fixed the body and only updated the leg parts of the dino, this eliminated most or if not all the flicker and I ended up with a very smooth looking animation.

![image1](/assets/img/DinoGame/dinoWalking.gif){: .shadow}


### Adding Clouds, Sounds, Score and other minor details
I now added other small details like sounds, intro and game over frames. I also added the score display and other features. I wanted to give a cloudly background for the game as the original has as it not only helps in the asthetics but also in showing that the dino is moving relative to the background all the time.

![image1](/assets/img/DinoGame/Home.gif){: .shadow}

I tired using clouds from bitmaps but there was a bit too much of a flicker in the clouds and there were soo small that I their features hardly mattered, so I ended up replacing the bitmaps with regular rounded rectangles which improved the game performance a lot.

## Results
After fixing a few more bugs I finally had a working dino game

<iframe width="640" height="385" src="https://www.youtube.com/embed/1rK0tkbr8Z8" frameborder="0" allowfullscreen></iframe>


## Future Upgrades
While I don't have immeadieat plans for improvements, I do have a few features in mind that I wana implement.
- [ ] Add Lives
- [ ] Dino has powers (Fire breathing)
- [ ] New obstacles (Birds?)
- [ ] Remove the flicker due to jumps
