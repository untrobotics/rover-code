# Bringing up the rover code

Note that these instructions assume you followed the steps in [rpi setup](rpi.md).

## 1 Manual rover bringup

In a sourced terminal (`source /opt/ros/foxy/setup.bash && source ~/osr_ws/install/setup.bash`), run

```commandline
ros2 launch osr_bringup osr_launch.py
```
to run the rover.

Any errors or warnings will be displayed there in case something went wrong. If you're using the Xbox wireless controller,
command the rover by holding the left back button (LB) down and moving the joysticks. You can boost as described in
the [RPi setup](rpi.md) by holding down the right back button (RB) instead. If this isn't working for you, 
`ros2 topic echo /joy`, press buttons, and adjust `osr_launch.py` to point to the corresponding buttons and axes. If you have questions, please [submit an issue](https://github.com/nasa-jpl/osr-rover-code/issues/new) or post on the Slack forum.

If the joysticks aren't sending through data (nothing shows when you run `ros2 topic echo /joy`) in a separate terminal, make sure
they have the appropriate permissions; `sudo chmod a+rw /dev/input/event0`. (The specific number may vary, run `ls -l /dev/input/*` to find out).
Then try again. You might want to consider using udev rules to automate this which you will need for automatic bringup using systemd services below.
If you need help, please post on GitHub Discussions or on the Slack forum.

### Optional arguments

If you want the code to calculate and publish wheel odometry, launch with the argument `enable_odometry:=true`.
![](wheel_odom_example.png)
Odometry is used for localization and SLAM.

## 2 Custom osr_mod_launch.py file

If you want to customize your `osr_launch.py` file, make a copy of it in the same directory (`osr-rover-code/ROS/osr_bringup/launch/`) and name it `osr_mod_launch.py`. The [systemd script](../init_scripts/launch_osr.sh) will automatically find it.

This is useful, for example, when you don't have the LED screen. In that case you would just remove the `<node name="led_screen" pkg="led_screen" type="arduino_comm.py"/>` line in `osr_mod_launch.py`.

## 3 Automatic bringup with launch script

Starting scripts on boot using ROS can be a little more difficult than starting scripts on boot normally from
the Raspberry Pi because of the default permission settings on the RPi and the fact that that ROS cannot
be ran as the root user. The way that we will starting our rover code automatically on boot is to create
a service that starts our roslaunch script, and then automatically run that service on boot of the robot.
[Further information](https://www.linode.com/docs/quick-answers/linux/start-service-at-boot/) on system service scripts running at boot.

There are two scripts in the ”init_scripts” folder. The first is the bash file that runs the
roslaunch file, and the other creates a system service to start that bash script. Open up a terminal on the
raspberry Pi and execute the following commands.
```
cd ~/osr_ws/src/osr-rover-code/init_scripts
# use symbolic links so we capture updates to these files in the service
sudo ln -s $(pwd)/launch_osr.sh /usr/local/bin/launch_osr.sh
sudo ln -s $(pwd)/osr_paths.sh /usr/local/bin/osr_paths.sh
sudo cp osr_startup.service /etc/systemd/system/osr_startup.service
sudo chmod 644 /etc/systemd/system/osr_startup.service
```

Your osr startup service is now installed on the Pi and ready to be used. The following are some commands
related to managing this service which you might find useful:

| Description | Command |
| --- | --- |
| Start service | sudo systemctl start osr startup.service |
| Stop service | sudo systemctl stop osr startup.service |
| Enable service (runs on boot of RPi) | sudo systemctl enable osr startup.service |
| Disable service (doesn’t run on boot of RPi) | sudo systemctl disable osr startup.service |
| Check status of service | sudo systemctl status osr startup.service |
| View live service list | sudo journalctl -f |

**Note: We do not recommend enabling the service until you have verified that everything
on your robot runs successfully manually. Once you enable the service, as soon as you power
on the RPi it will try and run everything. This could cause issues if everything has not yet
been fully tested and verified. Additionally, if you are doing development of your own software
for the robot we suggest disabling the service and doing manual launch of the scripts during
testing phases. This will help you more easily debug any issues with your code.**

Once you have fully tested the robot and made sure that everything is running correctly by starting the rover code manually
via `ros2 launch osr_bringup osr_launch.py`, enable the startup service on the robot with the command below:
```
sudo systemctl enable osr_startup.service
```

At this point, your rover should be fully functional and automatically run whenever you boot it up! Congratulations and happy roving!!
