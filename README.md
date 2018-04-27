# Mapping and Slam: Map My World Robot

### Udacity Robond Project

## Project Introduction

In this project you will create a 2D occupancy grid and 3D octomap from a provided simulated environment. Furthermore, you will then create your own simulated environment to map as well. 

Before we get started let’s take a moment to understand where we are and why we are doing this. 

After much testing and research we found RTAB-Map to be the best solution for SLAM to develop robots that can map environments in 3D. These considerations come from RTAB-Map's speed and memory management, its custom developed tools for information analysis and, most importantly, the quality of the documentation. Being able to leverage RTAB-Map with your own robots will lead to a solid foundation for mapping and localization well beyond this Nanodegree program. For this project we will be using the rtabmap_ros package, which is a ROS wrapper (API) for interacting with rtabmap. Keep this in mind when looking at the relative documentation.



### Project Steps

This project will draw heavily on previous knowledge in the course. As a roboticist, practice is key to understanding how everything works and how to efficiently debug problems when they arise.

The project flow will be as follows:

1. You will develop your own package to interface with the rtabmap_ros package.

2. You will build upon your localization project to make the necessary changes to interface the robot with RTAB-Map. An example of this is the addition of an RGB-D camera.

3. You will ensure that all files are in the appropriate places, all links are properly connected, naming is properly setup and topics are correctly mapped. Furthermore you will need to generate the appropriate launch files to launch the robot and map its surrounding environment.

4. When your robot is launched you will teleop around the room to generate a proper map of the environment. 

5. You then will build your own simulated environment and apply the code you used to map the supplied environment on this environment.

6. Finally you will create a report and organize the required files for submission. 

We are excited to see you grow with your ROS skills and we can't wait to see what you build!

### Notes

1. This project is intended to be completed in the Udacity Workspace or on Jetson TX2. You might encounter performance issue on personal laptop or the VM.

2. If you are working on Jetson TX2, you could use jetson_clocks.sh script in the home folder to speed it up! 


## Sensor Upgrades

The recommended configuration for RTAB-Map is as follows:
- 2D laser which publishes LaserScan sensor_msgs/LaserScan messages
- Odometry (IMU, wheel encoders, ...) which publishes nav_msgs/Odometry messages
- A calibrated RGB-D sensor

You may notice that a 2D laser scanner is listed and although it is recommended, it is not required. We will touch more on that shortly. 

### RTAB-Map Quick Sensor Overview

When building your ROS package you will need to make sure that you are publishing the right information for RTAB-Map to successfully work. In the image below you can get a visual overview of the high level connections enabling both mapping and localization. 

Before we move to how we are going to update our robot with an RGB-D camera, let's touch on how you could set up your robot (simulated and real) with just an RGB-D camera. Removing the laser scanner does not mean that we no longer need the scan topic, but that we need to find a new way to generate this scan topic. This is where the depthimage_to_laserscan package for the Kinect comes in. This package illustrates and provides a method of mapping the depth point cloud into a usable scan topic. This remapping can be viewed in the image below. 

This is a convenient solution for those without a laser scanner on their robot. Since we are upgrading our robot from the previous project, we will elect to keep the laser range finder and the scan topic it provides and sticking with RTAB-Maps optimal configuration. 

Furthermore, it can be noted that the scan input topic is a convenient way to efficiently create an occupancy grid based on 2D ray tracing. However, you can achieve the same results with depth image, but it is more costly to generate.

### Configuring the RGB-D Camera

In the previous project you constructed an RGB camera, now it's time to give that an upgrade. Specifically, we will simulate a Kinect camera. 

To do this we will need to replace the existing camera and its shared object file: libgazebo_ros_camera.so to that off the Kinect shared object file: libgazebo_ros_openni_kinect.so. On top of this, additional parameters need to be set for the RGB-D camera as well as matching the topics published by the drivers of its real world counterpart. We have provided an example for the camera code.

