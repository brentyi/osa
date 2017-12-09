---
title: Implementation
order: 3
latex: yes
---

# Implementation

<!--
Describe any hardware you used or built. Illustrate with pictures and diagrams.
What parts did you use to build your solution?
Describe any software you wrote in detail. Illustrate with diagrams, flow charts, and/or other
appropriate visuals. This includes launch files, URDFs, etc.
How does your complete system work? Describe each step.
-->

## Kinect Extrinsic Calibration

Using a [tweaked version of the ar_track_alvar](https://github.com/brentyi/ar_track_alvar) ROS package, we set up our Sawyer's gripper camera and our Kinect's depth camera to each track a shared set of randomly placed AR tags.

![calibration photo](https://i.imgur.com/8h9M3ah.jpg)

Then, we try to find the optimal **world => Kinect** transformation, which minimizes the squared positional error between the two broadcasted tfs associated with each tag. If we define a matrix X whose rows are the positions of the tags in the Kinect frame and a matrix Y whose rows are the positions of these tags in the world frame, we can solve for the position and orientation of the Kinect as follows:

$$
\begin{align*}
    X &= \begin{bmatrix}\vec{x}_1 & \vec{x}_2 & \dots & \vec{x}_n\end{bmatrix}^T\\
    Y &= \begin{bmatrix}\vec{y}_1 & \vec{y}_2 & \dots & \vec{y}_n\end{bmatrix}^T\\
    R, t &= \text{argmin}_{R\in\textit{SO}(3), t\in\mathbb{R}^3}\sum||(Rx_i + t) - y_i||^2
\end{align*}
$$

Solving this optimization by setting its derivative to 0 eventually gives us the solution:

$$
\begin{align*}
    U, \Sigma, V &= \text{SVD}((X - \mu_x)^T(Y - \mu_y))\\
    R &= VU^T\\
    t &= \mu_y - R\mu_x
\end{align*}
$$

$$VU^T$$ gives us the ideal orthonormal transformation matrix with a magnitude 1 determinant, but if our input data is extraordinarily bad this can theoretically also be a reflection and not a pure rotation. This can be rectified by checking the sign of the determinant -- see [code](https://github.com/brentyi/marshmellow_localization/blob/master/scripts/helpers.py) for the full implementation details.

## Running our project

At the start of our project, we discussed a lot about making a single launch file that the entire Mr. Marshmello stack -- hardware drivers, MoveIt, calibration code, face tracking, etc. However, we soon realized that this would make development and debugging significantly more difficult. We wouldn't be able to, for example, kill and restart just a single one of our nodes without restarting the entire stack.

Instead, we split our project up into several different logically grouped launch files. A [shell script](https://github.com/brentyi/marshmello_bringup/blob/master/run.sh) was then written to automatically launch each of them in named tmux panes. This was easy to run, yet also easy to debug:

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
