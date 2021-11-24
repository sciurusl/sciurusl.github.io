---
layout: post
title:  OpenGL village
date:   2017-10-27 10:11:50 +0300
img: PGR/fire.jpg
tags: [OpenGL, C++]
---
For a student project, I created a small village scene in OpenGL and C++, where the player could
somehow interact with the environment. 

The goal of this project was to create a scene with a village, where the player can move using WASD, 
cut trees by clicking on them, collect wood and light a fire by clicking on one of the rocks.
The player can turn on a flashlight, a fog, change camera and open a menu. The time changes, which is visualized by 
darkening and dawning of the sky.

![village1]({{site.baseurl}}/images/pages/PGR/screenshot1.jpg)

![fog]({{site.baseurl}}/images/pages/PGR/screenshot2.jpg)

An example of some of the shaders: 
This is part of the vertex shader which computes color according using texture and light in the current vertex

![vertex shader]({{site.baseurl}}/images/pages/PGR/openglvert.jpg)

This is part of the fragment shader which computes color in the current fragment

![fragment shader]({{site.baseurl}}/images/pages/PGR/openglfrag.jpg)