You may use the provided code, but you will need to make sure that you adjust for the required linkages in your URDF. Furthermore, you will need to be sure that you are publishing the correct topics and using the proper naming conventions. You will be taught some debugging techniques to help locate issues and resolve them in the upcoming concepts.

## Building Launch Files

You now know what needs to be done to upgrade your robot, so now it's time to turn our attention to the launch files for the project. There are infinitely many ways to set up your launch files. You could even go as far as having a single launch file for everything. However, this isn't advised, as there is a benefit to being able to launch each launch file in a dedicated terminal. Having each launch file in it's own respective terminal can serve important purposes for interaction and information visualization. 

In this project, you are required to create all of the necessary launch files. You have the resources to generate all of them except a mapping.launch file. We will walk through the creation of that file, but we expect you to assemble the rest to complete the project. Before we get to the mapping.launch file, let’s take a look at the ideal executable steps for this project to see what’s going on. 

A lot of these steps will seem familiar to you because you’ve seen them before in your last project. The new additions are:
- Launching your own teleop node
- Launching the mapping node

Your mapping launch file acts as the main node that interfaces with all the required parts to be able to perform SLAM with RTAB-Map. A labeled template for the mapping.launch file has been provided for you to use. Read through the code and the comments to understand what each part is accomplishing and why. Feel free to reach beyond this template with the resources linked below. Please note, that this template is sufficient in structure to complete the project!

At this point, you could assemble your components to build your ROS package, but you may be met with difficulties in getting it to run. There may be a mis-named topic or an improper linkage. So what can we do? Well, like all good programmers we turn to debugging and that is what you will learn next!

## Debugging with Transform Frames

While developing your robot’s URDF file, you created several links and joints. Each link had its own coordinate frame, along with additional coordinate frames for the world (map) and the odometry frames. 

The tf library helps keep track of all these different coordinate frames over time, and defines their relation with one another. 

The library allows you to debug your coordinate frames (or the tf tree) in several ways. view_frames will create a graphical representation of that tree, providing you with a broad view of how different frames (or links) in your setup connect to each other. 

Creating and reviewing the tf-tree is a great way to make sure that all of the links are in proper order. If, for instance, you saw that your laser sensor frame is connected to your right wheel’s coordinate frame, it would require you to take a step back and determine why it is not attached to the chassis.

You can also plot the different frames in RViz and visually inspect them there.

The tf library also allows you to dynamically collect and display information between two transforms using tf_monitor. This is great to make sure that two frames are associated in the expected way. 

With these simple tools you can rapidly and visually diagnose any tf issues. 

Sometimes those basic tests may not be enough. Here is a great tutorial if you find yourself in a more complicated tf problem. Though note that using these tools should not be necessary for this project, as the transforms are relatively simple compared to that of a robotic hand. 

Another debugging tool under the tf library that is useful for debugging your setup is roswtf. Let’s check it out next!

## Debugging with roswtf

Previously, in the EKF Lab Lesson you ran rqt_graph to visualize the ROS graph corresponding to that package. Here is an example for rqt_graph from the previous 2D SLAM lab. 

Often, you will find it useful to diagnose your system when it might not be behaving as you would expect. For such purposes, you can use roswtf. roswtf will examine and analyze the setup or the graph above, including any running nodes and environment variables, and warn you about any potential issues or errors.

Here is an output from the running of roswtf for the SLAM robot.

You can see the output contains 2 warnings - one regarding node subscribers not being connected and one regarding a node that has died. The significant outputs to look for are errors, but roswtf also outputs helpful warnings. Right now one can see some path related items and localization related items. Since no path generation or localization is going on, it’s okay to ignore these items. 

roswtf performs a lot of checks. For example, it checks if all running nodes are connected and communicating properly, or if any nodes have died. You can explore more such checks here.

## Debugging with rqt common plugins

RQT common plugins are a suite of tools you have seen before. We will mention several tools that you have seen before, but also learn several new ones as well!
Errors in ROS

### rqt console

ROS errors can be broken down into certain verbosity levels. This is helpful when trying to decide which problems to address first. The types are as follows:
- DEBUG
- INFO
- WARN
- FAILURE
- FATAL

