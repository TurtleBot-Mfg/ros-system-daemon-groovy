# ROS system daemon for Groovy
This package provides functionality for automatically starting/stopping ROS

## ROS System User
    Username:           ros
    Home Directory:     /var/lib/ros
    Group:              ros
    Shell:              /bin/sh

*ros* should be a member of group *dialout* to access serial ports.

```chown -R ros:ros /var/lib/ros```  
```chmod 2775 /var/lib/ros```

    ROS Log Dir:        /var/log/ros
    ROS PID:            /var/run/ros/roscore.pid

## Startup and shutdown
Startup and shutdown is controlled by an upstart script that can be auto-configured via D-Bus communication with Network Manager. It also supports manual overrides by setting ```ROS_IP``` and ```ROS_INTERFACE``` in ```/etc/ros/setup.bash``` or ```/etc/ros/envvars```. ROS is started when upstart receives a *net-device-up* signal and it is stopped when a network interface stops.

Future work will resolve the issue where ROS starts connected to *eth0* and the machine is also equipped with a WiFi interface. In this case the state of the WiFi device should not produce a stop/start event effecting ROS.

For this we plan to separate the upstart scripts into two packages
* *ros-system-upstart-lan-groovy*
* *ros-system-upstart-wan-groovy*  

These packages will provide *ros-system-upstart-groovy* for dependency management.

### ros-system-daemon-groovy.ros.upstart
* Config file ```/etc/init/ros.conf```
* ```start on net-device-up IFACE!=lo```
* ```stop on platform-device-changed```
* Check/update ownership & permissions of ```/var/run/ros```
* Autoconf network ```ROS_IP=`ros-network ip````
* Start via 'setuidgid ros rosctl start'
* Stop via 'setuidgid ros rosctl stop'

## System-wide Logging
    Log Dir:            /var/log/ros
    chown -R ros:ros /var/log/ros
    chmod 2775 /var/log/ros

### ros-system-daemon-groovy.ros.logrotate
* Config file ```/etc/logrotate.d/ros```
* Rotate logs daily *.log -> *.log.1 -> *.log.2 -> etc
* Compress the previous days logs daily *.log.2 -> *.log.2.gz
* Keep up to one weeks logs for an active rosmaster
* Archive inactive log subdirectories daily
* Remove archived logs older than one week
* Rotation currently done by copying and truncating the active log

Note: 01 Jan 2013 Sending SIGHUP to the ROS does not cause it to write to a new log.  
https://github.com/ros/ros_comm/issues/45

## Files
### /usr/sbin/rosctl
Modelled after ```apachectl```, this script allows ros to be started
locally by users or system-wide by user *ros*.

The script will attempt to source configuration files as follows
* ```/etc/ros/setup.bash```
* ```/opt/ros/$ROS_DISTRO/setup.bash```
* ```/opt/ros/groovy/setup.bash```

If ```rosctl``` is run as user *ros* it will attempt to launch ```/etc/ros/robot.launch``` or the launch file specified in ```ROS_LAUNCH``` if it exists.

The PID file will be written to ```ROS_PID``` if it is set or ```/var/run/ros/roscore.pid``` or if it run as a local user it will be written to ```~/.ros/roscore-11311.pid``` or similar.

If ```gnome-session``` is running, the script will attempt to send desktop notifications via ```notify-send``` to the user that is logged in when ROS starts or stops.

Sub-commands
* start - Start ROS (roscore + default launch file)
* stop - Stop ROS (rosnode kill nodes then killall roslaunch)
* restart - Start followed by Stop
* status - Display the PID and user running ROS

In the near future running this as script as root will check/repair directory permissions and re-run itself setuidgid ros. Also, when run as root it should issue a warning and run ```initctl stop ros``` to prevent upstart from respawning. It may also be worth issuing a warning when run as *ros* and ```initctl status ros``` shows that upstart will respawn the process.

### /usr/sbin/ros-network
This tool autodetects network settings by querying NetworkManager via D-Bus. Autoconf can be overridden by setting ```ROS_IP``` and ```ROS_INTERFACE```.

Sub-commands
* interface - Outputs the primary interface name (wlan0, eth0, etc)
* ip - Outputs the ipv4 address of the primary network connection
* ssid - If WiFi connection, provide ssid of access point

In the event of autoconf failure, the script returns ```lo``` for the interface name and ```127.0.0.1``` for the ip address.

Eventually we would like this tool to be able to provide an automatic configuration of the ```ROS_MASTER_URI``` when a user logs in.

### /etc/ros/envvars
The idea behind using this instead of modifying setup.ash is that if the file only contains environmental variables it can be parsed and edited by hardware vendor install scripts.

## Special Thanks
The program ```rosctl``` was inspired by ```apachectl```
Package was influenced by the alternate approach used by
Thomas Moulard <thomas.moulard@gmail.com> on ros_comm_upstart
