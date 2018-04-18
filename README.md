# **Udacity Self-Driving Car Nanodegree | Project 4** 

---

**Advanced Lane Finding**

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

List of files in Repository:
1. ***P4_AdvanceLaneLineDetection.ipynb*** is the python notebook containing all the implementation of the project.
2. ***camera_paramters.p*** is the pickle file to stire the calibrated camera parameters to avoid re-calibrating again.
3. ***writeup_report.md*** is the markdown for summarizing the methodology and results
4. Other Folders and files contains resources are either the input/output images/videos for implementation or the resources for the writeup.

Many components of this projects are inspired from https://github.com/jeremy-shannon/CarND-Advanced-Lane-Lines.

