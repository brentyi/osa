---
title: Conclusion
order: 5
---

![josh eating](https://i.imgur.com/g5M2goS.jpg?1)

<!-- ## Discuss your results. How well did your finished solution meet your design criteria? -->
The final outcome of our system worked extremely well, meeting all of our initial design criteria.

<!-- ## Did you encounter any particular difficulties? -->
## Difficulties
The majority of our difficulties were related to path planning. We had many problems while attempting to interact with the robot. The first problem involved the MoveIt Python library. We faced a great amount of confusion trying to figure out how to use this library since it’s not as well documented as its C++ counterpart. For instance we struggled to figure out end effector positions, relevant reference frames, and gripper controls. Most of these problems were solved through trial and error. Furthermore, MoveIt would crash constantly and was difficult to debug. We also had issues with path planning constraints. We tried to implement as many realistic constraints as we could. However, even if the constraints seemed flexible and realistic, the MoveIt path planning library would not be able to generate a valid plan. We ended up removing around half our constraints to get a working result. Additionally, we had difficulty with gripping controls. We originally had many ideas for precise gripping, such as setting the force and velocity of the gripper. However none of the gripper controls documented online would work, and our code would end up crashing. In the end we were limited to only opening and closing our gripper. We also did not realize that in order to use the gripper, we would have to initialize it, which involved not having an object between the grippers. In our first iterations, we would only initialize the gripper when there was a marshmallow between the grippers, leading to an uninitialized gripper and ultimately having our code crash. Through some tinkering we realized our mistake and fixed our code.

Outside of path planning, we had issues calibrating our kinect. If the kinect was calibrated incorrectly, the position of the marshmallows would be off, guiding Sawyer to the wrong position. To mitigate this problem, we set a maximum error limit while calibrating the kinect to minimize offset positions. We also had problems with getting the correct kinect drivers on the lab computers. It was only during the last couple of days that the correct drivers were installed on the lab machines and we could run our code. We were testing from our personal computers before that.

<!-- ## Does your solution have any flaws or hacks? What improvements would you make if you had additional time? -->
## Possible Improvements

Sawyer’s default path planning can be quite hectic, and due to its placement in the lab, Sawyer’s arm path would often collide with the wall behind or next to it. So we set up constraints, preventing it from hitting those walls, or the table we placed our marshmallows on. But then when we placed a vertical constraint at the user’s face position, we ran into the issue where the arm wouldn’t even be able to find a path to the marshmallow, or to the face - even though we could manually move the arm correctly in gravity mode. Our solution to this was to remove the face constraint, and to just make sure the subject was aware of where the arm was moving. This would definitely be unacceptable in a real scenario, but because the arm doesn’t move at high speeds, and our subjects were all capable of extrapolating the path of the arm as it moved, this was not a safety concern.
Another problem that we encountered was detecting if the gripper has successfully gripped the marshmallow. Since the necessary gripper position is fairly wide, it can’t always close to a point where the gripper can detect the spring force from the compressed marshmallow. We currently have no way to detect a failed grip attempt with certainty, but given more time, it is possible that for an apparently failed gripping attempt, we move the arm to a default position in front of the head camera to detect if a marshmallow is visible. With this secondary check, we can determine with certainty if the gripping sequence was successful or if it needs to be redone.

