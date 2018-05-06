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

[image1]: ./output_images/AnnotatedImage.PNG "Annotated"
[image2]: ./output_images/DetectedLanes.PNG "Detected lanes"
[image3]: ./output_images/Histogram.PNG "Histogram"
[image4]: ./output_images/PipelineResult.PNG "Pipeline"
[image5]: ./output_images/ThresholdingContribution.PNG "Thresholding contributions"
[image6]: ./output_images/Undistorted.PNG "Undistorted"
[image7]: ./output_images/WarpedBinaryImage.PNG "Warped binary image"
[image8]: ./output_images/WarpedImage.PNG "Warped image"
[image9]: ./output_images/UndistortedRoad.PNG "Undistorted Road"
[image10]: ./output_images/test5.jpg "distorted Road"
[image11]: ./output_images/Radius_curvature_formula.PNG "Radius of Curvature"

[video1]: ./project_video_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in cell 2 of the IPython notebook "AdvLaneDetection_P4.ipynb"

I start by preparing "objpoints", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners (using cv2.findChessboardCorners) in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. To reuse the coefficients later on, I save them to a pickle file. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image6]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to the test image
For example, the following images show a test image and its undistorted image:

![alt text][image10]

![alt text][image9]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds in the pipeline() (cell 6) function to generate a binary image. 
The input image is first converted to grayscale and then horizontal sobel + scaled sobel thresholds are applied.
The input image is then converted to HLS color space and the s channel is thresholded using appropriately chosen thresholds to produce a binary image that is later combined with the previous binary image.
Here's an example of my output for this step.  

![alt text][image4]

This image shows the contribution of each thresholding method:

![alt text][image5]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transformation()`, which appears in cell 12 of the IPython notebook).  This function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. It also takes the calibration coeffecients that are loaded from the saved pickle file. I chose the hardcode the source and destination points so as to represent the lane region of interest excluding the hood of the car. 

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 543, 480      | 300, 0        | 
| 307, 645      | 300, 700      |
| 1003, 645     | 900, 700      |
| 730, 480      | 900, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. This was tested on both binary and color images.

![alt text][image8]

![alt text][image7]


#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The lane-line pixels are identified in find_lines() in cell 16 of the notebook. This method takes as input the binary warped image and computes histograms across columns in multiple sliding windows throughout the image. The peaks in the histograms are then analyzed to decide on the lane-line pixels. The first peak is assumed to be the left lane-line and the next peak, the right lane-line. The location of the lane-lines in the current window contributes towards the initial search position in the next window.

Then I fit my lane lines with a 2nd order polynomial to approixmate a complete line:

![alt text][image2]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The metrics - radius of curvature and offset from center, are computed in the Metrics section of the IPython notebook. 
'get_radius_of_curvature()' and 'get_offset()' are the corresponding methods to compute these metrics in the real-world space.

The radius of curvature computation uses the following information:

![alt_text][image11]

The offset from center computation uses the lane-line approximations from polyfit for the left and right lines to compute the center of the lane. The center of the camera (image) is assumed to be the center of the vehicle and so the difference in the centers of the lane region and the vehicle gives us the required number.

To convert to real-world space I assume 30 meters per 720 pixels in the vertical direction and 3.7 meters per 700 pixels in the horizontal direction. 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The detected lane region is drawn onto the original image using the warp_and_draw_lines() method in cell 25.
The computed numbers are added to the image using the annotate_image() method in cell 27 

![alt text][image1]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

All the above steps are used to generate the below video output (process_image() in cell 30). An improvement to the lane detection in video frames was the addition of a moving average filter that uses lane-lines data from the previous 5 frames to decide whether the lane data for the current frame is valid. If invalid, the data from the previous frame is used.  

Here's a [link to my video result](./project_video_output.mp4)
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I initially faced problems with false positives when illumination and the road conditions changed. This was expected because a threshold empirically deduced using a few images will not be applicable to all possible cases since it is heavily dependent on illumination. Also, when new objects (other cars) entered the region of interest, they introduced false spikes in the histogram that also generated bad data. 
To overcome these problems, I implemented a moving average filter that uses data from previous frames to decide if the current data is good or bad.

A minor problem I faced was with the need to keep track of the calibration coeffecients for use later on. I used a pickle file to solve this issue.

I expect this approach to fail in the following conditions:
1. Other objects enter the lane region
2. Shape of lane lines changes frequently in small intervals
3. Vehicles in close proximity block view of lane lines
4. Bad visiblity and Illumination
5. Calibration coeffecients become invalid because of distortion from windshield for cases when camera is inside the car 
