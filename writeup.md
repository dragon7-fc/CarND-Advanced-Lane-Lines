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

[image1]: ./camera_cal/calibration1.jpg "./camera_cal/calibration1.jpg"
[image2]: ./output_images/calibration1_undistortion.jpg "./output_images/calibration1_undistortion.jpg"
[image3]: ./test_images/test1.jpg "./test_images/test1.jpg"
[image4]: ./output_images/test1_undist.jpg "./output_images/test1_undist.jpg"
[image5]: ./output_images/test1_undist_x_grad.jpg "./output_images/test1_undist_x_grad.jpg"
[image6]: ./output_images/test1_undist_y_grad.jpg "./output_images/test1_undist_y_grad.jpg"
[image7]: ./output_images/test1_undist_mag_grad.jpg "./output_images/test1_undist_mag_grad.jpg"
[image8]: ./output_images/test1_undist_dir_grad.jpg "./output_images/test1_undist_dir_grad.jpg"
[image9]: ./output_images/test1_undist_s_ch.jpg "./output_images/test1_undist_s_ch.jpg"
[image10]: ./output_images/test1_thresh.jpg "./output_images/test1_thresh.jpg"
[image11]: ./output_images/test1_undist_per_line.jpg "./output_images/test1_undist_per_line.jpg"
[image12]: ./output_images/test1_persp_line.jpg "./output_images/test1_persp_line.jpg"
[image13]: ./output_images/test1_persp.jpg "./output_images/test1_persp.jpg"
[image14]: ./output_images/test1_hist.jpg "./output_images/test1_hist.jpg"
[image15]: ./output_images/test1_window.jpg "./output_images/test1_window.jpg"
[image16]: ./output_images/test1_poly.jpg "./output_images/test1_poly.jpg"
[image17]: ./output_images/test1_lane.jpg "./output_images/test1_lane.jpg"
[image18]: ./output_images/test1_capt.jpg "./output_images/test1_capt.jpg"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the `camera_calibration()` function of the IPython notebook located in "./Advanced-Lane-Lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

 Original Chessboard |  Undistorted Chessboard
---------------------|--------------------
![alt text][image1]  | ![alt text][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the `undistort()` function of the IPython notebook.

To demonstrate this step, I applied the `cv2.undistort()`, as same as last step, to one of the test images like this one:

 Original Image      |  Undistorted Image
---------------------|--------------------
![alt text][image3]  | ![alt text][image4] 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at `threshold()` function).  Here's an example of my output for this step.


 Operation         |  Result              | Threshold Parameter
-------------------|----------------------|-----------
Undistorted Image  | ![alt text][image4]  |
X Gradient         | ![alt text][image5]  | Mininum=30, Maximum=100, Kernel size=15
Y Gradient         | ![alt text][image6]  | Minimum=30, Maximum=100, Kernel size=15 
Magnitude Gradient | ![alt text][image7]  | Minimum=70, Maximum=100, Kernel size=15 
Direction Gradient | ![alt text][image8]  | Minimum=0.7, Maximum=1.3, Kernel size=15 
S Channel          | ![alt text][image9]  | Minimum=170, Maximum=255
Thresholded Image  | ![alt text][image10]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective()`.  The `perspective()` function takes as inputs an image (`thresh`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
[[280,  700],  # Bottom left
 [595,  460],  # Top left
 [725,  460],  # Top right
 [1125, 700]]) # Bottom right

dst = np.float32(
[[250,  720],  # Bottom left
 [250,    0],  # Top left
 [1065,   0],  # Top right
 [1065, 720]]) # Bottom right 
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 280,  700     | 250,  720     | 
| 595,  460     | 250,    0     |
| 725,  460     | 1065,   0     |
| 1125, 700     | 1065, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

 Undistorted Image   | Perspective Transformed Image
---------------------|------------------------------
![alt text][image11]  | ![alt text][image12] 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

 Operation  |  Result
------------|---------------------
Perspective Transformed Image | ![alt text][image12] 
Threshoulded Perspective Transformed Image | ![alt text][image13] 
Find Histogram | ![alt text][image14] 
Sliding window search | ![alt text][image15] 
Fit a Polynomial | ![alt text][image16] 

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `measure_curvature_real()` function of the IPython notebook.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_lane()` and `caption()`.  Here is an example of my result on a test image:

 Drawed Lane Image   | Final output
---------------------|-----------------
![alt text][image17] | ![alt text][image18]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
