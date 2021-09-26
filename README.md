# **Advanced Lane Finding** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)


![Lanes Image](./images/project_output.gif)

---
## Introduction
When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project, I have used some advance computer vision algorithms and created a pipeline to identify the lane boundaries in a video.

You can see all the code and output images from **Lane-Detection.ipynb**.

## Implementation

Example outputs in **Lane-Detection.ipynb** created by images in **test_images** folder
First, I have implemented every function and tested them with images in **test_images**. 
Lastly, I have combine them in a function to create pipeline in `Pipeline for single image` section in **Lane-Detection.ipynb**.
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image. 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

You can find it in `Undistort Test Images` section of **Lane-Detection.ipynb**

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image.  
I have used absolute gradient for both orient (x and y) with threshold (20,100), magnitude gradient with threshold (100,255), direction gradient with threshold (0.3,0.7). Both are have kernel size 3. 
Combine this gradient threshold binary output with color threshold binary output that uses HLS color space. However, it uses only S channel
in order to work well on different light conditions. S channel is thresholded with the values (170,255).

Output of this process is a thresholded binary image. You can find examples in `Display Thresholded Images` section of **Lane-Detection.ipynb**

Implementation can be found in `Gradient Threshold Functions`, `Color Threshold Functions`, `Apply Threshold Filters To Images` and `Display Thresholded Images` sections

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Implementation and example outputs can be found in `Perspective Transform` section.
I have decided source and destination coordinates from previous lessons and by eye. After some trial, I have committed these pixel coordinates below.

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 590,450      | 300, 0        | 
| 700,450      | 970, 0      |
| 200, 720     | 300, 720      |
| 1130,720      | 970, 720        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
After prespective transform, I have created histograms for transformed binary images. I have divided them from middle. 
For the both lines, I have find max value of histogram in left and right sides of below of the image. 
Then, I have created windows at these peak points with margin +-100. After deciding the window area, I have gather nonzero pixel indexes and collected them in left_lane_inds and right_lane_inds.
When I finished a window, to start a new window, I had to decide where should mid x point of new window. To do that, there are two conditions. If there are not enough nonzero pixels at previous window, mid-x point should be same with before.
However, if there is enough nonzero pixels, I make mid-x coordinate as the mean-x of previous window nonzero pixel values. 
I have created 9 windows like that. Lastly, I have approximated two plots with the nonzero pixel indexes for both left and right lines.
These method is accurate and more powerful but does not have a good performance. Since, it examine all the image. 
Therefore, I have added a method for looking lanes with a prior information. In the pipeline for video input, I have used full image scan first, then I have used 
plot configurations as a prior information to look for new line position for all the frames after.
   
Implementation and output example images can be found at `Create Histogram`, `Find the Lines` and `Search from Prior` sections
  

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `Measuring Curvature` section.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `Lane Segmentation` section. Example images can be found in the same section.

---
### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](test_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
