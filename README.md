
## Writeup

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./Pictures/undistorted_chessboard.PNG "Undistorted"
[image2]: ./Pictures/undistorted_lane.PNG "Road Transformed"
[image3]: ./Pictures/both_feats0.PNG "Feature Map 1"
[image3b]: ./Pictures/both_feats1.PNG "Feature Map 1"
[image3c]: ./Pictures/Number_of_Features.PNG "Feature Map 1"
[image4]: ./Pictures/perspective_transform.PNG "Warp Example"
[image6]: ./Pictures/complete.PNG "Output"
[video1]: ./solution_project.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

All of my code can be found in my IPython notebook in "./My Project 3.ipynb" . In the following, I will consider all rubric points.

### Camera Calibration

#### 1. Calibration and Distortion:

The code for this step is contained in the second code cell of my notebook. My approach was the one presented in the lecture. Mainly, I used the provided pictures of chessboards and the cv2-functions findChessboardCorners and calibrateCamera to calibrate the camera.

I applied this distortion correction to the chessboard images and obtained the following result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The next image illustrates the distortion correction:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used two feature maps. The first one took the S-channel, the R-channel, the grayscale, the sobel operator in 'x' direction and the magnitude of the sobel operators into account. If this feature map had too few pixels, I flagged it with "VeryBad" and did no change to the previous lane parameters. If the number of pixels was below a certain threshold I fitted a second order polynomial through all points of the feature map in the neighborhood of the last lane fit.

However, if there were too many pixels, I flagged the feature map with "Bad" and tried another map that just consists of the S-channel and the sobel operator in 'x' direction. Again, if too few or too many pixels were found, I used the "VeryBad" flag. Otherwise, I fitted a polynomial as in the other case.

My pipeline consists of several steps where I first try high-thresholded filters to find lane line, and if this does not work, continue with lower-thresholded filters. In total, I used the S-channel and the R-channel (mainly to detect yellow lines), the grayscale (mainly to detect white lines), and also the Sobel-operator in 'x'-direction as well as the magnitude of the sobel operator.

Additionally, I used a median filter to sharpen edges. In the following picutres, we can see that the first feature map consists of more information but fails in certain situations where the second feature map still works fine.
 
 
![alt text][image3]
![alt text][image3b]


In the following graph, we can see that the number of feature pixels varies a lot for the two difficult situations in the project video:

![alt text][image3c]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Given the calibration matrix and distortion parameters, I used the picture 'straight_lines2.jpg' to calibrate the perspective transform. I chose the following source and destination points of my rectangles manually:

Position    | Source        | Destination   | 
:----------:|:-------------:|:-------------:| 
lower_left  | 280, 678      | 300, 719      | 
lower_right | 1040, 678     | 900, 719      |
upper_right | 705, 460      | 900, 0        |
upper_left  | 580, 460      | 300, 0        |


![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the first frame, I used the sliding window technique given in the lecture. Then, I fitted a second order poylnomial to the lane lines.

For the following frames, I fitted a second order polynomial through the feature map pixels that were close to the previous fitted line. Therefore I constructed two 'safe lines' for each lane and only considered the area inbetween them (see picture below). Also, I averaged the parameters of the polynomial to obtain smoother results.

If the feature map did not work well, it did not apply any changes to the previous parameters.



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this analogously to the lecture and the code can be found in my Ipython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The following pictures represent my pipeline. The green area marks the area between the lanes. The blue lines are the found lanes. The red lines are the 'safe lines' around the blue lanes. The red circles mark the 'bottom point' of each lane.


![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./solution_project.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found that my pipeline fits the project video but does not generalize so well to the other two videos. Here, the algorithm had some difficulties with shadows and sometimes recognized the shadows as lane lines.

I order to improve my pipeline, I could try out more features, especially to prevent fitting on shadow lines. Also, I noticed that I used lots of 'hard-coded' features. Maybe a machine learning approach would work better here.
