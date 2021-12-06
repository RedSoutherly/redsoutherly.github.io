---
title : Week Two - Milestone Reached
notetype : feed
date : 06-12-2021
tags : [Summer Research Project]
---

I wanted to originally call this post "trials & tribulations" for the nice alliteration with last week's update. But it was a more successful week than that title would suggest. Which I suppose is a good thing.
What I'll get to at the end of this update is that I have made a rough working demo of the projects idea, and that it works quite well in practice. But to be consistent I will first cover how I got there.

The first thing I needed to do was conclude whether or not I could achieve projective texture mapping within Lens Studio. Long story short i could not find anything that allowed this technique. There were two avenues that looked closest to our solution but were ultimately not suitable. [Landmark Tracking](https://lensstudio.snapchat.com/templates/landmarker/guide/) can be used to identify and apply effects to a landmark structure. But this is only available for provided landmarks with no way to specify a custom one. And there were some mentions of "projecting" a texture onto a [face mesh](https://lensstudio.snapchat.com/guides/face/face-effects/face-mesh/), but this is then limited to it being on a face mesh.

So we are left with the other two options; baking textures, and switching the platform to Unity. Ultimately we want to continue development using Lens Studio, not only does this simplify deployment to mobile, but also allows us to deploy to their AR glasses as the project intends to do. So baking textures it is.

### Baking
While researching how to bake the textures from the virtual projector in Blender, I discovered I didn't need to spend the time making the projector at all. Since I could bake the texture from the camera overlay directly, instead of first turning it into a light source. So a fair amount of time wasted then, but no bother. 
Lining up the camera's overlay image was quite straight forward thanks to fSpy, which I talked about last week. Importing the fSpy project into Blender gave us *almost* a perfect fit. But I did need to slightly distort the scale of the model to make it fit perfectly.

![](/assets/img/studentship/bakeoverlay.png)
*Featuring our new clocktower model, courtesy of Animation Research Limited (ARL).*

Baking the texture onto the model required the model to be properly UV unwrapped, and to setup the shader graph for baking. Our clocktower model is quite complex, so unwrapping it properly required some tricks. Thankfully I found a quick way to achieve this using this creators [tutorial](https://www.youtube.com/watch?v=icsECBsOxk4). The actually baking process involved turning the camera's overlay into an emission, and then baking that to a image compatible with our UV map. Another creators detailed [tutorial](https://www.youtube.com/watch?v=8NYNiayHvJI) helped me achieve that.

![](/assets/img/studentship/firstbaked.gif)

Seen above, the first attempt at baking a photograph as a texture for this project yielded great results. Aside from the obvious repeated texture on the right, where the model is not represented in the photograph, the correctly aligned section has good resolution and allows for a significant amount of parallax. Going forward I plan to avoid the texture being repeated by simply removing the sections of the model not visible in the photograph, making a textured facade instead of an entire model. But for the time being this works as a proof of concept.

### Demos
Now that I had a textured model, I could import it into Lens Studio. [[Note that it needs to export from Blender with PBR textures for this to work.::rmn]] Once in Lens Studio I just had to align it to the previous model and then I could set about creating a collection of test cases. We were testing for object stability with different camera ranges, markers, with and without extended tracking. We found that in real use the ranges and markers didn't make much noticeable difference, but extended tracking did. Normally a marker tracking lens will base all of its alignment information off of just the perceived marker. This means that you are limiting the world data you have to just the marker, and whether or not you can see it at all. We found that using this method the objects in the scene were very unstable, especially distant objects like our building model.
Extended tracking on the other hand uses more than just the marker in its view of the world. Once the marker is detected the first time, all the objects are placed relative to it. Once the objects are placed, the marker no longer matters. Instead the lens will use whatever data it can access, camera, LiDAR, etc, and track the entire space. This gives the lens many more points of reference to calculate the position and movement of the device in space, meaning the alignment is more accurate. And we found that this greatly increased the stability of the objects.

#### Mobile Demos

Demo without extended tracking | Demo with extended tracking
:---:|:---:
<video muted controls><source src="https://redsoutherly.github.io/assets/img/studentship/nonextendeddemo.mp4" type="video/mp4"></video> | <video muted controls><source src="https://redsoutherly.github.io/assets/img/studentship/extendeddemo.mp4" type="video/mp4"></video>

Above you can see the two demos running on a mobile phone, and the significant advantage that extended tracking provides. But we also got the demo to work on the AR glasses, see below.

#### Spectacle Demos

Demo without extended tracking | Demo with extended tracking
:---:|:---:
<video muted controls><source src="https://redsoutherly.github.io/assets/img/studentship/nonextendeddemo-spec.MP4" type="video/mp4"></video> | <video muted controls><source src="https://redsoutherly.github.io/assets/img/studentship/extendeddemo-spec.MP4" type="video/mp4"></video>

The instability when not using extended tracking is even more apparent in the demo from the spectacles. Where moving parallel to the marker sent the object completely off screen. So going forward use of extended tracking is going to be a given.

### Outlook
Now that we have a working demo, manual refinement would be trivial. Touching up the model and texture, working on alignment; this would not require further exploration. So what I must instead now do is work on the pipeline. This demo was created all manually, looking into solutions for automating as much of the process as possible is where the research will directed.

I have already found examples of papers generating 3D models from single 2D photographs. Many using convolutional neural networks (CNNs) to define architectural features. One [paper](https://www.researchgate.net/publication/325489913_Procedural_Modeling_of_a_Building_from_a_Single_Image), from Nishida et al, stood out to me. They created a procedural generator that could produce many different types of buildings, and then used a CNN on a single image to calculate a grammar for the generator. This created consistent uniform models for many of the input images. I wonder if it would be possible to use a similar method, but with a procedural generator designed to produce classical and Gothic architecture. So more historical buildings could also be reproduced from a single image using inverse procedural generation. Although I won't lock myself into that direction just yet, as there are many other avenues to check.

Time to do some research then. See you next week, probably...






