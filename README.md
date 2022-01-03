﻿# ROS robot simulation based on a Laser scanner!

This is a robot simulation in a circular path (like a racing path) which benefits from the Robot Operating System **(ROS)**. The robot is equipped with a laser scanner in front of the robot which provide distances from different obstacles.  

# Aims of this project!

In this project, it is supposed to move and control this robot autonomously without any collision with the walls surrounded the whole complex path. Meanwhile, it is also essential to have a simple user interface to communicate the robot in the case of increase/decrease the speed, resetting the robot position, etc. In addition, it is worth mentioning that all requirements should not be considered in more that two nodes. 

## How to run and compile...! 

In this project, since multiple terminals and the user interface were needed to be run, a launch file is considered. In order to run and compile the project just do as follows:

__`roslaunch laserscanner_robot_ROS launchfile.launch`__
```xml
<launch>
<node  name="world"  pkg="stage_ros"  type="stageros"  args="$(find laserscanner_robot_ROS)/world/my_world.world"  />
<node  name="controller"  pkg="laserscanner_robot_ROS"  type="controller"/>
<node  name="robot_service"  pkg="laserscanner_robot_ROS"  type="Robot_Services"  required="true"  output="screen"/>
</launch>
```

## Simple user interface
Here is the simple user interface to communicate with the robot by using arbitrary services.


| inputs               |Action                          |Report                         |
|----------------|-------------------------------|-----------------------------|
|"r"| <p  align="center">`Reset`            |           |
|"+"          |<p  align="center">`Increase the speed`            |Current Velocity:            |
|"-"          |<p  align="center">`Decrease the speed`|
|"q"|<p  align="center">` Exit`| 

By sending the command "+", the speed will be increased by 0.5 m/s.
By sending the command "-", the speed will be increased by 1 m/s.
By sending the command "q", whole nodes will be killed and exited successfully.
**Note :** The initial velocity is 0.0, so, to move the robot the user should increase the speed by command "+". 
**Note :** In order to have the best performance, the minimum and maximum velocity of the robot is bounded from 0 to 5 m/s. 

## ROS nods and services Structure

The whole block diagram of the running nodes and services in this project is illustrated as follows:

<p align="center">
<img src="https://github.com/dssdanial/LaserScanner_robot_ROS/blob/main/world/rosgraph.png">
</p>


 **Description:**
 
 - **Node: "*/world*":** provide the racing map and some required topics and services.
 - **Node: "*/robot_service*":** provide the desired services to communicate with robot.
 - **Node: "*/controller*":** Consist of the main configuration, control of the robot and some features such as obstacle avoidance, speed regulation, etc.
 - **Topic *"/cmd_velociy*" :** This topic provides the velocity of the robot from ***geometry_msgs*** package by a ***Twist*** type message, which involves both linear and angular velocities in a 3D dimension.
 - **Topic "/base_scan" :** This is a topic created by ***Stageros*** Node, from **sensor_msgs** package which provides **LaserScan** message, generating scan ranges.
 - **Topic *"/Odom"*:** This topic provides information about the robot's position, orientation, etc, from ***nav_msgs/Odometry*** package. This can help to know the real-time position and other helpful details, when controlling the robot.

## Code Description

 
 **1. controller-Node:**

 

> **1-1.Publisher and Subscriber:**
```C
ros::Publisher pub=nh.advertise<geometry_msgs::Twist>("/cmd_vel",1);
ros::Subscriber sub=nh.subscribe("/base_scan",1, clbk_fcn);
```
This node is defined as both publisher and subscriber. To control the robot, linear and angular velocity will be published through ***/cmd_vel*** topic. Also, to configure and obtaining the distances from different obstacles during the complex path, the robot subscribes from ***/base_scan*** topic which provide the laser details. 
.
> 
> **1-2. Control:**
```C
if (there is an obstacle in the front of the robot){
	if (the left distance is lower than the right distance){
		turn right;
		}
	else if ( the right distance is lower than the left distance){
		turn left;
		}
}
else{
drive straight based on the input velocity;
}
```
.
> **1-3. Laser Scanner Configuration:**

This configuration is defined in the callback function which provide information about distance from obstacles. In this function the laser scanner data is defined in three different area include: front side, left side, and right side which indicate the minimum distance from each side from obstacles. 
```C
for (int  k=300; k<=400; k++){
	set the minimum distance For front side;
}

for (int  k=0; k<=100; k++){
	set the minimum distance For right side
}

for (int  k=610; k<=710; k++){
	set the minimum distance For left side
}
```
Based on these ranges, the robot will be avoided from any collision.

.

> **1-4. Service:**

This is a callback function for our arbitrary service. This service gets two character as inputs, and outputs the desired speed chosen by user.The maximum speed is assumed 5.0 and the minimum is 0.0 (which prohibits the robot to go backward).

```C
bool  clbk_speedup(laserscanner_robot_ROS::Speedup::Request  &req, laserscanner_robot_ROS::Speedup::Response  &res){

defining some local variable like minimum and maximum speed;
defining some objects of Geometry messages;

if(user ask to increse the speed){ 
	Speed will be increased by +0.5 unit;
	if(user ask higher speed than the threshold){
		The maximum allowed value will be set; 
	}
}
if(user ask to decrease the speed){ 
	Speed will be decreased by -1 unit;
	if(user ask lower speed than the threshold){
		The Zero value will be set; 
	}
}

The final requested value For speed will be returened; 

return  true;
}
```
.

.
 **2. Robot_service Node:**
In this node, two client servers are defined, one for resetting the robot position, and another one is designed to increase/decrease the robot's speed.
```C
ros::ServiceClient client1 = nh.serviceClient<std_srvs::Empty>("reset_positions");
ros::ServiceClient client2 = nh.serviceClient<laserscanner_robot_ROS::Speedup>("speed_up");
```
The service is called ***Speedup.srv*** which is located in the **srv** folder.  
```xml
string data 
---
float32 result
```
The service file contains *String* type data as inputs, and a *float* data as output, which returns the robot's speed.
Also, this node can be consider as a **Subscriber** from the ***/odom*** topic to receive the monitor the information about the robot's position, orientation ,etc.



# Simulation
The simulation and results are illustrated in the following video:
[! (https://github.com/dssdanial/LaserScanner_robot_ROS/blob/main/world/tracciato.png)] (https://youtu.be/MY6nIdYGh3Q)

## Further improvements
To improve performance of the robot movements, some extra conditions can be added, which can make the movements more smooth, especially when higher speed level is requested.  
