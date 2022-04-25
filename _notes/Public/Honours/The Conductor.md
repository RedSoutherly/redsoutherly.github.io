---
title : The Conductor
notetype : feed
date : 24-04-2022
tags : [Honours]
---

Hello again and welcome back. This is my first update to this blog following the start of my honours year. The project for my honours is the continuation of the project I worked on over the summer, so if you haven't already feel free to read the previous update posts to familiarise yourself with the project. And now that this project is for my honours, it has a name: **XRephotography: Immersive 3D Exploration of Historic Photography**.

Now that's enough intro, into the subject of this post.

### The Conductor

One of my first goals for the project was to create a script that streamlined the process of running the inpainting program. This was because the inpainting program is implemented in C# and runs on the dotnet framework, while the rest of my pipeline modules are implemented in Python. 

The original scope of this script was to simply pass some command line arguments which were then used to place data in the correct relative directories to the inpainting program before calling it. However, this scope grew and turned into a helper program used to run all of the pipeline modules in the project. Aptly named conductor due to its orchestration of the pipeline. The script does not require any arguments to be passed, and selecting datasets/models/etc is done by prompting the user with options found by scanning the project directories. So a new dataset or model will appear in the options for subsequent tasks.

```
What do you want to do?
1. Train

2. Test

3. Mask

4. Inpaint

5. Full Pipe (1, 3, & 4)


Enter a number (or type quit to quit):
```

Running the script presents the user with the current runnable modules, and option(s) to run modules in sequence as a pipeline segment. Any required arguments for the modules will be prompted for the user to select/input. Modules will run to completion and display a completion message, then the program will hang until the user presses a key to return the the main menu where they can run mode modules or quit the program.

Overall this script has made it easier for me to rapidly run the modules without having to copy/paste directory labels and move data around manually. The ability to also run pipeline segments unattended is beneficial for testing chains of processing, and helpful due to the lengthy processing time for the inpainting program.