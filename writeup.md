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

[image1]: ./output_images/binary_warped_image.png "Binary warped" 
[image2]: ./output_images/detected_lane_final_result.png "Detected lane final result"  
[image3]: ./output_images/detected_lane_win_poly.png "Detected lane windows and poly. func."    
[image4]: ./output_images/original_test_image.png "Test image" 
[image5]: ./output_images/stacked_thresholds_image.png "Thresholds"      
[image6]: ./output_images/undistorted_test_image.png "Undistorted"
[image7]: ./output_images/combined_image.png "Combined"      
[image8]: ./output_images/detected_lane_histogram.png "Histogram"     
[image9]: ./output_images/original_chessboard_image.png "Chessboard"      
[image10]: ./output_images/undistorted_chessboard_image.png "Undistorted chessboard"
[video1]: ./test_videos_output/project_video.mp4 "Final result video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in [Here](https://github.com/patdring/CarND-Term1-Advanced-Lane-Lines/blob/master/P4.ipynb)  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image9]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image4] 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (this step is contained in the third code cell).  Here's an example of my output for this step.

![alt text][image7] 

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is contained in the forth code cell.  This code takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the source and destination points in the following manner:

```python
img_size = (combined_binary_test_images[2].shape[1], combined_binary_test_images[2].shape[0])
offset = 85
src = np.float32([
    [  585,   445 ],
    [  690,   445 ],
    [ 1125,   675 ],
    [  150 ,  675 ]])

dst = np.float32([
    [offset, 0], 
    [img_size[0] - offset, 0], 
    [img_size[0] - offset, img_size[1]], 
    [offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 445      | 85, 0         | 
| 690, 445      | 1195, 0       |
| 1125, 675     | 1195, 720     |
| 150, 675      | 85, 720       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` colored points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image1]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

To decide which pixels are part of the lines and which belong to the left line and which belong to the right line I took a histogram (only on the lower half of the image).
The two peaks in this histogram are good indicators of the x-position of the base of the lane lines. A sliding window, placed around the line centers, find and follow the lines up to the top of the frame.
After estimated which pixels belong to the left and right lane lines a second order polynomial curve is determined to those pixel positions. 

![alt text][image8]

![alt text][image3]

This step is contained in the fifth code cell.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature of the curve at a particular point is defined as the radius of the approximating circle. This circle closely fits nearby points on a local section of a curve. For more detals about the math behind see [Here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php) 

This step is contained in the sixth code cell.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

This step is contained in the seventh code cell.  Here is an example of my result on a test image:

![alt text][image2]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./test_videos_output/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took:

##### 1. Camera calibration using a chessboard image

Camera calibration worked equally well with all test images.

##### 2. Distortion correction of test images

Interesting here was the effect of deliberated deactivation of correction on the late result (drawed lines on image). The impact is noticeably great

##### 3. Color/gradient threshold

Thresholds with gradients (Sobel operator in the x-direction) identified the lane lines by gradient direction alone. But there was still noise in the resulting image.
I converted the image to the HLS color space. The S-channel is robust of picking up the lines under very different color and contrast conditions so I combined it with 
gradients thresholds result. In each case I had adjusted the threshold parameters to do as good as possible of picking out the lines. That was the most time-consuming part.

##### 4. Perspective transform

I defined 4 source points and 4 destination points, used cv2.getPerspectiveTransform() to get the transform matrix and finally warped the image to a bird view with cv2.warpPerspective(). 
Four colored dots helped to choose the right area. 

##### 5. Detect lane lines

The histogram and its increasing or decreasing peaks were very helpful during development. 

##### 6. Determine the lane curvature

##### 7. Draw Identified lanes on Image

For pipeline tests and implementing (especially adjusting the thresholds) I analyzed the examples and chose the most complicated one. This is how I evaluated my pipeline

### Where could the pipeline fail?

##### Conditions that make the road boundaries disappear, e.g.:
- Weather conditions (a grey and foggy day). 
- Light conditions so that no lines can be seen in the S-channel either (sunset or sunrise, tunnels)
- Very thin road limits that are incorrectly filtered out.
- Vehicles on the same lane covering the track lanes.

### What could it make more robust?
- Introducing and combining further tresholds (other color spaces, edge detection algo., ...) in the third code cell. 
- Including vehicle data such as speed and steering angle or weather and date/time/map data from the online infotainment system could help to filter out very unlikely predictions. 
- Several independent pipelines whose results are only combined at the end. Depending on the external conditions detected, these pipelines can also be prioritized. 
   E.g. one pipeline for good weather conditions, another for sunset.
 

