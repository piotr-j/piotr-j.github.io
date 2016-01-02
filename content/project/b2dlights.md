+++
author = "PiotrJ"
date = "2016-01-01T00:55:00+01:00"
description = "wip library"
tags = ["gamedev", "library", "box2d"]
title = "Smooth Box2d Lights"

+++

Extension to libgdx box2d lights that implements "smooth" lights. Default box2d lights are very "jumpy" due to them using simple raycasts spaced out at certain angle. 
Unless that angle is small, the gaps between individual lights cause the jumps. Smooth lights solve this problem by casting additional rays to vertices of objects in lights range.
This is less efficient then a light with same maximum ray count, but results in a lot better visual effect. 
This extension is used in [Project Sim](/projects/project-sim) and will be open sourced sometime after its release.