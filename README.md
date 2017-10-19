
## Writeup Template

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

[image1]: ./Pics/undistorted_chessboard.PNG "Undistorted"
[image2]: ./Pics/undistorted_lane.PNG "Road Transformed"
[image3]: ./Pics/feature_map1.PNG "Feature Map 1"
[image3b]: ./Pics/feature_map3.PNG "Feature Map 2"
[image4]: ./Pics/perspective_transform.PNG "Warp Example"
[image5]: ./Pics/feature_map1.PNG "Fit Visual 1"
[image5b]: ./Pics/centroids.PNG "Fit Visual 2"
[image5c]: ./Pics/trusted_points.PNG "Fit Visual 3"
[image5d]: ./Pics/detected_lanes.PNG "Fit Visual 4"
[image5e]: ./Pics/top_view1.PNG "Fit Visual 5"
[image6]: ./Pics/complete_pic2.PNG "Output"
[video1]: ./solution_project.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

All of my code can be found in my IPython notebook in "./Write up.ipynb" . In the following, I will consider all rubric points.

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

My pipeline consists of several steps where I first try high-thresholded filters to find lane line, and if this does not work, continue with lower-thresholded filters. In total, I used the S-channel and the R-channel (mainly to detect yellow lines), the grayscale (mainly to detect white lines), and also the Sobel-operator in 'x'-direction as well as the magnitude of the sobel operator.

In the following pictures, we can see that more and more features are recognizeable. Also, features were only considered if they had a minimum number of white pixels. Additionally, I used a median filter to sharpen edges.
 
 
![alt text][image3]
![alt text][image3b]

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

In the first frame, I used the sliding window technique given in the lecture. Then, I fitted a weighted second order poylnomial to the lane lines.

For the following frames, I tried to find window centroids. Here, I also did this consecutively for every of my chosen features and combined them. Then, I fitted a weighted second order polynomial to the center points of the windows.

Some more tricks I used:
- I saved the bottom point of the left and right lane in a global variable and started the window-centroid method from that point.
- Windows that were too far from the last found window were discarded.
- I computed two 'safe lines' around each lane and used them as a mask.
- If a lane consisted of few points, I added three more points from the polynomial from the last iteration.
- If a lane consisted of few points, but the other lane had a confident fit, I transformed points from the confident lane to the other.
- I used a moving average over the parameters of the polynomials to smoothen the result.

In the first picture, we can see a feature map.

![alt text][image5]

Next, the window centroids are shown:
![alt text][image5b]

Now, we can see the trusted points (red). We can observe that some points of the left lane are transferred to the right lane:
![alt text][image5c]

Finally, the lane is drawn (yellow). The blue crosses are points of the lane that are transported to the next frame for more stability.
![alt text][image5d]


The next picutre shows the final top view, cluding including the blue found lanes, the red bottom points and the search area for lane pixels in the next frame (marked by the red safe lines).

![alt text][image5e]



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

I found that my pipeline fits the project video but does not generalize so well to the other two videos. Here, the algorithm had some difficulties with shadows and sometimes recognized the shadows as lane lines. Also, using a moving average to smooth the lanes resulted in less variable curves. Maybe, one could use a better idea here.

I order to improve my pipeline, I could try out more features, especially to prevent fitting on shadow lines. Also, I noticed that I used lots of 'hard-coded' features. Maybe a machine learning approach would work better here.
