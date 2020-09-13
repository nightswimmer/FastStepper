# FastStepper
------------

Arduino library to control fast moving stepper motors, with acceleration and endstops support. This library is intended to be used with stepper motor drivers that have digital inputs for STEP, DIRECTION and optionally SLEEP (like the ones used in 3d printers). It is not used to controll the stepper motors directlly, or with a H-bridge.

This library only work on 8-bit Atmel boards, like the Atmega328 or the Mega2560. It uses interrupts and timers in order to generate the steps, so it can output fast steps and precise steps at high frequencies while allowing more code to be run at the same time.

The library also processes accelerations curves, meaning we can define a minimum and maximum velocity and the desired acceleration. Then we just give it the target position and the library handles the rest. The supported velocities are between 31 and 32767 steps per second.

It also supports the usage of endstops, so we can home the positions and make sure the stepper never moves outside the desired positions.

We can control multiple steppers by creating multiple instances of the library. The maximum number of steppers that can be controlled depend on the available timers on the arduino in use.	Atmega328 can control 3 steppers, and the Atmega2560 can control 6 steppers.

Note that using all the timers may cause other libraries to stop working, and even arduino functions like analogWrite() or millis()

The low-level parts of the library (the code to generate the pulses) are based in the arduino Tone library, by Brett Hagman, so this library will have the same limitations

How to use:
------------

We need to create a FastStepper object for each stepper we want to control:

    FastStepper _stepper;

In setup(), we need to initialize the object by telling it what pins to use to control the stepper.

The STEP pin is used to send the step pulses to the motor driver.

The DIR pin is used to tell the driver in which direction the motor should move.

The SLEEP pin is used to stop powering the motor in certain situations (this one is optional).

	  _stepper.begin(STEP_PIN, DIR_PIN, SLEEP_PIN);

For everything to run, we also need to call the run() method on the loop. This method needs to be called often, in order to have smooth acceleration curves. This means no using long delays() in the rest of the code (which is always a good advice anyway ;)

	  _stepper.run();

This is the bare minimum for the library to work. To make ste stepper move to a specific position, use:

    _stepper.moveTo(2000);

Typically you will want to control the velocity and acceleration of the motor, this can be done with:

    _stepper.setMovementParameters(MIN_MOTOR_SPEED, MAX_MOTOR_SPEED, MOTOR_ACCELERATION);

When receiving a moveTo() oder, the motor will start moving at MIN_MOTOR_SPEED, then it will accelerate untill it reaches MAX_MOTOR_SPEED. It will then move at that speed untill it's close to the desired position. Then it will slow down again, so that by the time it reaches the position it will be running at MIN_MOTOR_SPEED again. It is also possible to move at a constant speed by passing the speed parameter to the moveTo method:

    _stepper.moveTo(2000, 200);

This will move 2000 steps at a constant speed of 200 per second.

------------
We can also set constraints, so that the movement is limited to specific coordinates, by using:

    _stepper.setPositionConstraints(MIN_MOTOR_POSITION, MAX_MOTOR_POSITION);

When using moveTo() commands outside of these values they will be ignored.

------------
It is also useful to use endstops, they serve two purposed. They make sure the motor never moves to a position where it could break something, and when our sketch boots up the motor can move to the initial or final endstop and know exactlly where it is. We do this by initializing the pins associated with each endstop. We can use only one or both endstops.

	_stepper.setStartEndstopPin(START_ENDSTOP_PIN);
	_stepper.setFinishEndstopPin(FINISH_ENDSTOP_PIN);

This will never allow the motor to move to a position where it could break something. In many cases it is also useful to make the motor move to touch one of the endstops so we can know exactlly where the mininum or maximum positions are exactlly. We can use one of the following methods for this (use only one of the two):

	_stepper.homeStart();
	_stepper.homeEnd();

    
