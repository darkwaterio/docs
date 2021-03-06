# Programming the 640

## Python

### Introduction

The Python libraries for the 640 board and some example scripts are available via our GitHub repository. To install them open a terminal window on your Raspberry Pi (unless you are running with only the command line) and enter the following:

``` bash
$ git clone https://github.com/darkwaterfoundation/darkwater_python_640.git
```

Next you need to navigate into the new directory so enter:

``` bash
$ cd ./darkwater_python_640
```

And once in there we can install the libraries with:

``` bash
$ sudo python setup.py install
```

### Example scripts

Once everything is installed we can have a play with the example scripts included in the download. As well as being useful to test each part of your board, they are also handy as a starting point when writing your own scrips.

Let's move into the examples directory and take a look at what is there.

``` bash
$ cd ./examples
```

If you list the files in this directory, you should see a few test scripts

``` bash
$ ls -al
```

#### 640motortest.py

This script will start each motor port, in the forwards direction, in turn from left to right and then do the same backwards. To run the script enter the following:

``` bash
$ python 640motortest.py
```

#### 640servotest.py

This script will move any servos connected to the servo headers left, then center, then right. To run the script enter the following:

``` bash
$ python 640servotest.py
```

#### 640steppertest.py

This script divides the 6 motor ports into 3 stepper motor ports. Motor 1 and 2 will be stepper 1, motor 3 and 4 will be stepper 2 and motor 5 and 6 will be stepper 3.

Each stepper will be moved forwards and backwards through 400 steps when the test script is run:

``` bash
$ python 640steppertest.py
```

### The 640 board API

Now you know everything works, it's time to write your own scripts. So create a new python script in your editor with a memorable name and add the following lines to import our libraries:

``` python
import time
from darkwater_640 import dw_Controller, dw_Motor, dw_Servo, dw_Stepper
```

#### Create a controller

The **dw_controller** object controls access to all the elements on the 640 board, so the first thing we need to do is create a controller - we pass in the address of the 640 board as a parameter - the default address is 0x60

``` python
dw = dw_Controller( addr=0x60 )
```

Now that we have the controller created, we can access all the connectors on the board.

#### Select a Motor

There are 6 motor ports on the 640 board numbered 1 to 6 from left to right (with the ports facing you ).

If we want to control a motor on port number 1 then we need to request the motor object for that port from our controller - this is very easily done with a single line

``` python
m1 = dw.getMotor(1)
```

#### Motor driving

There are two main commands that you can give a motor - to move in a direction and to stop. 

We'll start with the main command to stop the motor

##### off()

The off command will switch off the motor and apply the brakes

``` python
m1.off()
```

##### setMotorSpeed( *speed* )

We can also stop the motor by using the second command and passing a speed of 0

``` python
m1.setMotorSpeed(0)
```

The **setMotorSpeed** command allows you to specify the speed of each motor - there are two different speed ranges the first goes from *-255* to *255*, the second from *1000* to *2000*. 

If you are familiar with radio control vehicles and ESC motors then you will recognise the second range.

For now we'll concentrate on the first range.

To get your motor going forwards at full speed you should set its speed at 255

``` python
m1.setMotorSpeed(255)
```

To get your motor going backwards at full speed you should set its speed to -255

``` python
m1.setMotorSpeed(-255)
```

The numbers from 0 to the maximum in each direction will drive the motor at a slower speed, so for half speed forwards we'd use

``` python
m1.setMotorSpeed(125)
```

And for a slow speed backwards we can use

``` python
m1.setMotorSpeed(-50)
```

##### Alternate speed range

If you plan to move from a DC driven robot to an ESC motor powered robot then it makes sense to use the same conventions that will work on both, so you can also use the 1000 to 2000 speed range with the 640 board

To get your motor going forwards at full speed you should set its speed to 2000

``` python
m1.setMotorSpeed(2000)
```

For full speed reverse you should set the speed to 1000

``` python
m1.setMotorSpeed(1000)
```

And to stop the motor we can set the speed to the mid point which is 1500

``` python
m1.setMotorSpeed(1500)
```

As before, any number between 1500 and the maximum in each direction will drive the motor at a slower speed, so for half speed forward you'd set the speed to 1750

