# Prerequisites

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/7fc794d1f7084cda9c591378ec0ef8d2)](https://app.codacy.com/app/ro-boy/roboy_plexus?utm_source=github.com&utm_medium=referral&utm_content=ro-boy/roboy_plexus&utm_campaign=Badge_Grade_Dashboard)

1. Install dependencies
```
sudo apt install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf 
```

2. [Download](https://dl.altera.com/soceds/) and install Intel SoC FPGA Embedded Development Suite (Standard Edition).
3. Configure the environment using the `embedded_command_shell.sh` script
```
~/intelFPGA/17.0/embedded/embedded_command_shell.sh
```

# Usage
1. Launch ROS node on FPGA

  ```
  ssh root@192.168.240.240
  ~/.roboy_plexus
  ```
* look for `Gandalf Data Limited` in the following command output in order to find out FPGA's IP, substitute `192.168.0.255` by your network broadcast address
  ```
  sudo nmap -nsP 192.168.0.255/24
  ```
* if ssh doesn't work, connect the FPGA board to the PC via the USB port (use the one on the ethernet side of FPGA board) and connect to the serial monitor with
  ```
  screen /dev/ttyUSB0 115200
  ```
* if the board was rebooted and the ethernet network interface doesn't work try
  ```
  ifup eth0
  ```

* make sure the PC and FPGA know where to look for ROS, i.e. `ROS_MASTER_URI` and `ROS_IP` are set (usually in the ~/.bashrc)

* if `catkin_make` fails, set up FPGA dev environment by calling
  ```
  ~/intelFPGA/17.1/embedded/embedded_command_shell.sh
  ```
  
  

2. Status of the motors is published under the rostopic `roboy/middleware/MotorStatus`
  ``` 
  rostopic echo /roboy/middleware/MotorStatus
  ```

* check for available rostopic and services:
  ``` 
  rostopic list
  rosservice list
  ```

* type definitions for msgs and srvs are in `roboy_communication/middleware` or on https://github.com/Roboy/roboy_communication/tree/master/middleware

3. The control commands are set by publishing ROS messages under `/roboy/middleware/MotorCommand`:
  ```  
  rostopic pub /roboy/middleware/MotorCommand roboy_communication_middleware/MotorCommand "id: 0 
		motors: [0,1]
		setPoints: [0,0]"
  ```

4. (Optional) Configure motor mode by calling ROS service `/roboy/middleware/MotorConfig`
* available control modes:
	* 0: position (encoder ticks with `rad_per_encoder_count: 0.00005788606746738269`)
	* 1: velocity (rad/s) - untested!
	* 2: displacement (encoder ticks)
	* force mode is available only from GUI and requires a YAML file with spring constants 
	* [example transform from displacement to force](https://github.com/Roboy/roboy_rqt_plugins/blob/67975a98dddd83cdcb4ce4571191d6eeab046822/roboy_motor_command/src/roboy_motor_command.cpp#L125)
	* myomuscle calibration file is required, specify the path in GUI (start by calling `rqt`)
	* `Plugins->Roboy->roboy motor calibration`
	* use myomuscle tab

5. Start GUI (the plots show all the motors together encoded in different colors) 
  ```
  rqt
  ```
# Daemon
You can find the daemon in scripts/automaticRoboyPlexusLauncherDaemon, execute the following commands for installing the daemon on the fpga:
```
sudo mv automaticRoboyPlexusLauncherDaemon /etc/init.d/
echo "/etc/init.d/automaticRoboyPlexusLauncherDaemon start" >> ~/.bashrc
sudo ln -s /home/root/roboy_plexus /usr/local/bin/roboy_plexus 
```
Edit the file /lib/systemd/system/serial-getty@.service and replace the line:
```
ExecStart=-/sbin/agetty --keep-baud 115200,38400,9600 %I $TERM
```
with auto-login:
```
ExecStart=-/sbin/agetty --autologin root --keep-baud 115200,38400,9600 %I $TERM
```
This will auto-login root when the fpga boots and the daemon is started via the bashrc entry.
