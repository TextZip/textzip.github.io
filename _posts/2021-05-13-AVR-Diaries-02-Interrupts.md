---
title: Hands On with AVR - 02 Interrupts
date: 2020-05-13 16:23:20 +/-0530
categories: [Tutorials, AVR Dairies]
tags: [atmega328p,external interrupts,pin change interrupts,ISR]     # TAG names should always be lowercase
image: /assets/img/PortManipulation/arduino_label.png
---

Most processors have interrupts. Interrupts let you respond to "external/internal" events while doing something else. 

This article uses concepts that have been covered in the previous post "AVR Diaries 01 - Port Manipulation", if your not familiar with port manipulation, it is highly recommended that you go through the earlier parts before continuing any further.

## Understanding the need for interrupts
Before we dive headfirst into interrupts and all the superpowers they grant to us, let's take a step back and see why we need interrupts. Let's take a simple sketch where an LED on pin 13 is turned on every time we press a button connected to pin 9. The LED turns off right after we release the button. 
```c++
int prev_state = LOW;
int LED_state;
int current_state;

void setup() {
  pinMode(13, OUTPUT);
  pinMode(9, INPUT_PULLUP);
}

void loop() {
  current_state = digitalRead(9);
  current_state = !current_state;
  if(current_state ==HIGH && prev_state==LOW){
    LED_state = HIGH;
  }
  else if(current_state ==LOW && prev_state==HIGH){
    LED_state = LOW;
  }
  prev_state=current_state;
  digitalWrite(13, LED_state);
}
```
This program on its own isn't that great and doesn't seem to be much useful, let us consider that the above sketch is part of a bigger program where an math intensive task runs in the void loop for about 2 second along with the above program.
The resulting code would look like this 
```c++
int prev_state = LOW;
int LED_state;
int current_state;

void setup() {
  pinMode(13, OUTPUT);
  pinMode(9, INPUT_PULLUP);
}

void loop() {
  current_state = digitalRead(9);
  current_state = !current_state;
  if(current_state ==HIGH && prev_state==LOW){
    LED_state = HIGH;
  }
  else if(current_state ==LOW && prev_state==HIGH){
    LED_state = LOW;
  }
  prev_state=current_state;
  digitalWrite(13, LED_state);
  delay(5000);
}
```
You can immediately notice that the push button is no longer responsive, and some of the inputs are not registered and processed. The Arduino is busy doing some other heavy math tasks for 5 seconds every loop cycle. Therefore the Arduino will not record any inputs during these 5 seconds. 

When we start writing bigger and more complex projects in arduino, reaction time to events is very crucial and interrupts are the arduinos way of dealing with time critical events.

## Interrupts & their types
A very technical and broad definition would be that
> An Interrupt is a signal emitted by hardware or software when a process or an event needs immediate attention. It alerts the processor to a high priority process requiring interruption of the current working process.

A more simple and understandable definion would be 
> An Interrupt's job is to make sure that the processor responds quickly to important events. When a certain signal is detected, an Interrupt (as the name suggests) interrupts whatever the processor is doing, and executes some code designed to react to whatever external stimulus is being fed to the Arduino. Once that code has wrapped up, the processor goes back to whatever it was originally doing as if nothing happened.

Arudino Uno has a wide varitiy of interrupts each specifically designed for a paticular task.
We will be looking at three of them:
1. External Interrupts
2. Pin Change Interrupts
3. Timer Interrupts (covered in the next post)

## External Interrupts
These are the most commonly used interrupts, like the name suggests they are responsible for triggering an interrupt based on changes in signals originating outside the arduino, like button presses for example. Note that not all pins are capable of this and Digital Pins 2 and 3 are the only two pins that can detect external interrupts in the Arduino uno.

External interrupts can be setup via two different methods we will now be looking at both the methods. 
### Method 1
We will be using the external interrupt on digital pin 2, to toggel an LED on pin 13 everytime we press we button. The trigger mode is set to rising.