``` python
m1.setMotorSpeed(1750)
```

and half speed in revers would be 1250

``` python
m1.setMotorSpeed(1250)
```

#### Select a Servo

There are two servo ports on the 640 board. They are numbered 1 and 2 with number 1 to the left hand side and number 2 the closest to the motor ports.

You select a servo in the same manner as you select motors, by requesting a servo object from the controller - to select the first servo we use:

``` python
s1 = dw.getServo(1)
```

#### Servo control

Once you have a servo object there are currently three commands you can run.

##### off()

The off command will switch off your servo and stop any signals being sent to it.

``` python
s1.off()
```

##### setPWMuS( *microseconds* )

This command will allow you to set the PWM pulse to the Servo in microseconds. 

Most standard servos use a parameter value of 1000 for fully counter-clockwise, 2000 for fully clockwise, and 1500 for the middle - though you may have a wider range on your servo, so you should check the technical documentation for it to get the finer details.

``` python
s1.setPWMuS(1500) # middle
s1.setPWMuS(2000) # fully clockwise
s1.setPWMuS(1000) # fully counter clockwise
```

##### setPWMmS( *milliseconds* )

This command allows you to specify the PWM pulse in milliseconds rather than seconds.

``` python
s1.setPWMmS(1.5) # middle
s1.setPWMmS(2.0) # fully clockwise
s1.setPWMmS(1.0) # fully counter clockwise
```

#### Select a Stepper motor

You can control up to 3 stepper motors with the 640 board - each stepper motor uses two motor ports for 4 wire stepper motors and three motor ports for 5 wire stepper motors. 

Running 5 wire stepper motors is almost the same as 4 wire stepper motos but requires a small extra step which we'll explain at the end.

Each stepper motor is assigned to a pair of motor ports:
- **Stepper motor 1** - uses motor ports 1 and 2
- **Stepper motor 2** - uses motor ports 3 and 4
- **Stepper motor 3** - uses motor ports 5 and 6

The first step is to identify the two wires for each coil on your stepper motor (you may need to read the technical documentation for your motor to find this out) and attach these two wires to each port.

*Example: You have a stepper motor with 4 wires - orange, pink, yellow and blue. If the orange and pink wires for your stepper motor are attached to coil one then attach these wires to motor port 1, attach the two remaining wires to motor port 2.*

Once you have your stepper motor wired up you need to request the relevant stepper motor object from the controller.

``` python
stepper1 = dw.getStepper(1)
```

#### Stepper motor control

There are four commands for stepper motors. The first one you'll recognise

##### off()

The off command will switch off the stepper motor

``` python
stepper1.off()
```

##### setMotorSpeed( *rpm* )

This command allows you to set the speed of your stepper motor. Pass the number of revolutions per minute that you want your stepper motor to run at.

``` python
stepper1.setMotorSpeed(200)
```

##### oneStep( *direction*, *style* )

This command will move the stepper motor one step in your chosen direction -

- **dw_Controller.FORWARD** - set the stepper motor to move forward
- **dw_Controller.REVERSE** - set the stepper motor to move backwards

There are two stepping styles available - 

- **dw_Controller.SINGLE** - this is the simplest method of stepping which activates a single coil at a time to move and hold the motor. This method uses the  least amount of power.
- **dw_Controller.DOUBLE** - this is a slightly more complex method of stepping which uses to coils to move and hold the motor. This method uses twice as much power as the single step, but is more powerful.

``` python
stepper1.oneStep(dw_Controller.FORWARD, dw_Controller.SINGLE)
stepper1.oneStep(dw_Controller.REVERSE, dw_Controller.DOUBLE)
```

##### step( *steps*, *direction*, *style* )

If you want to move the stepper motor a set number of steps then you can use this command. This, however, will stop all processing until the motor has moved the specified number of steps.

``` python
stepper1.step(200, dw_Controller.FORWARD, dw_Controller.SINGLE)
stepper1.step(200, dw_Controller.REVERSE, dw_Controller.DOUBLE)
```

If you want more control and need to move two or more motors at the same time then you should use the **oneStep** command.

### Next steps