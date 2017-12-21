---
title: Implementation
order: 3
latex: yes
---

## Kinect Extrinsic Calibration

Before we can use the Kinect to accurately sense objects and people in the robot's environment, we first need to identify where the sensor is located relative to the robot.

This calibration procedure not only needs to be accurate, but it also must be fast enough to do each time we set up our demo.

The procedure is uses a [tweaked version of the ar_track_alvar](https://github.com/brentyi/ar_track_alvar) ROS package, which allows our Sawyer's gripper camera and our Kinect's depth camera to each track a shared set of randomly placed AR tags.

![calibration photo](https://i.imgur.com/8h9M3ah.jpg)

Then, we try to find the optimal **world => Kinect** transformation, which minimizes the squared positional error between the two broadcasted tfs associated with each tag. If we define a matrix X whose rows are the positions of the tags in the Kinect frame and a matrix Y whose rows are the positions of these tags in the world frame, we can solve for the position and orientation of the Kinect as follows:

$$
\begin{align*}
    X &= \begin{bmatrix}\vec{x}_1 & \vec{x}_2 & \dots & \vec{x}_n\end{bmatrix}^T\\
    Y &= \begin{bmatrix}\vec{y}_1 & \vec{y}_2 & \dots & \vec{y}_n\end{bmatrix}^T\\
    R, t &= \text{argmin}_{R\in\textit{SO}(3), t\in\mathbb{R}^3}\sum||(Rx_i + t) - y_i||^2
\end{align*}
$$

Solving this optimization by setting its derivative to 0 gives us a solution built off the singular vectors of the de-meaned position covariance:

$$
\begin{align*}
    \vec{\mu}_x &= \frac{1}{n}\sum \vec{x}_i\\
    \vec{\mu}_y &= \frac{1}{n}\sum \vec{y}_i\\
    \tilde{X} &= X - \begin{bmatrix}\vec{\mu}_x & \vec{\mu}_x & \dots & \vec{\mu}_x\end{bmatrix}^T\\
    \tilde{Y} &= Y - \begin{bmatrix}\vec{\mu}_y & \vec{\mu}_y & \dots & \vec{\mu}_y\end{bmatrix}^T\\
    \\
    U\Sigma V^T &= \tilde{X}^T\tilde{Y}\\
    R &= VU^T\\
    t &= \mu_y - R\mu_x
\end{align*}
$$

$$VU^T$$ is the ideal orthonormal transformation matrix with a magnitude 1 determinant, but if our input data is extraordinarily bad this can theoretically also be a reflection and not a pure rotation. This can be rectified by checking the sign of the determinant -- see [helpers.py](https://github.com/brentyi/marshmellow_localization/blob/master/scripts/helpers.py) for full implementation details.

## Marshmallow Pose Estimation

Once we have our depth camera calibrated, we can use it to search our robot's environment for marshmallows. For our project, we assume that these marshmallows are spread out on a table in front of it.

![marshmallow tfs](https://i.imgur.com/oOOaKOh.png?1)

Our marshmallow localization node parses the point cloud outputted by the Kinect, and applies a naive search algorithm to estimate the poses of marshmallows on it:

![marshmallow flow chart](https://i.imgur.com/iaKP4Jt.png)

## Face Tracking

We simplify mouth detection to just face detection with the position of the mouth estimated to be towards the bottom center of the face’s bounding rectangle. OpenCV provides a simple frontal face detection algorithm which uses Haar feature-based cascade classifiers to recognize potential faces in an image. The issue with just this is that the user’s face must be very “face-like,” looking straight ahead at the camera, and not tilted in any direction. This is unideal, so we add in another algorithm called template matching in order to track the face even when it’s not oriented properly. The way this works is that when OpenCV’s haar cascade algorithm is no longer able to detect a face, we switch to the template matching phase.

The previous face pattern is remembered, so that when we enter template matching, we search the video frame for that pattern, and the closest match is assumed the face. We return this result as the face. 

With the combination of these two algorithms, we’re able to track the subject’s face with much more precision and consistency. Once we know the 2D coordinates of the mouth in the RGB camera’s face, we can find the corresponding 3D coordinate in the Kinect point cloud, which is then broadcasted as a transform to the planning node!

This image shows Rviz on the left, where you can see the “face” transform indicating the 3D position of the mouth, and the face detected from the Kinect’s rgb camera drawing a box and circle around the user's face and mouth:

![mouth tracking](https://i.imgur.com/EnqBAi0.jpg)

## Path Planning

In order to perform path planning so that the gripper moves to an appropriate position relative to the marshmallow or mouth, we [broadcast TFs](https://github.com/williammlu/planning/blob/master/scripts/tf_broadcast.py) offset by a certain distance above the marshmallow or away from the mouth, all relative to the Sawyer base. These value were determined by measuring the gripper finger lengths and setting an offset that worked well when testing. The TFs that we broadcasted were `marshmallow_waypoint_goal` (10 cm above the marshmallow), `marshmallow_final_goal` (the appropriate TF to put the marshmallow between the grippers), and `face_gripper_goal` (a comfortable offset from the mouth TF).

In the ROS node that performs planning and movement, we listen for the aforementioned custom TFs to determine where the end effector should be moved to. We divide the path planning movement into three different phases with path planning constraints as mentioned:

- Moving to the `marshmallow_waypoint_goal` position and orientation in preparation for gripping. Restricted to 50% of maximum velocity and position tolerance of 1cm.
- Moving to the `marshmallow_final_goal` position and orientation to perform gripping. This movement happens directly after moving to the waypoint, so the movement is vertically downward. Restricted to 10% of the maximum velocity and position tolerance of 5mm.
- Moving to the `face_gripper_goal` position and orientation. Restricted to 50% of the maximum velocity and position tolerance of 1cm.
- While path planning, we restrict our path planner’s scene with custom constraints. We created 0.1 by 4 by 4 meter blocks to take coincide with the table, back and right walls, and a fake wall aligned in the same plane as the user’s face. We fixed the wall constraints relative to the base of the robot by manually measuring each obstacle from the base of the robot. This was done because the walls were not easily measurable by our robot’s sensors or Kinect. WIth these walls, the path planner would ideally avoid any collisions with the walls, table, or user.

These actions were abstracted on our web user interface into a “marshmallow” button to move to the waypoint then to the marshmallow gripping position, and a “mouth” button to move to the mouth position.

## Gripping

Marshmallows are particularly challenging to grip because they are soft, deformable objects. Additionally, the Sawyer’s gripper do not always detect enough force applied when a marshmallow is present. In order to address these issues, we applied these following techniques to maximize the probability of gripping:

- Two phase movement sequence to gripping position: to ensure that we move the gripper to an accurate position with reasonable planning speed, we divide the movement to the marshmallow into two parts. One is a coarse movement to a waypoint directly above the marshmallow. The second movement is a high precision movement directly downward from the waypoint with low velocity and 10 planning attempts. Separating the coarse and fine movement allows for a consistently high accuracy movement without making the user wait for too long.
- Multiple gripping attempts: to tolerate the greatest gripper position error, we installed Sawyer’s grippers at just under the widest setting. Due of the limited travel of the gripper, closing the gripper and asking if it is gripping has inconsistent results when when fully shut. In order to combat this, we allow the gripper to make 3 attempts. From our experience, a failing first gripping attempt usually aligns the marshmallow appropriately for a second gripping attempt. Due to the problem of not being able to consistently identify a failed gripping attempt, we decided to assume that the final gripping attempt was successful for the sake of testing, but we have ideas about how to address this in the future using the head camera or Kinect as mentioned.
- Using a wide gripper: to maximize the possibility of gripping the marshmallow, we chose to use an extra wide gripper to to allow for the maximum error in the direction perpendicular to the travel. Additionally, we put tape over the gripper pads to avoid marshmallow residue from getting on the gripper fingers and to minimize friction between the marshmallow and the gripper when the user is fed.

![gripper](https://i.imgur.com/7p8PIYj.jpg)

The gripping action is abstracted into the “grip” button on the web interface.


## Control Interface

![web ui](https://i.imgur.com/3l6DA2x.png)

To send commands to our robot, we connected it to a web service in the cloud.

A ROS node communicates with the server, and makes a corresponding ROS service call each time a button on our web interface is pressed.

In the future, we could use this connection to the cloud to allow the robot to be commanded through a voice assistant, such as Amazon Alexa.

## Running our project

At the start of our project, we discussed a lot about making a single launch file for the entire Mr. Marshmello stack: hardware drivers, MoveIt, calibration code, face tracking, etc. However, we soon realized that this would make development and debugging significantly more difficult. We wouldn't be able to, for example, kill and restart just a single one of our nodes without restarting the entire stack.

Instead, we split our project up into several different logically grouped launch files. A [shell script](https://github.com/brentyi/marshmello_bringup/blob/master/run.sh) was then written to automatically launch each of them in named [tmux](https://github.com/tmux/tmux/wiki) panes. This was easy to run, yet also easy to debug:

```
tmux new-session -d -s marshmello

tmux rename-window 'sawyer'
tmux send-keys 'roslaunch marshmello_bringup sawyer_moveit.launch'

tmux new-window -t marshmello -n 'rviz'
tmux send-keys 'roslaunch marshmello_bringup sawyer_rviz.launch'

tmux new-window -t marshmello -n 'camera_drivers'
tmux send-keys 'roslaunch marshmello_bringup camera_drivers.launch'

...
```
