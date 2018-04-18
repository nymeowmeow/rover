## Project: Search and Sample Return


**The goals / steps of this project are the following:**  

**Training / Calibration**  

* Download the simulator and take data in "Training Mode"
* Test out the functions in the Jupyter Notebook provided
* Add functions to detect obstacles and samples of interest (golden rocks)
* Fill in the `process_image()` function with the appropriate image processing steps (perspective transform, color threshold etc.) to get from raw images to a map.  The `output_image` you create in this step should demonstrate that your mapping pipeline works.
* Use `moviepy` to process the images in your saved dataset with the `process_image()` function.  Include the video you produce as part of your submission.

**Autonomous Navigation / Mapping**

* Fill in the `perception_step()` function within the `perception.py` script with the appropriate image processing functions to create a map and update `Rover()` data (similar to what you did with `process_image()` in the notebook). 
* Fill in the `decision_step()` function within the `decision.py` script with conditional statements that take into consideration the outputs of the `perception_step()` in deciding how to issue throttle, brake and steering commands. 
* Iterate on your perception and decision function until your rover does a reasonable (need to define metric) job of navigating and mapping.  

[//]: # (Image References)

[image1]: ./misc/rover_image.jpg
[image2]: ./calibration_images/example_grid1.jpg
[image3]: ./calibration_images/example_rock1.jpg 
[image4]: ./output/warped_example.jpg
[image5]: ./output/warped_threshold.jpg
[vid1]:   ./output/test_mapping.mp4 

## [Rubric](https://review.udacity.com/#!/rubrics/916/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!
![alt text][image1]

### Notebook Analysis
#### 1. Run the functions provided in the notebook on test images (first with the test data provided, next on data you have recorded). Add/modify functions to allow for color selection of obstacles and rock samples.

We first capture images from the robot perspective. And then choose source points for the grid cell in front of the rover (each grid cell is 1 square meter) (left)

and then perform a perspective transformation, to provide a top down view of the world.(rigth)

![alt text][image2] ![alt text][image4]

next we has to differentiate which part of the image is obstacles, rocks or open space. We did it by apply color threshold. Obstacles tends to be much darker, open space is lighter and rocks is yellowish in color. By applying different RGB threshold to the image, we classify which pixels is open space, obstacles or rock, and a typical image  after we apply color threshold is as shown below.
![alt text][image5]

#### 1. Populate the `process_image()` function with the appropriate analysis steps to map pixels identifying navigable terrain, obstacles and rock samples into a worldmap.  Run `process_image()` on your test data using the `moviepy` functions provided to create video output of your result. 

In process_image, we perform the following steps to identify navigable terrains, obstacles and rock smaples.
1. perform perspective transform as mentioned above
2. apply color thresholding to the image, to see which part is navigable terrains, rocks or obstacle. 
3. convert robot centric pixel value to world coordinates
4. update worldmap
![alt text][vid1]

### Autonomous Navigation and Mapping

#### 1. Fill in the `perception_step()` (at the bottom of the `perception.py` script) and `decision_step()` (in `decision.py`) functions in the autonomous mapping scripts and an explanation is provided in the writeup of how and why these functions were modified as they were.

in Perception_step, has to first define the threshold for classifying navigable terrains, rocks or obstacles. Then i add the logic to classify rocks. initially the fidelity is not doing well, i have to add the constraint when the pitch or roll angle is > 0.5 then do not update the world map.

In decision_step, i add the code to check if we are in the pick up model or near sample, so we should stop the Rover and do the pick up. Also if rocks are close by, we should drive the Rover towards the rock in order to pick it up.

The code is modified to use median instead of mean, since it is less sensitive to outlier.

#### 2. Launching in autonomous mode your rover can navigate and map autonomously.  Explain your results and how you might improve them in your writeup.  

**Note: running the simulator with different choices of resolution and graphics quality may produce different results, particularly on different machines!  Make a note of your simulator settings (resolution and graphics quality set on launch) and frames per second (FPS output to terminal by `drive_rover.py`) in your writeup when you submit the project so your reviewer can reproduce your results.**

I am running the simulator with 1024x768 screen resolution, good graphics quality.

Initially the fidelity is not that great, but after adding the constraint the pitch and roll angle has to be < 0.5, before i will update the world map, the fidelity becomes much better. 

Sometimes the Rover get stuck, so i have to add some simple logic to identify when it get stuck, the logic implemented is very simple, when the rover is marching forward, but its velocity is low (0.2), and this happens for extended period of time (100 cycle), then i assume the rover is stuck. And then i will try to reverse the direction and rotate the rover.

To find the direction for marching forward, i took the median of the navigable angles instead of the mean, which should be less sensitive to outliers, and perform better.

Sometimes the Rover will go into circle, and need manual intervention to break the loop. Ideally, the Rover will remember where it has visited, and if it coming back to the same location, it will try to break the loop. Initially i add logic to count number of times the Rover moves in same direction, if it exceed certain threshold then i assume it is in a loop, and must do recovery, however this doesn't work too well, so i took it out, and the Rover can run into this looping issue.