Examples of each can be found here.

With all of these errors flying around all over the place, it can be hard to isolate a particular issue. In comes rqt_console. It aggregates all of the log messages and allows you to sort them. 

Per documentation each message includes:
- Message: The message specified by the user 
- Severity: The severity level of the message, e.g. Debug, Info, etc.
- Node: The name of the node which broadcast the message
- Time: The time at which the message was broadcast
- Topics: The topics advertised by the node broadcasting the message
- Location: Combines the file, function and line using colons

Source

To catch all of the messages, be sure to start rqt_console before launching any other ROS related code. 

### rqt_image_view

This is a great tool to visualize images streaming from camera and depth maps from sensors. Use it to make sure you are properly receiving the data you want on its specific topic.

### rqt _graph

As you saw in the previous project rqt graph is a great tool to visualize the connections of your robotic system. Be sure to use this to check that everything is working in its intended fashion. 

## Database Analysis

The rtabmap-databaseViewer is a great tool for exploring your database when you are done generating it. It is isolated from ROS and allows for complete analysis of your mapping session. 

This is how you will check for loop closures, generate 3D maps for viewing, extract images, check feature mapping rich zones, and much more! 

Let’s start by opening our mapping database:

We can do this like so: rtabmap-databaseViewer ~/.ros/rtabmap.db

Once open, we will need to add some windows to get a better view of the relevant information, so:
- Say yes to using the database parameters
- View -> Constraint View
- View -> Graph View

Those options are enough to start, as there are many features built into the database viewer!

Let’s talk about what you are seeing in the above image. On the left, you have your 2D grid map in all of its updated iterations and the path of your robot. In the middle you have different images from the mapping process. Here you can scrub through images to see all of the features from your detection algorithm. These features are in yellow. Then, what is the pink, you may ask? The pink indicates where two images have features in common and this information is being used to create neighboring links and loop closures! Finally, on the right you can see the constraint view. This is where you can identify where and how the neighboring links and loop closures were created.

You can see the number of loop closures in the bottom left. The codes stand for the following: Neighbor, Neighbor Merged, Global Loop closure, Local loop closure by space, Local loop closure by time, User loop closure, and Prior link.

When it comes time to design your own environment, this tool can be a good resource for checking if the environment is feature-rich enough to make global loop closures. A good environment has many features that can be associated in order to achieve loop closures. 

## Real time mapping visualization

Another tool that you can use is rtabmapviz, which is an additional node for real time visualization of feature mapping, loop closures, and more. It’s not recommended to use this tool while mapping in simulation due to the computing overhead. rtabmapviz is great to deploy on a real robot during live mapping to ensure that you are getting the necessary features to complete loop closures.

This will launch the rtabmapviz GUI and provide you with realtime feature detection, loop closures, and other relevant information to the mapping process.

### Supplied Files

We will provide you with the required environment you will need to map and other assorted files. These will not be provided in the same structured order that you saw the Localization project. You will be building, placing, and modifying all the required files from scratch. 

Hint: Consider using a similar structure to that of the Localization project. 

Note 1: A telop file is included and can be placed in your scripts folder. You may need to give it executable permissions. You will also need to create a launch file for this telop file. 

Note 2: Another one of the provided scripts, called rtab_run, will help you expedite the testing of launch files and the project. You may need to adjust the paths, naming, and other variables accordingly. It is not required to use this file, but it can help save time! 

Note 3: Be sure to add the rtabmap_ros package to you src folder in your catkin_ws. You can find a link to the repo here and can install with cd ~/catkin_ws/src && git clone https://github.com/introlab/rtabmap_ros.

Note 4: Take this chance to understand what role each of the files plays and how they fit into the larger picture. 

### Mapping Best Practices

Our goal is to create a great map with the least amount of passes as possible. Getting 3 loop closures will be sufficient for mapping the entire environment. You can maximize your loop closures by going over similar paths two or three times. This allows for the maximization of feature detection, facilitating faster loop closures! When you are done mapping, be sure to copy or move your database before moving on to map a new environment. Remember, relaunching the mapping node deletes any database in place on launch start up!