```c++
volatile int state = LOW; 
const int led = 13;
const int button = 2;

void setup(){
  pinMode(led, OUTPUT);
  pinMode(button,INPUT_PULLUP);
  attachInterrupt(digitalPinToInterrupt(button),blinker,FALLING);
}

void blinker(){
  state = state == HIGH ? LOW : HIGH;
  digitalWrite(led,state);
}
void loop()
{
 delay(5000);//some random process to show that interrupts do interrupt.
}

```
- General syntax for `attachInterrupt()` is `attachInterrupt(digitalPinToInterrupt(pin), ISR, mode)`
	- The first parameter to `attachInterrupt()` is an interrupt number. Normally you should use `digitalPinToInterrupt(pin)` to translate the actual digital pin to the specific interrupt number. For example, if you connect to pin 3, use `digitalPinToInterrupt(3)` as the first parameter to `attachInterrupt()`.
	-  the ISR to call when the interrupt occurs; this function must take no parameters and return nothing. This function is sometimes referred to as an interrupt service routine.
	-  mode: defines when the interrupt should be triggered. Four constants are predefined as valid values.
		-  **LOW** to trigger the interrupt whenever the pin is low,

		- **CHANGE** to trigger the interrupt whenever the pin changes value

		- **RISING** to trigger when the pin goes from low to high,

		- **FALLING** for when the pin goes from high to low.

### Method 2
We will be working on the same example that was demonstrated in method 1, but we will now be using port manipulation instead of pre-defined functions.

The three main registers we will be working with are **EICRA, EIMSK, EIFR**.
- **EICRA (External Interrupt Control Register A)**
This register contains 4 bits two each for one external interrupt pin. They are used to select the trigger mode for the external interrupt. For our current example we are only conserned with the external interrupt on digital pin 2 which is refered to as **INT0**.
- **EIMSK (External Interrupt Mask Register)**
This register has two bits one for each of the external interrupts, they are called **External Interrupt Request Enable** bits, when set to high along with the Global Interrupts being enabled in the **SREG**. The external interrupt for that paticular pin will be enabled. Activity on the pin will cause an interrupt request even if INT1 is configured as an output. The corresponding interrupt of External Interrupt Request is executed from the Interrupt Vector.
- **EIFR (External Interrupt Flag Register)**
This register contains two bits one each for the external interrupts, When an edge or logic change on the INTx pin triggers an interrupt request, INTFx becomes set (one). If the Ibit in SREG and the INTx bit in EIMSK are set (one), the MCU will jump to the corresponding Interrupt Vector.The flag is cleared when the interrupt routine is executed. Alternatively, the flag can be cleared by writing a logical one to it. This flag is always cleared when INTx is configured as a level interrupt.

```c++
ISR(INT0_vect)
{
  PORTB ^= (1 << 5); // Toggle the state of the LED
}

void setup()
{
  cli();
  //Enable D13 as OUTPUT
  DDRB |= (1 << 5);
  //Enable D2 as INPUT_PULLUP
  DDRD &= ~(1 << 2);
  PORTD |= (1 << 2);

  //Enable Falling edge
  EICRA = 0;
  EICRA |= (1 << ISC01);
  //Enable Mask
  EIMSK = 0;
  EIMSK |= (1 << INT0);
  //Enable global interrupts
  sei();
}
void loop()
{
  delay(5000); //some random process to show that interrupts do interrupt.
}
```
`sei()` - Enable Global Interrupts by setting the I-bit in the SREG register to one.
`cli()` - Disable Global Interrupts by setting the I-Bit in the SREG register to zero.

## Pin Change Interrupts
The Pin Change Interrupt is triggered when the state of any of the pins with the PCINT enabled is changed. We primarly deal with three registers **PCICR, PCIFR, PCMSKx**.
- **PCICR (Pin Change Interrupt Control Register)**
When the **PCIEx** bit is set (one) and the **I-bit** in the **Status Register (SREG)** is set (one), pin change interrupt for that paticular port is enabled. Any change on any enabled **PCINT** pins will cause an interrupt. The corresponding interrupt of Pin Change Interrupt Request is executed from the **PCIx Interrupt Vector**. **PCINT** pins are enabled individually by the **PCMSKx** Register
- **PCIFR (Pin Change Interrupt Flag Register)**
When a logic change on any **PCINT** pins triggers an interrupt request, **PCIFx** becomes set (one). If the **I-bit** in **SREG** and the **PCIEx** bit in **PCICR** are set (one), the MCU will jump to the corresponding Interrupt Vector. The flag is cleared when the interrupt routine is executed. Alternatively, the flag can be cleared by writing a logical one to it.
- **PCMSKx (Pin Change Mask Register x)**
Each **PCINT-bit** selects whether pin change interrupt is enabled on the corresponding I/O pin. If **PCINT** is set and the **PCIEx** bit in **PCICR** is set, pin change interrupt is enabled on the corresponding I/O pin. If **PCINT** is cleared, pin change interrupt on the  corresponding I/O pin is disabled.

