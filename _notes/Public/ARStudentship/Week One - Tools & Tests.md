---
title : Week One - Tools & Tests
notetype : feed
date : 25-11-2021
tags : [Summer Research Project]
---

My first week in the lab involved familiarising myself with the tools we plan on using and also finding new ones. While starting to tackle the fundamental "how?"s of the project.

Currently, the plan is to use Snapchat's [Lens Studio](https://lensstudio.snapchat.com/) to implement the AR functionality of the project. This IDE gives a wide range of tools to create your own AR "lenses" to use in Snapchat very easily. The main basis we are using for this project is marker tracking, this allows the lens to recognize some kind of 2D marker in the camera view and then place assets relative to it.

Once I had familiarised myself with the software I needed to apply it in the real world. I have chosen to focus on the university's clocktower for this since we have many photographs and it is very prominent. Then I needed to find a suitable marker, which was rather easy given there was a sign with a view of the building directly behind it (all be it with trees in the way). The sign is not entirely unique across campus but is the only one with the relevant positioning.

![](/assets/img/studentship/ClocktowerMarker.jpg)

### Modeling 
The next task was to figure out how exactly to align a historical photograph with its real counterpart live. Originally we were just going to try to line up the picture as a 2D plane, so from an exact position by the sign, it would overlay on the building. This felt a bit too "clunky", it wouldn't look very nice in the 3D space, and if the user moved from the position it wouldn't line up anymore. So the new idea is to place a 3D model of the building relative to the marker so that it sat "on top of" the real one. Since we could place this at the correct relative distance from the marker, the user moving around wouldn't cause a parallax between the real building and its model. Then we can "texture" the 3D model with the photograph, allowing the user to move without losing alignment of features.

So now we need a 3D model. I looked at a few options for modeling software but settled on [Fusion360](https://www.autodesk.co.nz/products/fusion-360/overview) by Autodesk, who thankfully offers a free personal license. Using [OpenStreetMap](https://www.openstreetmap.org/) I could find a rough floor plan of the clocktower to sketch, and then extrude up into the general shape of the building.

![](/assets/img/studentship/F360ClockTower.png)

This gave me a rough, but scale, model of the clocktower building. When modeling, I also marked the location of the sign on the map. This was so when I imported the model into lens studio I could immediately place the building at roughly the correct distance from the marker. Although my GPS location of the sign was not quite correct, so I had to move the model manually with a preview image to align it correctly with the real building. But just a little.

![](/assets/img/studentship/F360LensPreview.png)

### Texturing
At this point, I had a lens that could turn the clocktower building into a grey box. This was good progress but the issue now became how to get a historical photograph onto a 3D model of a building.   

The photos we have are not orthographic, which they likely never will be. So we can't just apply them as textures to the surfaces of the model, since the perspective of the photo will not line up. Perspective is the limiting factor, we can only align the features of the building with the model if we do so from the exact perspective of the camera from many years ago. This also means only the surfaces of the model that are visible in the photo can be textured, but that is a problem that is simply not solvable in this instance.   

But assuming we could exactly line up the photo (back to this later), how do we then place the textures on the surfaces of the model. Well, the photo was taken from a camera lens seeing light from a specific perspective; so how about we reverse this process? Instead, we can project the photo onto the model from a given perspective position.   

After spending a fair amount of time looking at how to do this virtually I ended up at [Blender](https://www.blender.org/). This is a popular software used for 3D modeling, animation, and art, alongside I'm sure many other uses. But the key point is that I could use it to create a virtual "projector". Specifically, I could set up a spotlight in the scene that used an image for its emission. And then the Blender software can map the light onto our 3D model of the building. Concisely this is called projective texture mapping.

![](/assets/img/studentship/F360BlenderProjection.png)

Now we're caught up to current tinkering. The problems I am working on now are; how to align the projector perfectly, and how to turn the mapped texture into a static texture.

As far as lining the projector up, I am currently testing out [fSpy](https://fspy.io/). This is a piece of software that can be used in conjunction with Blender. Its exact purpose is to locate the position of a camera in 3D space, based on a single photograph. This is done manually by lining up the 3D axis on the axis visible in the photograph. And it can then export this to Blender where it will create a camera relative to the origin point you specify. So this looks like it will be the tool used in the long run.

![](/assets/img/studentship/fSpyExample.png)

The final issue to be mentioned here is turning the mapped texture into a static texture, or not at all. My original plan was to have the 3D model in lens studio with pre-applied textures on the surfaces. This would keep it lightweight in terms of processing. This meant I needed to somehow "capture" the mapped texture and then apply it directly to the model's surface. One way I could possibly do this is to take orthographic renderings of the model in Blender, and then use these as 2D textures. Which I think of as turning the photograph into pseudo-orthographic views of the building.

At this point, Stefanie suggested not bothering with static textures and instead investigating if lens studio could do the texture mapping live in the lens itself. However, after some research, it doesn't look like lens studio has this capability. Although I need to confirm this further still. Another option is switching the AR platform. [Unity](https://unity.com/) is another software capable of making AR applications. And it also has more advanced features, including projective texture mapping. 

So going forward from here I will investigate whether to continue with pseudo-orthographic textures or if having live texture mapping will prompt a shift from Lens studio to Unity.