### Building Your Own Simulated Environment

Once you have mastered the provided environment, it’s time to build upon your skills and use them for something more relevant to your robotics desires. Perhaps you want to test out some control algorithms for a drone you have built. One might argue that it is better to test in simulation first than to make costly mistakes in the real world. 

Let’s cover some basics that will help you create your own environment to map.

1. Open Gazebo
2. Go to Insert -> Model Database
3. Browse through the available models and find several objects that you would like to use-such as walls, tables, and pre-built environments. 
4. Drag your selected objects into the environment and situate them to your liking (you can change their position, orientation, scale.)
5. As you're build your environment, take into consideration that your robot will spawn at coordinates (0,0,0), so place the blue z-axis in Gazebo accordingly. 
6. When you have everything situated the way you like, export the model: File -> Save World As, then navigate to your custom package, and then the ‘models’ directory, and save the file with a .world extension.
7. When you are done, you may test your world by launching it as so, gazebo your_world.world

This is a relatively simple way to generate a world file. If you are interested in building custom models to generate a hyper-realistic environment there will be links to do so below. This would add a fantastic personal touch to your project, however, using existing models is sufficient for passing the project.

## Localization

If you desire to perform localization, there are only a few changes you need to make. You can start by duplicating your mapping.launch file and calling it localization.launch.

The following changes also need to be made to the localization.launch file:
1. Remove the args="--delete_db_on_start" from your node launcher since you will need your database to localize too.
2. Remove the Mem/NotLinkedNodesKept parameter
3. Lastly, add the Mem/IncrementalMemory parameter of type string and set it to false to finalize the changes needed to put the robot into localization mode. 

This is another method for localization you can keep in mind when working on your next robotics project!

## RTAB-Map Features

We have only touched on the main features of RTAB-Map, but let's take a few minutes to highlight some other fantastic features that one may like to incorporate into their robotics project! 


### Visual Odometry

In situations where odometry data may not be available (ex. there are no wheel encoders), you can use visual odometry instead to gather the same information with less resolution. This might be more applicable to a drone with just an RGB-D camera. 

### Obstacle Detection

This is a handy nodelet that extracts obstacles and the ground from your point cloud. With this information in hand, you can avoid these obstacles when executing a desired path. 

### WiFi Signal Strength Mapping

This feature allows you to visualize the strength of your robot’s WiFi connection. This can be important in order to determine where your robot may lose its signal, therefore dictating it to avoid certain areas. You could also try to find ways to remedy the situation with larger antennas. 


### Finding Objects both in 2D and 3D

Object detection is a great addition to your robotic stack if you are searching for an object and would like to find it and its position when surveying the area. 

### Multisession Mapping

Multisession Mapping is a great way to map specific areas and then come back to do more mapping beyond your first mapped area and have it be added onto you previous mapping session.


## Project Submission

It's now time to start taking all that you have learned and start using your created ROS package to create both a 2D and 3D map of the supplied environment. From there you will then build your own environment and use your ROS package to generate a 2D and 3D map of that environment. 

Let's lay out the project steps:
1. Download the supplied files to bring them into an environment with a terminal. 
2. Create a ROS package that is able to launch your robot and have it map the surrounding area. 
3. Use Debug tools and other checks mentioned in the lessons to verify that your robot is mapping correctly.
4. Create a 3D map for the supplied environment.
5. Create your own Gazebo world and generate a map for that environments as well. 
6. Follow the rubric to generate a proper report and determine which files need to be submitted. 

When you have completed the steps above and have cross-checked them to the rubric here, go ahead and submit your work! 

Be sure to include the following items in your project submission:
- A write up in PDF format that addresses the rubric points. Be sure to include sample images of your 2D Maps, 3D Maps, tf tree, etc. 
- A copy of your ROS package. 
- A copy of the RTAB-Map database for the supplied environment.





