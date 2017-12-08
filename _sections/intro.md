---
title: Introduction
order: 1
---

<iframe class="video" src="https://www.youtube.com/embed/7qmvjB_mxrE" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>
*This video should be moved down, and we should just put a photo here. Maybe one of the ones Brent drew*

# Introduction

## Describe the end goal of your project.
## Why is this an interesting project? What interesting problems do you need to solve to make your solution work?
## In what real-world robotics applications could the work from your project be useful?

Our goal for this project was to autonomously feed a human a marshmallow. 

There would be a few phases in order to accomplish this:
1. Detect a single marshmallow from a group of marshmallows
2. Detect and find the position of the human's mouth
3. Move the end effector to the position of the chosen marshmallow
4. Grip the marshmallow
5. Move the end effector with the gripped marshmallow to the user's mouth
6. Repeat as needed

This task is interesting both because it is directly applicable to many real world scenarios and because there are a few challenging problems to solve here as well.
This concept is aligned in the assistive technologies space, dealing with helping people who have trouble reaching out and feeding themselves, such as those with under developed motor skills or paraplegics for example.
In the future, fully automated systems which can detect items to grasp, and bring them to ideal destinations will revolutionize the standards of living for all types of people.

For this specific project, there were a few challenging problems we had to deal with.
First off, the method to calibrate the Kinect is actually a much more complicated mathematical problem than we expected.
Then, identifying marshmallows itself is not a very straight forward task, especially because we aimed to be able to do so without AR tags or pre-defined placements.
From there, the subtleties of face detection and gripping of a soft deformable material bring up their own series of challenges as well.
All these points are elaborated in the following detailed sections.
