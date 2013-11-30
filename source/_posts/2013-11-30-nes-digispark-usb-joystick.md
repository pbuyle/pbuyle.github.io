---
title: NES USB Joystick Adapter with Digispark
allow_share: true
tags:
 - nes
 - digispark
 - arduino
---

The [Digispark](http://digistump.com/products/1) is a tiny Arduino compatible (with restrictions) USB enabled
development board. A while ago I baked their
[Kickstarter campaign](http://www.kickstarter.com/projects/digistump/digispark-the-tiny-arduino-enabled-usb-dev-board)
but didn't made anything with my Digisparks until this week. I finally tried to connect one to another item I bough long
ago, an original [NES](https://en.wikipedia.org/wiki/Nintendo_Entertainment_System) controller.

Consuming a NES controller with an Arduino is actually easy, a quick googling returns tons of working code. There is
even an [Arduino library](https://code.google.com/p/nespad/) which looks like it could work on a Digispark.  Exposing the
the Digispark as an USB Joystick is even easier, thanks to the DigiJoystick library which is itself based on the well
known [V-USB](http://code.google.com/p/vusb-for-arduino/) (if you ever googled the "_arduino usb_" keywords, then you've
already encountered v-usb).

I ended with the following code, which seems to do the job. Unfortunately, my NES controller revealed itself to be
broken, the _left_ button is always seen as pressed except when _down_ is pressed.
~~~.c
#include "DigiJoystick.h"

#define LATCH   1
#define CLOCK   2
#define DATA    5
#define UP      B00001000
#define DOWN    B00000100
#define LEFT    B00000010
#define RIGHT   B00000001
#define BUTTONS B11110000

byte state = 0;

void setup() {
  pinMode(LATCH,OUTPUT);
  pinMode(CLOCK,OUTPUT);
  pinMode(DATA,INPUT);

  digitalWrite(LATCH,HIGH);
  digitalWrite(CLOCK,HIGH);
}

void loop() {
  // Read controller's state
  state = 0;
  digitalWrite(LATCH,LOW);
  digitalWrite(CLOCK,LOW);
  digitalWrite(LATCH,HIGH);
  delayMicroseconds(2);
  digitalWrite(LATCH,LOW);
  state = ~digitalRead(DATA);
  for (int i = 1; i <= 7; i ++) {
    digitalWrite(CLOCK,HIGH);
    delayMicroseconds(2);
    state = state << 1;
    state = state + ~digitalRead(DATA) ;
    delayMicroseconds(4);
    digitalWrite(CLOCK,LOW);
  }
  // set USB Joystick state.
  DigiJoystick.setY((byte)127);
  if (state & UP) {
    DigiJoystick.setY((byte)0);
  }
  if (state & DOWN) {
    DigiJoystick.setY((byte)255);
  }
  DigiJoystick.setX((byte)127);
  if (state & LEFT) {
    DigiJoystick.setX((byte)0);
  }
  if (state & RIGHT) {
    DigiJoystick.setX((byte)255);
  }
  DigiJoystick.setButtons((char)((states>>4) & BUTTONS), (char) 0);
  DigiJoystick.delay(10);
}
~~~