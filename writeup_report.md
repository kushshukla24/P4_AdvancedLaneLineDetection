# **Advanced Lane Line Detection** 

---

**Advanced Lane Finding Project**

Self-Driving Cars have a camera mounted over the roof-top, which provides vision to the car.
The goal of this project is to create a pipeline to detect lane lines on road in a frame captured from a roof mounted camera.

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

[image1]: ./output_images/writeup_images/distUndist.png "Undistorted"
[image2]: ./output_images/writeup_images/distCorrection.png "Distortion Correction"
[image3]: ./output_images/writeup_images/colorTransforms.png "Binary Example"
[image4]: ./output_images/writeup_images/warped.png "Warp Example"
[image5]: ./output_images/writeup_images/laneLineDetection.png "Fit Visual"
[image6]: ./output_images/writeup_images/laneAreaCurvature.png "Output"
[image7]: ./output_images/writeup_images/curvatureCalculation.png "Curvature Maths"
[video1]: ./project_video.mp4 "Video"

---

### Concepts Used
* Camera Calibration
* Calculating Distortion
* Perspective Transformation (Bird-eye View)
* Gradient Thresholding
* Sobel Operator
* Color Spaces
* Color Thresholding
* Sliding Window Search
* Polynomial Fitting
* Curvature Measurement


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb'.  

I start by preparing object points, which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `obj_points` and `img_points` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Distortion coefficients are calculated using camera calibration methodology and applied to the test images. Below is one of the test images:

![alt text][image2]

Undistortion in the image above can be perceived by the change in the shape of the hood of the car at the bottom. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I tried combinations of color and gradient thresholds to generate a binary image.
For applying Color Transformation I performed following steps:
* Converted the image to respective color space
* Extracted the channel for thresholding
* Created Binary Image only if the extracted channel has non-zero values above threshold

For applying the Gradient Transformation, I performed following steps:
* Converted the image to Gray Scale
* Applied Sobel Gradient based on x direction (as we are concerned with the vertical lines for lane detection)
* Normalized the gradient
* Created Binary Image only if the extracted gradient has non-zero values above threshold

The code for this step is contained in the 4th code cell of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb'.

I tried variations following color thresholds:
* Saturation in HLS Color Space
* Lightness in HLS Color Space
* b(blue-yellow) in Lab Color Space
* Absolute Sobel Gradient in x direction

Inspired by the work @ https://github.com/jeremy-shannon/CarND-Advanced-Lane-Lines
I finalized with using Lightness in HLS color space and b(blue-yellow) in Lab Color Space as they worked well for detecting the relevant lane lines for the project video images.

Here's an example of my output for this step for a test image.

![alt text][image3]

The Lane-lines are clearly visible with these color transformation.

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspectiveTransform()`, which appears in cell 7 of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb'.  The `perspectiveTransform()` function takes as inputs an image (`img`).  I chose the hardcode the source and destination points in the following manner:

```python
srcPoints = np.float32([[w, h-10],
                          [0, h-10],
                          [550, 460], 
                          [750, 460]])
offset = 10
destPoints = np.float32([[w-offset, h-offset],
                          [offset, h-offset],
                          [offset, offset],
                          [w-offset, offset]])
```
where, w and h is the width and height of the image.

Our images are of w = 1280 and h = 720
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 1280, 710     | 1270, 710     | 
| 0, 710        | 10, 710       |
| 550, 460      | 10, 10        |
| 750, 460      | 1270, 10      |

Below is the comparison of the original image and warped counterpart. I can be easily seen that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for my lane line detection utilizes sliding window method which is includes in function called `sliding_window_polyfit()`, which appears in cell 10 of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb'. 

The sliding window method takes undistorted, color and perspective transformed binary image.
Here is the algorithm for sliding window:
1. Create a histogram of pixel values in bottom half of the image.
2. Look for the peaks in left and right of the mid-points. The left side will give the position of left lane line and the right one will give the right lane line depicted in image.
3. Create a small window which will move in vertical directions from the two detected foundation points for left and right lane.
4. From the detected left and right position of step 1, start moving the window in vertical direction.
5. If the good amount of pixels (above threshold) are captured inside the window, store the pixels. Recenter the window to fit the pixels properly and move forwrard in up direction uptill height of the image.

This approach will give us pixel points in image which are contenders for left and right lane lines on image. 

The stored points are then used to fit a second degree polynomial which will give us a smooth shape for the entire height of the image depicting the path of the lane lines. This step needs to be done individually for left and right lane lines.

Here is the image after processing the above algorithm over a test image:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did wrote a method `calculateCurveRadiusAndDistanceFromCenter` in cell # 17 of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb' to calculate the lane curve and its distance from the center.

I utilized the below mathematics to come up with the numerals
![alt text][image7]

The tricky part was conversion of pixel values to meters (real scenario measurements).
I assumed the lane lines are 3.7 m wide and this part is captured with 700 pixels on image. Similarly the lane lines show 30 m of road way and this is captured by height (720 pixels) in the image. With these assumption, I came up with constant coefficient to map the pixel values in (meters).



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I incorporated all these steps in a function `process_image`in cell # 27 of the IPython notebook located in './P4_AdvanceLaneLineDetection.ipynb'.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I invested a lot of time in parameter tuning for distinctly detecting the lane line features through color and gradient thresholding. Other major time was spent in understanding the sliding window search methodology and calculating the curvature radius.

Based on my implementation, I can see following scenarios on which the lane detection will likely fail: 
1. For this project, I have just used the color thresholding for lane detection, so its not robust enough to working in different lightening condition. I faced same issues when tried my pipeline over the harder_challenge_video.
2. The sliding window search is not very robust and may result in wrong predicted lane line due to varying pixel intensities in some situations.

Two areas I can see improving the current implementation is:
1. Exploring more color spaces and intelligent color thresholding techniques
2. Making the window search algorithm more robust will surely help.

I will surely pursue to work further on this project to ensure a better and robust lane detection pipeline.