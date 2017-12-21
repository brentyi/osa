---
title: Design
order: 2
---

We split our task of feeding an individual a marshmallow into a handful of distinct steps:


1. Receive a “feed” command from the user
2. Detect and localize a single marshmallow that’s within our robot arm’s workspace
3. Detect and find the position of the human’s mouth
4. Move the end effector to the position of the chosen marshmallow
5. Grip the marshmallow
6. Move the marshmallow to the user’s mouth


To complete all of this successfully, our assistive feeder had to be efficient, accurate, and interactively simple. We wanted to reliably detect and pick up marshmallows, consistently bring them to within inches of the user’s mouth position, and implement a straightforward control interface.

We use our personal Kinect2 over a provided Kinect1 because of the higher resolution the Kinect2 provided, which was integral to our desired accuracy. The Sawyer arms are also more precise than the Baxter arms. We use Sawyer’s wrist cam instead of its head cam to help calibrate our system because of the head cam had skewed perspective issues. 

We also decided to eliminate the use of AR tags during the actual feeding operation, because they’d be unrealistic to use in the real world. To successfully accomplish this, we had to use some sort of RGBD camera, like the Kinect2, which supplied both a point cloud and color image in order for us to properly detect the marshmallows and face. 

Since our Kinect was our main means of sensing, we had to figure out a way to transform the Kinect’s feed to be relative to the frame of the robot. We did this so that Sawyer could know where everything was in relation to its base frame. We spent some time thinking of the best orientation for our human subject, the placement of the Kinect, and the placement of Sawyer.
Another design choice we faced was related to how we wanted to interface with our system. The easiest approach would be for our system to purely take in command-line commands, but this wasn’t clean enough. So we looked into using some sort of web interface, or even possibly a voice-controlled interface. The web interface was the most robust and effective, and was the right choice. 

These are definitely decisions that would be made in industry applications. The type of hardware to use, the ideal placement of hardware, the interaction between human and robot, are all choices to be made in real engineering projects. Our project’s design resulted in a robust and efficient system.

