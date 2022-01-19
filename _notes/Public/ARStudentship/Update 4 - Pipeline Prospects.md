---
title : Update 4 - Pipeline Prospects
notetype : unfeed
date : 19-01-2022
tags : [Summer Research Project]
---

Haven't written an update for a while so this will probably be a big one.


## Detectron 2
In the last update I said that the next destination of investigation was whether the use of CNNs would be suitable for our application. So since then I have been playing with and implementing the Detectron2 library created by the Facebook AI Research group, or FAIR. 

This library allows us to easily implement and train state-of-the-art detection and segmentation algorithms. Finding clearly explained tutorials for implementation has been a bit of a challenge not knowing much of the assumed knowledge, but I got there in the end. The main benefit of the library so far has been the ability to train for a custom dataset, on top of a pre-trained model that the library provides. This means that while one of my lecturers said a custom dataset should have ~100,000 images, I am getting surprisingly good results with just 26 images. Which is a saving grace since I have found that labelling data is a very lengthy process. 

Speaking of labelling, the detectron library uses the COCO format internally for its annotation data. So while the first tutorial I followed had me use the [labelme](https://github.com/wkentaro/labelme) tool, and then convert the JSON to COCO format using a python script. I thought it would be more efficient to have them in the correct format to begin with. So after a short search, I found [COCO Annotator](https://github.com/jsbroks/coco-annotator). This is data labelling tool that conveniently runs as a web application in a docker container. You can manage and export multiple datasets, and it has good drawing tools for segmentation.

#### Training with a custom dataset
My first go at training the network was with a custom dataset I called low_occlusion. The idea behind this approach was that you would give it a relatively clean image of a building with a low amount of objects occluding the view of the structure. I labelled it with five classifications; Ground, Sky, Background, Occlusion, and Subject, (the latter being the building). The idea with this was to do the whole classification process from the [Popup paper](https://dhoiem.cs.illinois.edu/publications/popup.pdf) in one go, but with a little bit on inpainting thrown in; I.e. find the silhouette of the building, strip away the sky portion of the image, and inpaint anything blocking the view of the building. The early results looked promising for the idea of using machine learning. 

Annotated Image | Segmentation after 1500 training iterations
:---:|:---:
![](/assets/img/studentship/lo_label_r.png) | ![](/assets/img/studentship/lo_1500_r.jpg) 

But I wasn't satisfied with some of the images being too busy. So I thought maybe I should break this more up into sections.

## Pipeline Concept
The thought of breaking up the process into stages brings us to the current plan of constructing a pipeline. The vision of the pipeline would to be to give it a single image of a historical building and get given back and 3D model of the buildings visible facades, along with a camera reference point.

The general process would be like this for each image inputted:
- All occluders in the image are identified, and a binary mask is generated.
- An inpainting algorithm takes the image and the mask, and removes the occluders.
- Using the "cleaned" image, just the building is identified and the rest is discarded.
- The image of the building is analysed for structural lines, i.e. where two walls meet.
- The image is then folded along these lines into a 3D "popup".
- The parallel lines on the model are used for a vanishing point calculation that identifies the location and perspective of the camera, and a marker is placed.

Currently I am up to implementing the inpainting portion of the pipeline. I will outline my process up to that point now.

#### Occlusion
This is the dataset that I am currently working forwards from. With this dataset I have a single class, occluder. The aim is that if I can identify all the objects in an image that are occluding the view, or are extraneous, then I could handle the inpainting first. This would then give me a "cleaner" image to further process; ideally containing just the building, ground, and sky. Since I have just one class I worried it was going to be too generalised for the CNN to handle. But I have been happily surprised by how well, in still very early stages, it can pick out objects in a scene that are not part of the building.

![](/assets/img/studentship/so_5000_example.jpg) 

#### Inpainting

