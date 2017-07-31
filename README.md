# carnd-advanced-lane-lines
The 4th project of the udacity Self-Driving Car Nanodegree term1.

---
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

[image1]: ./output_images/camera_cal.png "Camera Calibration"
[image2]: ./output_images/undistorted.png "Undistorted"
[image3]: ./output_images/thresholded_binary.png "Binary Example"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/curve_fit.png "Fit Visual"
[image6]: ./output_images/processed.png "Output"
[image7]: ./output_images/radius.png "Radius of Curvature"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first and the third code cell of the IPython notebook located in "./advanced_lane_finding.ipynb".   

I used the `cv2.findChessboardCorners()` function to find 2D corner points in the given chessboard image which are corresponding to real world 3D points. It needs at least 10 test chessboard images to find the good mapping. After the sufficient mappings found, I called the `cv2.calibrateCamera()` function to get the camera calibration matrix and distortion coefficients. Then I applied them to the test image using the `cv2_undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (test images)

#### 1. Provide an example of a distortion-corrected image.

Here is the example of the distortion corrected test image. The code can be found in the 3rd code cell of the IPython notebook:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tried several color thresholds, gradient thresholds and combination of them. I ended up using color thresholds only. The gradient-based thresholds created too much false positives and did not work well on the challenge video. 

I used color thresholds to find white color and yellow color in the image. It's proved to be robust enough to find lane lines in both project video and challenge video. The B channel in RGB color space works quite well on detecting white color and the yellow color was detected by using the b channel in LAB color space (the code is in the 4th code cell). Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is the `warp_image()` function, which appears in the 5th code cell. I used the `cv2.getPerspectiveTransform()` function to perform a perspective transform. It takes source and destination points as inputs. Both points are static and the source points were found by eyeballing the test image (`test_images/straight_lines1.jpg`).

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 555, 475      | 200, 0        | 
| 730, 475      | 1080, 720      |
| 1060, 680     | 1080, 720      |
| 240, 680      | 200, 0        |

I verified that my perspective transform was working as expected by drawing the source and destination points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I used the histogram to find the peak X positions and searched upward from those positions to find the left and right lane lines. The newly detected lane line would be rejected if its curvature was deviated too far from the previous one. The result was smoothed over the last 5 frames of video to have a more stable lines.

After finding the good lane lines, I used their pixels to fit a 2nd order polynomial curve. The code is contained in the 7th code cell of the the IPython notebook.

Here's an example of the lane line identified result.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the 6th code cell of the IPython notebook. The radius of curvature was calculated by using the following equation given a second order polynomial curve:

![alt text][image7]

The vehicle offset is the difference between the detected lane center and the camera center, assuming the camera is mounted at the center of the car. The measurements are converted from pixels to meters according to the lane's width and height projected in the warped image. The conversion ratio is in the 2nd code cell of the IPython notebook.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the 8th code cell in the `process_image()` function. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here are my video results:

- [The project video](./output_videos/project_video_output.mp4)
- [The challenge video](./output_videos/challenge_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My pipeline doesn't work on the harder challenge video at all and it's a little bit sensitive to the lighting, which may cause no lane lines detected. I might need to use gradient-based thresholds as a fallback in case of this issue.

I spent a lot of time on tuning thresholds for identifying lane lines. There seems no perfect thresholds that can handle all three videos very well. I always need to make compromise. I believe this is not a good approach to generalize my pipeline. I might try to apply the deep learning to this project to see whether the pipeline could generalize well by feeding it enough data.
