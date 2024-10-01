# JPL Open Source Rover Code
# Test
This repository contains the code that runs on the Raspberry Pi (RPi) and Arduino to control the 
[JPL open source rover (osr)](https://github.com/nasa-jpl/open-source-rover).
This includes the Arduino code that controls the LED matrix.
The rover runs on ROS2 (tested on Foxy), mostly using Python 3.

**Status**: actively developed. Please refer to the issues tab in GitHub for an overview of ongoing work.

## Internals & structure

Please refer to README files associated with each folder for insight in how components work and what they do. 
This is also the place to look when you have modifications on your rover that require the code or parameters to be
changed.

* The [ROS overview](ROS/README.md) gives an overview of the setup related to ROS and links to specific implementations
such as how the drive and corner commands are being calculated
* The [Arduino readme](Arduino/README.md) details the code that runs on the Arduino, used to control the LED screen.

## Setup

The setup directory contains tutorial files for getting everything setup and configured for running the rover code. You should proceed through them in the following order:
- [Arduino setup](setup/arduino.md)
- [Raspberry Pi setup with Ubuntu 20.04](setup/rpi.md)
- [Rover code bringup](setup/rover_bringup.md)
