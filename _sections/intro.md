---
title: Introduction
order: 1
---

<!-- <iframe class="video" src="https://www.youtube.com/embed/7qmvjB_mxrE" frameborder="0" gesture="media" allow="encrypted-media" allowfullscreen></iframe>
*This video should be moved down, and we should just put a photo here. Maybe one of the ones Brent drew*-->

![josh eating](https://i.imgur.com/g5M2goS.jpg?1)

# Introduction

<!--
Describe the end goal of your project.
Why is this an interesting project? What interesting problems do you need to solve to make your solution work?
In what real-world robotics applications could the work from your project be useful?
-->

Our goal for this project was to use the Sawyer collaborative robot to assist a human with eating.

This is exciting to us not only because of our interest in the involved technical challenges, but also because these challenges are directly applicable to real world problems. In the assistive technologies space, we believe that robots have a lot of potential in giving individuals such as quadriplegics the autonomy to perform everyday activities such as eating on their own. In the future, we hope robotic systems that can grasp and manipulate items like food will play a large part in allowing everybody -- regardless of disability, background, or socioeconomic status -- to live independent and meaningful lives.

As a proof-of-concept, we simplified this challenge into the task of feeding somebody a marshmallow.
Several questions needed to be answered to make this happen:
- What sensors should we use to make the robot aware of its surroundings?
- How are these sensors mounted, positioned, and calibrated?
- How do we track the user’s mouth in 3D space?
- How do we detect and localize a marshmallow, again in 3D space?
- How do we reliably and repeatedly grip the marshmallow?
- How do we move the arm safely around the human?
