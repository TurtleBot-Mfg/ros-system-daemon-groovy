ROS system daemon for Groovy

This package provides functionality for automatically starting/stopping ROS

* creates ros user
* create log directory /var/log/ros
* stores PID in /var/run/ros/roscore-11311.pid
* ```rosctl``` command line tool for starting and stopping ROS
* ```ros-network``` network autoconfiguration tool queries Network Manager over D-Bus for the current network configuration


Special Thanks
----
The program ```rosctl``` was inspired by apachectl
Package was incluenced by the alternate aproach used by
Thomas Moulard <thomas.moulard@gmail.com> on ros_comm_upstart