> NOTE: Pin Change Interrupts do not have mode selection, therefore any change in the logic level will trigger an interrupt.

Let us now look at a sketch that we earlier worked with using external interrupts, we will be using pin change interrupts on PORT B and C with masks enabled only for pins A1 and D9 to toggle and LED connected to digital pin 13. Given that a single press of the push button will raise the pin change interrupts twice as its senstive to any change and not just rising or falling, we also will be using a small polling based counter to toggle the LED like before.

```c++
volatile int counter_1 =0;
volatile int counter_2 =0;
ISR (PCINT0_vect) //ISR for Digital Pin 9
 {
  counter_1++;
  if (counter_1%2 ==0){ 
    PORTB ^= (1<<5); 
  }
  
 }
ISR (PCINT1_vect) // ISR for A1 Pin
 {
  counter_2++;
  if (counter_2%2 ==0){
    PORTB ^= (1<<5); 
  }
  
 }
void setup() {
  cli(); // Disable Global Interrupts
  PCICR = 0; 
  PCICR |= (1<<PCIE0)|(1<<PCIE1); // Enable Pin Change Interrupts for PORT B and C
  PCMSK0 = 0; // Reset Masks
  PCMSK1 = 0; // Reset Masks
  PCMSK0 |= (1<<PCINT1); // Enable Pin Change mask for A1 Pin
  PCMSK1 |= (1<<PCINT9); //// Enable Pin Change mask for Digital Pin 9
  sei(); // Enable Global Interrupts
  pinMode(13,OUTPUT); 
  pinMode(A1,INPUT_PULLUP);
  pinMode(9,INPUT_PULLUP);
}
void loop() {
  delay(5000); //some random process to show that interrupts do interrupt.
} 
```

We can also add an millis based cooldown time once a trigger has been called to take care of debouncing and multiple interrupts being called for a single press. Further, we can even save the states of the pins and compare them to the states of the pins as soon as the interrupt has been called to find out which pin caused the interrupt to be triggered. These two applications are left for you to try them on your own.

## Input Servie Routine
ISRs are special kinds of functions that have some unique limitations most other functions do not have. An ISR cannot have any parameters, and they shouldn’t return anything.

Generally, an ISR should be as short and fast as possible. If your sketch uses multiple ISRs, only one can run at a time, other interrupts will be executed after the current one finishes in an order that depends on the priority they have. millis() relies on interrupts to count, so it will never increment inside an ISR. Since delay() requires interrupts to work, it will not work if called inside an ISR. micros() works initially but will start behaving erratically after 1-2 ms. delayMicroseconds() does not use any counter, so it will work as normal.

Typically global variables are used to pass data between an ISR and the main program. To make sure variables shared between an ISR and the main program are updated correctly, declare them as volatile.

**Quick Facts about ISR**
- ISR's have a priority order incase two or more of them are called at once.
- Once inside an ISR the global interrupts are turned off, to prevent nested interrupt calls and once the ISR has finished its service the global interrupts are turned back on. You can manually turn on global interrupts inside an ISR for some crazy results.
- Do not use Serial, Delay and other heavy functions inside an ISR.
- External interrupts, pin-change interrupts, and the watchdog timer interrupt, can also be used to wake the processor up. This can be very handy, as in sleep mode the processor can be configured to use a lot less power (eg. around 10 microamps). A rising, falling, or low-level interrupt can be used to wake up a gadget (eg. if you press a button on it), or a "watchdog timer" interrupt might wake it up periodically (eg. to check the time or temperature). Pin-change interrupts could be used to wake the processor if a key is pressed on a keypad, or similar. The processor can also be awoken by a timer interrupt (eg. a timer reaching a certain value, or overflowing) and certain other events, such as an incoming I2C message.
- You must have seen me using cli() to disable interrupts quite often in the examples covered above, this is to make sure an interrupt isn't triggered while I am writing data to the registers.

## External Resources 

<iframe width="640" height="385" src="https://youtube.com/embed/J61_PKyWjxU" frameborder="0" allowfullscreen></iframe>

[Nick Gammon’s notes on Interrupts](http://gammon.com.au/interrupts)