Switec X25 library
==================

This is an Arduino library for driving Switec X25 miniature 
stepper motors. It was written specifically for the Switec X25.168,
and compatible steppers from other manufacturers including the 
VID29 and MCR1108.

For more information including [datasheets](http://guy.carpenter.id.au/gaugette/resources) see the [Gaugette blog](http://guy.carpenter.id.au/gaugette/).

Usage Notes
-----------
This library assumes you are driving the X25 directly from
the Arduino IO lines, not through an intermediate controller circuit.
A motor can be connected to any 4 Arduino digital output lines.
The Arduino Uno has 14 I/O lines, limiting you to three motors.
The Mega boards have more.

The rate at which these miniature stepper motors 
can accelerate is dependent upon the inertia
of the needle you are using.  You may need to tune the acceleration
tables in the library for your needle.


This Library Is ASYNCHRONOUS NON-BLOCKING
-----------------------------------------
If you read nothing more, please read and understand this:

The ```update()``` call is _non-blocking_.  It does _NOT_ move
the motor to the target position, it advances the motor _at most
just one step_ toward the target position and returns.

The correct way to use this library asynchronously is:

- Add a call to ```update()``` inside your main loop.  It should be called as frequently as possible.  It will return quickly if there is nothing to be done.  This means for example if you are waiting for serial I/O, you should be calling ```update()``` while you are waiting.

- Call ```setPosition()``` whenever you need to change the target position of the motor.

The advantage of this over a blocking/synchronous library is that you can
control many motors simultanously, and you wont have long periods of inactivity
(and potentially missed I/O events) while your motor is moving.

If instead you require a blocking call which will complete the motor action before returning:

- Call ```setPosition()``` to to the target position of the motore

- Call ```updateBlocking()``` which will move the motor smoothy to the position set by ```setPosition()```.

Using the Library
-----------------

```C++
#include "SwitecX25.h"

int steps = 945;

SwitecX25 motorRpm(steps, 2,3,4,5);
SwitecX25 motorSpd(steps, 6,7,8,9);
SwitecX25 motorFuel(steps, 10,11,12,13);

byte gear, spd, shift, rpm_h, rpm_l, engine;

void setup(void) {
  Serial.begin(115200);

  // run both motors against stops to re-zero
  motorRpm.zero();   // this is a slow, blocking operation
  motorSpd.zero();
  motorFuel.zero();
}

void loop(void) {
  if(Serial.available())
    if(Serial.available() > 8)
      if(Serial.available() == 255)
      {
        gear = Serial.parseInt();
				spd = Serial.parseInt() /300 * steps;
				rpm_h = Serial.parseInt();
				rpm_l = Serial.parseInt();
				fuel = Serial.parseInt() / 100 * steps;
				shift = Serial.parseInt();
				engine = Serial.parseInt();

				int rpm = (rpm_h << 8) | rpm_l;
        
        motorRpm.setPosition(rpm / 7000 * steps);
        motorSpd.setPosition(spd);
        motorFuel.setPosition(fuel);
      }
      
  motorRpm.update();
  motorSpd.update();
  motorFuel.update();
}

```












