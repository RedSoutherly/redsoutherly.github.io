---
title : Building extraction
notetype : feed
date : 25-04-2022
tags : [Honours]
---

Following the creation of the conductor script, which I talked about in the post before this one, the next goal for the project was to take the inpainted images and extract just the building from the image, making the background transparent. This is so only the building's texture is being used in the future folding process.

Overall this task required two things. A new dataset to train a detectron model with, and a script to use the model to cut the building's out of the photos.

### Building Silhouette Dataset

In order to train a model for outlining buildings I needed a quick dataset. So I ran the masking and inpainting process over the training data from the scene occlusion dataset. I then labelled these images each with one instance of 'building' and this became the training data for the new set. I thought to use inpainted images for the training data since these would be the kind of images being fed into the start of the pipeline. 

The inpainted test images from the scene occlusion dataset were then taken as the test data for the new dataset. The labelled and testing images were both from a scene occlusion model that was trained for 20,000 iterations.

The labelling was once again done by my hosted instance of COCO Annotator.

![](/assets/img/honours/sileval.png)
*An example of the instance segmentation from a model trained on the dataset for 20,000 iterations.*


### Cutter script

Now that I had a model to outline the building instance in an image, I needed to "cut" it out. Which I did by making the rest of the image transparent. 

Using the trained model I can create a matrix of boolean values that has the same number of columns and rows as the given image. This is the mask of the instance on the image. I convert the image to have a alpha channel. And then using the mask matrix, if the coordinate in the image is False I set the alpha to zero. The resulting image is just the areas determined to be a building by the model.

![](/assets/img/honours/silcut.png)
*An example of the building cut out from a model trained on the dataset for 20,000 iterations.*

Next I need to add this process as a module to the conductor script, and then move on to implementing the folding process.