---
title : Update 4 - Pipeline Prospects
notetype : feed
date : 20-01-2022
tags : [Summer Research Project]
---

Haven't written an update for a while so this will probably be a big one.


## Detectron 2
In the last update I said that the next destination of investigation was whether the use of CNNs would be suitable for our application. So since then I have been playing with and implementing the [Detectron2](https://github.com/facebookresearch/detectron2) library created by the Facebook AI Research group, or FAIR. 

This library allows us to easily implement and train state-of-the-art detection and segmentation algorithms. Although, finding clearly explained tutorials for implementation has been a bit of a challenge, not knowing much of the assumed knowledge, but I got there in the end. The main benefit of the library so far has been the ability to train for a custom dataset, on top of a pre-trained model that the library provides. This means that while one of my lecturers said a custom dataset should have ~100,000 images, I am getting surprisingly good results with just 26 images. Which is a saving grace since I have found that labelling data is a very lengthy process. 

Speaking of labelling, the detectron library uses the COCO format internally for its annotation data. So while the first tutorial I followed had me use the [labelme](https://github.com/wkentaro/labelme) tool, and then convert the JSON to COCO format using a python script. I thought it would be more efficient to have them in the correct format to begin with. So after a short search, I found [COCO Annotator](https://github.com/jsbroks/coco-annotator). This is a data labelling tool that conveniently runs as a web application in a docker container. You can manage and export multiple datasets, and it has good drawing tools for classification.

#### Training with a custom dataset
My first go at training the network was with a custom dataset I called low_occlusion. The idea behind this approach was that you would give it a relatively clean image of a building with a low amount of objects occluding the view of the structure. I labelled it with five classifications; Ground, Sky, Background, Occlusion, and Subject, (the latter being the building). The idea with this was to do the whole classification process from the [Popup paper](https://dhoiem.cs.illinois.edu/publications/popup.pdf) in one go, but with a little bit on inpainting thrown in; I.e. find the silhouette of the building, strip away the sky portion of the image, and inpaint anything blocking the view of the building. The early results looked promising for the idea of using machine learning. 

Annotated Image | Segmentation after 1500 training iterations
:---:|:---:
![](/assets/img/studentship/lo_label_r.png) | ![](/assets/img/studentship/lo_1500_r.jpg) 

But I wasn't satisfied with some of the images being too busy. So I thought maybe I should break this more up into sections.

## Pipeline Concept
The thought of breaking up the process into stages brings us to the current plan of constructing a pipeline. The vision of the pipeline would to be to give it a single image of a historical building and get given back a 3D model of the building's visible facades, along with a camera reference point.

The general process would be like this for each image inputted:
- All occluders in the image are identified, and a binary mask is generated.
- An inpainting algorithm takes the image and the mask, and removes the occluders.
- Using the "cleaned" image, just the building is identified and the rest is discarded.
- The image of the building is analysed for structural lines, i.e. where two walls meet.
- The image is then folded along these lines into a 3D "popup".
- The parallel lines on the model are used for a vanishing point calculation that identifies the location and perspective of the camera, and a marker is placed.

Stepping back a bit. Our current methodology for the project is to produce more of  concept than a final product, due to the time limitations. So reaching "good enough" at a stage will prompt the move to the next one for now.

With that in mind, I am now up to implementing the inpainting portion of the pipeline. I will outline my process up to that point now.

#### Occlusion
The first process that I needed to implement was identifying the "occluding" objects in an image. So I got my set of training images and started drawing polygons around everything from trees to dogs. It feels like training with a negative assertion but I want to identify anything that isn't ground, sky, or building. Although, as you can see below, it worked pretty well for early days. 5000 iterations on a dataset containing 26 images, taking only about 40 minutes on the departments server, produced visually quite good results.

![](/assets/img/studentship/so_5000_example.jpg) 

I will see if I find the time to improve the size of the dataset to make this model better, but for now "good enough".

#### Inpainting

Carrying on from identifying the occluders, they now need to be removed. Inpainting usually requires the original image, and a binary mask of what needs to be removed. So I spent a morning making a new detectron script which instead of outputting a colourfully labelled picture, outputs a binary mask of the occluders. Although I am currently between whether to use a gaussian blur on the mask or not. This will depend on which looks best after inpainting.

Gaussian | No Gaussian
:---: | :---:
![](/assets/img/studentship/mask_blur.jpg) | ![](/assets/img/studentship/mask.jpg)  

Once I had the masks, there were a couple avenues to explore.
OpenCV has a couple built-in inpainting algorithms, which is convenient because OpenCV is already being used for loading images into detectron. So that was clearly the first option to look into, however the results weren't that good. The algorithms that OpenCV provided work by replicating the pixels from around the edge of the mask and "stretching" them over the void in the mask. This may work for small voids in the mask or thin lines, but for large sections of image it looks more like a smudge.

![](/assets/img/studentship/cv2_inpaint.jpg)

What I needed was an inpainting algorithm that could handle proper object removal. So after a bit of searching around, I found [this promising looking implementation by zavolokas](https://github.com/zavolokas/Inpainting) on GitHub. It is a .NET implementation of a content-aware fill algorithm which can be quite effective at removing objects from images. It took a bit of tinkering to figure out how to use the implementation since I had never used .NET before. But it is looking good from the first test.

![](/assets/img/studentship/result.png)

The key area for me is where the fence posts have been inpainted. The filling of the space using the weather boards from the building behind is exactly what I want to have happen. On the other hand it has replaced the trees with replications of the building, which I would prefer to be blank patches of sky. I am going to explore using this implementation further to see if I can get it to achieve more satisfactory results. 

I am aware that the implementation has a mechanism to specify a "donor" area for the inpainting, but this might require either manual intervention in the pipeline, or adding a processing step to the occlusion identification. Although, to add a processing step I could perhaps make a temporary building mask, and any part of the occluder mask within that gets the building as a donor. And any part of the occluder mask that doesn't overlap with the building can get the sky and ground as a donor area. But this might be too much extra to implement into the pipeline at this stage.

## Overview

So, to wrap up this big update then.

The general outlook is that we are planning on creating a pipeline for 3D content creation. Namely the creation of a 3D building facade from a single photograph. We will be developing this theoretically in multiple passes. This means creating a rough but full pipeline first, i.e. a rough proof-of-concept. This can then be iterated back over and each stage polished to increase the fidelity of the final artifact. So far on the first pass I have implemented the instance detection of occluders in an image, and I have started implementing the inpainting of the occluders.

Here is a small overview of the image processing:
Identify Occluders | Mask Occluders | Inpaint Occluders
- | - | -
![](/assets/img/studentship/so_5000_example.jpg) | ![](/assets/img/studentship/mask.jpg) | ![](/assets/img/studentship/result.png)

I will try make the next update cover a shorter period of development as this one has, likely when I am happy with the inpainting implementation. See ya then.