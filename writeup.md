## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[//]: # "Image References"

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./examples/undistort_road_output.png "Road Transformed"
[image3]: ./examples/threshold_example.png "Binary Example"
[image4]: ./examples/warp_example.png "Warp Example"
[image5]: ./examples/linefit_example.png "Fit Visual"
[image6]: ./examples/final_example.png "Output"
[video1]: ./projectvideo_output.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first four code cells of the IPython notebook located in the [](submission_code.ipynb#Calibration-matrix-computation).

I have created a class to encapsulate the camera parameters named `CameraParams`. This class has all the necessary camera parameters as its attributes

I have defined a function that computes and returns the camera parameters given the path of the calibration images. The function is named `cameraCalibration()` and is located in the third code cell of the notebook. The function first defines object points on 9x6 grids. These points represent the inner corners of the chessboard. Then for each calibration image, we find the pixels that represent the corners. These are the image points which, along with the object points defined above are passed to `cv2.calibrateCamera()` to get the camera parameters. The function then encapsulates these parameters in an object of `CameraParams` and returns this object.

The calibration images for this project reside in './camera_cal' directory, and are saved as jpeg files. That's what we pass to the above defined `cameraCalibration()` function from which we receive a `CameraParams` object.

The camera parameters are used to undistort an image taken from that camera to negate artefacts introduced by the camera lens. I use these parameters in a call to `cv2.undistort()` on an example image. The results are displayed below:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I have used color and gradient thresholds to generate a binary image. The various threshold functions are in [](submission-code.ipynb#Various-transforms) with the main threshold pipeline in `threshold()`  Within this function, I first generate a binary image obtained by thresholding the x, y gradient maps as well as the gradient magnitude and direction maps. I convert the original image to HLS color space and threshold the S channel. I finally combine these two threshold images to obtain the resulting binary image. Here's an example of my output for this step:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in the function `getPerspectiveMatrix()`. The function has hard-coded `src` and `dst` points which are then passed to `cv2.getPerspectiveTranform()` to obtain the the perspective transform matrix `M`. I also obtain the inverse transform matrix `Minv` .

An example of the warped image using the perspective matrix above is shown below:

![Warped Image][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code to identify the lane lines is in the section [](submission-code.ipynb#Lane-line-detection) . Before I move on to describe the pipeline, I will describe some important functions and data structures used by the pipeline. 

I first create two data structures names `Line` and `Lane`. `Line` has attributes that are used to store the important information returned by the pipeline. `Lane` contains attributes for the lane itself as well two `Line` objects for the left and right lane lines respectively.

The function `sliding_window()` , in sub-section [](submission-code.ipynb#Sliding-window-search) performs a sliding window search on the warped binary image. In brief, this function identifies the peaks in the histogram of the lower half of the binary image and assigns these peaks as the line bases. It then searches upwards window-by-window and records the positions of the pixels where it finds significant bright pixels. These pixel locations are then used to fit lines and derive the coefficients for the same.

Here's an example of the line-fitting by this method:

![Line Fit][image5]

The function `margin_search()` is similar to `sliding_window()` except that it searches in the vicinity of the previously found lines. This helps to speed up the line finding significantly given previous detections.

The other functions are `getCurvature()`, `getCenter()`, `getLaneWidth()` and `overlay()`. The first three functions are described in the section below and here I'll briefly describe `overlay()`. It takes the detected lane parameters, the inverse perspective matrix `Minv` , the original image and overlays the detected lane on the image.

Finally, the function `pipeline()` ties up these together to detect the lane and return the lane parameters. It  can be found in the sub-section [](submission-code.ipynb#The-lane-detection-pipeline) . The comments are pretty self-explanatory in the code, but I'll briefly go over the steps in the pipeline. I first get a warped image from the thresholded image and then perform a sliding window search on it. In the video version of the pipeline, it alternately runs the margin search using the previously detected parameters. Once the lane is detected, the parameters are used to calculate the curvature and the width of the lane as well the centrality of the vehicle in the lane.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The functions to calculate lane attributes are in the sub-section [](submission-code.ipynb#Lane-attributes-functions).

The function `getCurvature()` computes the curvature of the lane given the lane parameters. It computes the curvature individually for each lane line from their fit coefficients. The final curvature is the average of the two curvatures.

The functions `getCenter()` and `getLaneWidth()` use the last of the fit coefficients to compute the the centrality of the car in the lane and the lane width respectively. As the last coefficient of the fits is the bias of the line, it acts as the offset along the x-axis at which the line lies.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Below is the final image of the detection on one of the test images:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./projectvideo._outputmp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The detection pipeline works by thresholding the image on various transforms, namely gradients, their magnitude and direction as well the saturation of the colors. The parameters for these transforms are not well tuned and are likely to fail when the image undergoes lot of changes due to lighting/shadows and occlusions. 

The source and object points for the perspective transform are hard-coded, so too much variance from a test images will lead this transform to fail.

The sliding window search can fail if it cannot identify clear peaks in the histogram of the binary image. This can happen if there is too much clutter and when there is lack of contrast between the lane lines and its neighbouring regions. 

This project was pretty challenging and it could not dedicate more time exploring the various modifications and tweaks I have in mind. However, I do plan on implementing a few more ideas I have regarding this project. Firstly, I would like to see how well deep learning methods perform in this project. Since the sliding window search is essentially a classification problem of identifying good pixels for fitting the lane line, I will try to apply a deep learning model to this problem. An example of such an experiment can be found [here](https://medium.com/@heratypaul/experiment-using-deep-learning-to-find-lane-lines-c668a6e42070). Also, this project involves a regression problem in the form of estimating the fit for the next frame given the parameters of the current frame. Hopefully, this can be solved by a classical machine learning algorithm.

The speed of the detection for a video is too slow to be useful for real time performance. Hopefully, some of the modifications and some good optimization of the code, will lead to improved times.
