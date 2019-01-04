---
title: "Computer-aided system to control a moving of a robotic system in an environment with obstacles"
excerpt: "The work was been made for the University in masters thesis by me"
header:
  image: assets/images/robot-motion/robot-motion-main-min.png
  teaser: assets/images/robot-motion/robot-motion-main-min.png
sidebar:
  - title: "Place"
    image: assets/images/robot-motion/pnrpu-logo-min.png
    image_alt: "logo"
    text: "Perm National Research Polytechnic University (PNRPU), Perm"
  - title: "Role"
    text: "Student"
  - title: "Achievements"
    text: "Made a scientific work include robot motion models for a department in University"
gallery:
  - url: assets/images/robot-motion/robot-motion-1-min.png
    image_path: assets/images/robot-motion/robot-motion-1-min.png
    alt: "A manipulator robot trying to bypass obstacles"
  - url: assets/images/robot-motion/robot-motion-2-min.jpg
    image_path: assets/images/robot-motion/robot-motion-2-min.jpg
    alt: "A prototype of a real frame robot"
  - url: assets/images/robot-motion/robot-motion-3-min.png
    image_path: assets/images/robot-motion/robot-motion-3-min.png
    alt: "A model of a frame robot with visible joints"
---

In my last course of the University, I've decided to help my department to create a model that will simulate the real system for scanning details in many angles.
The main purpose of the work is to find an algorithm to move robots by path without collision to each other and the environment.
I use [V-REP][vrep] system to design models and to solve inverse and forward kinematics tasks of together motion two robots.
In the picture gallery is present models are consisting of two main robots is a manipulator (in the picture it is a "1") and a frame robot ("2").
Also has a table robot ("3") that rotates a detail ("4").

Maybe it is not a work for IT or programmer but I was exploring many science articles and approaches,
studying it to make a good thesis of a scientific work and presents it to a commission of the University.
And also I've learned the basics in Lua language.

I've uploaded the source code of the project into [Github](https://github.com/RuslanGainanov/robot-motion).
Also you can view the video of a scanning process of a detail.
<iframe width="640" height="360" src="https://www.youtube-nocookie.com/embed/yxw9i_CwjF0" frameborder="0" allowfullscreen></iframe>

{% include gallery %}

[vrep]: http://www.v-rep.eu/
