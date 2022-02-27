---
title : Summer Wrap-up - Partial Pipeline
notetype : feed
date : 22-02-2022
tags : [Summer Research Project]
---

So, we have reached the end of the summer school period. Meaning that I am concluding the work on this project as far as it being done under the summer scholarship. This doesn't mean that I am done with the project, but that it is just being put down until the first semester study period; where I will continue to work on it as my honours project.

This post serves as a summation of the work up until now, and will outline how to use the partial pipeline that I have created.

## Wrap-up

To see the work I have done so far fully detailed please refer to the [tags page](https://tommyhasselman.com/tags/) and see the posts linked under the "Summer Research Project" tag. 

In summary I have created the base functionality of a pipeline that can currently be given a image of a building in a scene, identify objects in the scene that could be occluding the building and mask them, then inpaint those objects. This ideally results in a "cleaner" image of the building without much else in the scene.

Original Image | Inference from model
:---:|:---:
![](/assets/img/studentship/IE221695_og.jpg) | ![](/assets/img/studentship/IE221695_inf.jpg) 

Mask | Inpainted Image
:---:|:---:
![](/assets/img/studentship/IE221695_mask.png) | ![](/assets/img/studentship/IE221695_inpainted.png) 

This pipeline is intended to carry on and isolate the building from the image, analyse its geometry, and create a 3D model of the buildings visible facades. This is then to by used as content for augmented reality applications, as [demonstrated](https://tommyhasselman.com/posts/Update-2-Milestone-Reached) at the start of this project.

## Using the partial pipeline

This section is mainly for my supervisor since anyone else reading this likely won't have access to my project files, but it should give a good example of the process anyway.

### Pre-requisites
Clone the repository using [this](https://github.com/RedSoutherly/deep/tree/summer) tag from the time of writing this.

The python scripts require a range of dependencies to work. Following the detectron2 [install guide](https://detectron2.readthedocs.io/en/latest/tutorials/install.html) should set these up for you. I was able to use an anaconda environment to make this easier. The scripts also need to be run on a machine that has access to CUDA cores. I ran it on the departments deep server.

The dotnet project requires .NET version 2.0 to run.

### Overview

Currently in my repository there are three python scripts, a dotnet project, and my dataset. The python scrips are differentiated as trainer, tester, and masker. The trainer script is of course used to train a model on a dataset, the tester script is used to create inference images on the model like you can see above, and the masker script is is similar to tester but instead of an inference image it outputs a PNG binary mask. The dotnet project is a slightly altered copy of the base implementation of [zavolokas' inpainting project](https://github.com/zavolokas/Inpainting) from GitHub. And the dataset is my custom occlusion dataset labelled in COCO format. The directory structure is as follows:

```
Project
├── deep_trainer.py
├── deep_tester.py
├── deep_masker.py
├── scene_occlusion
│   ├── train
│   │   └── train.json
│   ├── test
│   └── models
├── inpainting_targets
│   ├── images
│   └── output
└── inpainting_zav
    └── Inpaint
        └── Program.cs
```
*Irrelevant directories not shown.*

### Dataset setup
Before any training sessions have been run the dataset directory will only have "train" and "test" sub-directories. Both contain sets of images for what I hope is clear purposes. The only difference being that the train directory will contain a file named "train.json" (must be named this for scripts to work), this is the json file containing the COCO annotation data for all the images in the train directory. The test images do not need to be annotated currently.

### Training a model
In order to run a training session on the dataset we use the deep_trainer.py script. Making sure that the script is adjacent to the dataset in the directory as shown, we can pass some command line arguments to train a model. The first argument is the directory name of the dataset we want to use, the second we use to tell the script how many classes the dataset contains, and the third is the amount of iteration we want to train for.

So to train the scene_occlusion dataset (which has one class) for 5000 iterations; we would run the following command:
```sh
python deep_trainier.py scene_occlusion 1 5000
```

This may take some time to run but once it is done you will find a new sub-directory in the dataset directory named "models". This directory will contain all the models that you have trained on this dataset. They are labelled with their iteration count and a timestamp as follows: {iterations-year-month-day-time}. 

### Using a model | Testing and Masking
Now that we have a model trained we can use it with both the tester and masker scripts. Both scripts take the same two arguments to run: the name of the dataset and the ID of the model in that dataset you want to use (this is the directory label with the date in it).

So the commands for one of the models I have done would be as follows:
```sh
python deep_tester.py scene_occlusion 20000-2022-02-08-1948
```
```sh
python deep_masker.py scene_occlusion 20000-2022-02-08-1948
```

Note that you won't be able to use this exact model as the binary file is too large to store in the repository properly. So you will need to use the ID of a model you have trained locally.

These two scripts will make two different sub-directories within the models directory. Tester will make an "infeval" directory. This directory will then contain timestamped directories of each time the script has been run. Which will contain the images of the models inference on the testing set of images. The masker script will create a sub-directory named "masks". This will also contain timestamped sub-directories which contain the png masks for each image in the testing directory.

### Inpainting
Using the inpainting program is currently a little more manual. In the inpainting_targets directory is a images directory. This is where you need to place the images you want inpainted (likely the originals from the dataset test directory), and with them place the matching png masks from a given model. The masking script should label the masks as the original file name but as a png file instead of a jpg. So for every jpg in the directory there should be the same as a png (This being the image and mask pairs). I.e. a.jpg and a.png are present.

Once the target images are ready move to inpainting_zav/Inpaint/ and run the program using the following command:
```sh
dotnet run
```

This currently takes quite some time to run as the program does many iterations over each image. But once it is complete the results will be in the output sub-directory of the inpainting_targets directory.

## Conclusion
And that is currently as far as the pipeline goes.

While it can seem that the pipeline doesn't do much at the end, it has taken quite a bit of background process to produce that inpainted output. Next steps on this pipeline would be to extract the building from the inpainted image, analyse the buildings geometry, and then fold it into our 3D façade. This is what I will be working on for my 2022 honours project.
