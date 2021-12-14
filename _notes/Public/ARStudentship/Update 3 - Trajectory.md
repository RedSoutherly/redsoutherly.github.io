---
title : Update 3 - Trajectory
notetype : feed
date : 14-12-2021
tags : [Summer Research Project]
---

Following the milestones from the previous update, this post will briefly cover where the project is heading from here. 

The recent demo is the product of a completely manual process. The majority of this process centers on the modeling and texturing of the subject from a photograph. So, the key area of focus now is to automate as much of this process as we can. 
Ignoring time limitations this would culminate in a tool that could take a single photo of a building, and produce a 3D textured model of the visible facades. This would highly automate the creation of building assets for our AR use, but could also be used for any case where 3D reconstruction is required. Given the shorter time frame of this project, any point of significance during the processing would be an appropriate deliverable.

After discussing with Stefanie, we have decided that an appropriate method for this would be to modernise the approach used by Hoiem et al. in their [paper](https://dhoiem.cs.illinois.edu/publications/popup.pdf). They created a program in Matlab that uses statistical analysis of a photograph to identify a building's silhouette and "pop" it up into a 3D model of the visible faces.
In our approach, instead of using statistics we would use a convolutional neural network (CNN), to identify features in the photograph. Such as identifying the line where the building's foundation meets the ground, and where its walls meet at a corner. Using this information we can then lift and fold the building from the image into a 3D facade, similar to Hoiem et al.

Following this post I will be testing out existing tools to see different approaches. And to see if using a CNN will in fact be suitable for this. So there will be another update when I have made some ground on this and perhaps have some examples.
