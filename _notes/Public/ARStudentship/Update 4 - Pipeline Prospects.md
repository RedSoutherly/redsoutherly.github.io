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

Stepping back a bit. Our current methodology for the project is to produce more of  concept than a final product, due to the time limitations. So reaching "good enough" at a stage will prompt the move to the next one for now.

With that in mind, I am now up to implementing the inpainting portion of the pipeline. I will outline my process up to that point now.

#### Occlusion
The first process that I needed to implement was identifying and masking the "occluding" objects in an image. So I got my set of training images and started drawing polygons around everything from trees to dogs. It feels like training with a negative assertion but I want to identify anything that isn't ground, sky, or building. Although, as you can see below, it worked pretty well for early days. 5000 iterations on a dataset containing 26 images, taking only about 40 minutes on the departments server, produced visually quite good results.

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

