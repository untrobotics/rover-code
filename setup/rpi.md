# Setting up the Raspberry Pi (RPi)

In this section, we’ll go over setting up the Raspberry Pi (RPi) and setting up all the code that will run the rover. Our rover uses ROS (Robotic Operating System); we will set these up below.

These instructions should work for both the RPi 3 and 4. You are free to use other versions of RPi, ROS, or OS, but setting these up is not covered here and it is not guaranteed that those will work.

## 1 Installing Ubuntu

The first step is to install the Ubuntu Operating System on your Raspberry Pi.

Download Ubuntu 20.04 from [here](https://ubuntu.com/download/raspberry-pi) for your RPi version. For an RPi3 the best default is 32-bit, but if you're on an Rpi4 then you can (should) go for 64-bit.

Preparing the image for the RPi and boot it up:

- Flash Ubuntu onto your microSD card. There are instructions for doing this on [Ubuntu](https://ubuntu.com/tutorials/create-an-ubuntu-image-for-a-raspberry-pi-on-ubuntu#1-overview), [Windows](https://ubuntu.com/tutorials/create-an-ubuntu-image-for-a-raspberry-pi-on-windows), and [Mac](https://ubuntu.com/tutorials/create-an-ubuntu-image-for-a-raspberry-pi-on-macos)
- Attach a monitor and keyboard to the RPi (note an alternative is to use `screen`, see [here](https://elinux.org/RPi_Serial_Connection))
- Insert the flashed SD card in the RPi
- Power it on
- Login, using "ubuntu" for the username and password. You should be prompted to make a new password.

You should now be logged in to your newly minted copy of Ubuntu!
As you will have noticed, there is no Graphical User Interface (GUI). We installed the bare minimum, the 'server' version of Ubuntu because you don't
necessarily need all that (it takes up a lot of computer power). However we recommend for those new to Linux to install the GUI as below.

## 2 Further setup: wifi, desktop GUI (optional), ssh

### 2.1 Connect to wifi from the command line

Get your new device on the internet. Instructions [here](https://linuxconfig.org/ubuntu-20-04-connect-to-wifi-from-command-line). 

1. Basically, you need to edit the `/etc/netplan/50-cloud-init.yaml` file and add your wifi network)
2. `SSID-NAME-HERE` is your network's name, and `PASSWORD-HERE` is the password for it.
3. After following these steps, you should see an ip address assigned in the output of `ip a`. It will be an `inet` value like `192.168.1.18`, underneath an interface entry like `wlan0`

### 2.2 Install a desktop GUI environment (optional)

This is a good option for newbies to the linux world. It's pretty easy to do, though it'll take a while (maybe an hour).

Follow the instructions [here](https://phoenixnap.com/kb/how-to-install-a-gui-on-ubuntu#htoc-update-repositories-and-packages).

1. We recommend using SLiM as the Display Manager, it seems lighter weight than the other options
2. We also recommend using GNOME for the GUI
3. Note that you'll probably need to `sudo tasksel` (instead of without sudo, per the instructions), otherwise you'll get a permissions error.

### 2.3 Enable SSH

You probably will also want to connect to your newly configured RPi remotely over ssh, rather than having to using a separate monitor every time. Instructions [here](https://askubuntu.com/a/681768).

1. Basically, run `sudo systemctl enable ssh.socket` from the command line
3. Now you should be able to login from your dev machine. `ssh ubuntu@192.168.1.18`, using the ip address for your RPi that you found above.
4. It should prompt you for a password. Once you enter it successfully, you'll be logged on! The `enable` step above should configure the ssh server to automatically come up on reboot, so you can just login to the RPi remotely from now on.

## 3 Installing ROS

We'll install ROS2 (Robot Operating System) Foxy on the RPi. If you're new to ROS, we recommend learning it as it is a crucial part in the code base.

You'll need to be logged in to the RPi via ssh, or open a terminal in the desktop GUI if you installed it above.

Follow the [instructions](https://index.ros.org/doc/ros2/Installation/Foxy/Linux-Install-Debians/) for installing ROS2.

You can choose to either install the 'full version' (`sudo apt install ros-foxy-desktop`
) which comes with graphical packages like RViz and QT or install just the barebones version (`sudo apt install ros-foxy-ros-base`). The latter
allows you to install packages in the full version whenever you need them. If you didn't install the GUI on Ubuntu, definitely install the base
version as you will have little use for GUI applications in the full version.

## 4 Setting up ROS environment and building the rover code

### 4.1 Setup ROS build environment

First we'll create a ROS workspace for the rover code. 

```
# Create a colcon workspace directory, which will contain all ROS compilation and 
# source code files, and navigate into it
mkdir -p ~/osr_ws/src && cd ~/osr_ws

# Source your newly created ROS environment
source /opt/ros/foxy/setup.bash

# install the build tool colcon
sudo apt install python3-colcon-common-extensions
```

### 4.2 Clone and build the rover code
For this section, you'll be working with the version control software `git`. Now's a good time to [read up](https://try.github.io/) on how that works if you're new to it and make a GitHub account!
In the newly created catkin workspace you just made, clone this repo:
```commandline
sudo apt install git
cd ~/osr_ws/src
git clone https://github.com/nasa-jpl/osr-rover-code.git
cd osr-rover-code
git fetch origin
git checkout foxy-devel

# install the dependencies using rosdep
sudo apt install python3-rosdep
cd ..
sudo rosdep init
rosdep update
rosdep install --from-paths src --ignore-src --rosdistro=foxy
# build the ROS packages
colcon build --symlink-install
```
It should run successfully. If it doesn't, please ask on Slack or [submit an issue](https://github.com/nasa-jpl/osr-rover-code/issues/new).

Now let's add the generated files to the path so ROS can find them
```
source install/setup.bash
```

The rover has some customizable settings that will overwrite the default values. 
Whether you have any changes compared to the defaults or not, you have to manually create these files:
```
cd ~/osr_ws/src/osr-rover-code/ROS/osr_bringup/config
touch osr_params_mod.yaml roboclaw_params_mod.yaml
```

To change any values from the default (if your rover doesn't match the default instructions), modify these files (the _mod.yaml ones) instead if the original ones. This way your changes don't get committed to git. 
The files follow the same structure as the default. Just include the values that you need to change as the default
values for other parameters may change over time.

You might also want to modify the file `osr-rover-code/ROS/osr_bringup/launch/osr_mod_launch.py` to change the velocities the gamepad controller will
send to the rover. These values in the node joy_to_twist are of interest:
```
    {"scale_linear": 0.8},  # scale to apply to drive speed, in m/s: drive_motor_rpm * 2pi / 60 * wheel radius * slowdown_factor
    {"scale_angular": 1.75},  # scale to apply to angular speed, in rad/s: scale_linear / min_radius
    {"scale_linear_turbo": 1.78},  # scale to apply to linear speed, in m/s
```
The maximum speed your rover can go is determined by the no-load speed of your drive motors. The default no-load speed is located
in the file [osr_params.yaml](../ROS/osr_bringup/config/osr_params.yaml) as `drive_no_load_rpm`, unless you modified it in the corresponding `_mod.yaml` file. 
This maximum speed corresponds to `scale_linear_turbo` and can be calculated as `drive_no_load_rpm * 2pi / 60 * wheel radius (=0.075m)`.
Based on this upper limit, let's set our regular moving speed to a sensible fraction of that which you can configure to your liking.
Start with e.g. 0.75 * scale_linear_turbo. If you think it's too slow or too fast, simply scale it up or down.

The turning speed of the rover, just like a regular car, depends on how fast it's going. As a result, `scale_angular`
should be set to `scale_linear / min_radius`. For the default configuration, the `min_radius` equals `0.45m`.

### 4.3 Add ROS config scripts to .bashrc

The `source...foo.bash` lines above are used to manually configure your ROS environment. We can do this automatically in the future by doing:
```
cd ~
echo "source /opt/ros/foxy/setup.bash" >> ~/.bashrc 
echo "source ~/osr_ws/install/setup.bash" >> ~/.bashrc
```
This adds the `source` lines to `~/.bashrc`, which runs whenever a new shell is opened on the RPi - by logging in via ssh, for example. So, from now on, when you log into the RPi your new command line environment will have the appropriate configuration for ROS and the rover code.


## 5 Setting up serial communication on the RPi

The RPi will talk to the motor controllers over serial.

### 5.1 Disable serial-getty@ttyS0.service

Because we are using the serial port for communicating with the roboclaw motor controllers, we have to disable the serial-getty@ttyS0.service service. This service has some level of control over serial devices that we use, so if we leave it on it we'll get weird errors ([source](https://spellfoundry.com/2016/05/29/configuring-gpio-serial-port-raspbian-jessie-including-pi-3-4/)). Note that the masking step was suggested [here](https://stackoverflow.com/a/43633467/4292910). It seems to be necessary for some setups of the rpi4 - just using `systemctl disable` won't cut it for disabling the service.

**Note that the following will stop you from being able to communicate with the RPi over the serial, wired connection. However, it won't affect communication with the rpi with SSH over wifi.**

```
sudo systemctl stop serial-getty@ttyS0.service
sudo systemctl disable serial-getty@ttyS0.service
sudo systemctl mask serial-getty@ttyS0.service
```

### 5.2 Copy udev rules

Now we'll need to copy over a udev rules file, which is used to configure needed device files in `/dev`; namely, `ttyS0 and ttyAMA0`. Here's a [good primer](http://reactivated.net/writing_udev_rules.html) on udev. 

```
# copy udev file from the repo to your system
cd ~/osr_ws/src/osr-rover-code/config
sudo cp serial_udev_ubuntu.rules /etc/udev/rules.d/10-local.rules

# reload the udev rules so that the devices files are set up correctly.
sudo udevadm control --reload-rules && sudo udevadm trigger
```

This configuration should persist across RPi reboots.

### 5.3 Add user to tty and dialout groups

Finally, add the user to the `tty` and `dialout` groups:
```
sudo adduser $USER tty
sudo adduser $USER dialout
```
You might have to create the dialout group if it doesn't already exist.

<!-- You'll need to log out of your ssh session and log back in for this to take effect. Or you can restart Ubuntu. -->

### 5.4 Remove console line in cmdline.txt boot config file

Do the following steps:

```
cd /boot/firmware
sudo cp cmdline.txt cmdline.txt.bak
sudo nano cmdline.txt
```

- And then delete the substring `console=serial0,115200` from the single line of text in the file. Save and exit.

You can confirm that you edited the file correctly using `cat cmdline.txt` from the command line, and inspecting the output.

Note that the cmdline.txt.bak is just a backup file in case you need to revert this change at some point (that file won't affect the boot process)

For more background on why we do this, see [serial_config_info.md](serial_config_info.md)

### 5.5 Disable bluetooth in config.txt boot config file

Execute the following commands

```
cd /boot/firmware
sudo cp config.txt config.txt.bak
sudo nano config.txt
```

- And then add the new line `dtoverlay=disable-bt` immediately after the existing line `cmdline=cmdline.txt` towards the bottom of the file

### 5.6 Restart the RPi

We need to restart for all of these changes to take effect. Execute: `sudo reboot now`

## 6 Testing serial comm with the Roboclaw motors controllers

Run the roboclawtest.py script with all of the motor addresses:
```
cd ~/osr_ws/src/osr-rover-code/scripts
python3 roboclawtest.py 128
python3 roboclawtest.py 129
python3 roboclawtest.py 130
python3 roboclawtest.py 131
python3 roboclawtest.py 132
```
Each of these should output something like, within a very short execution time:
```
(1, 'USB Roboclaw 2x7a v4.1.34\n')
(1, 853, 130)
```

If the script seems to hang, or returns only zeros inside the parantheses, then you have a problem communicating with the given roboclaw for that address. Some troubleshooting steps in this cases:

- Make sure you followed the instructions in the "Setting up serial communication" section above, and the serial devices are configured correctly on the RPi.
- Also make sure you went through the calibration instructions from the [main repo](https://github.com/nasa-jpl/open-source-rover/blob/master/Electrical/Calibration.pdf) and set the proper address, serial comm baud rate, and "Enable Multi-Unit Mode" option for every roboclaw controller (if multi-unit mode isn't enabled on every controller, there will be serial bus contention.). If you update anything on a controller, you'll need to fully power cycle it by turning the rover off.
- If you're still having trouble after the above steps, try unplugging every motor controller except for one, and debug exclusively with that one.
