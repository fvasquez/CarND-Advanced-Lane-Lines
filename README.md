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

[image1]: ./camera_cal/calibration1.jpg "Original"
[image2]: ./output_camera_cal/undist_calibration1.jpg "Undistorted"
[image3]: ./test_images/test5.jpg "Original"
[image4]: ./output_images/undist_test5.jpg "Undistorted"
[image5]: ./output_images/mono_test5.jpg "Binary"
[image6]: ./output_images/mono_trapezoid.jpg "Source Points"
[image7]: ./output_images/mono_rectangle.jpg "Destination Points"
[image8]: ./output_images/windows_test5.jpg "Fit Windows"
[image9]: ./output_images/test5.jpg "Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration

#### 1. Have the camera matrix and distortion coefﬁcients been computed correctly and checked on one of the calibration images as a test?

The code for this step is contained code cell [2] of the IPython notebook located in "./P2.ipynb".  

I start by preparing "object points" which are the (x, y, z) coordinates of the chessboard corners in the world. These object points are the same for every calibration image. For every calibration image I use `cv2.findChessboardCorners()` to detect all chessboard corners in a test image.  `imgpoints` are appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  I then used `objpoints` and `imgpoints` to compute the camera calibration and distortion coefﬁcients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the calibration images using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]
![alt text][image2]

### Pipeline (single images)

#### 1. Has the distortion correction been correctly applied to each image?

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image3]

The code for this step is contained code cell [3] of the IPython notebook.
I applied the distortion correction from code cell [2] to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image4]

While the original and undistorted images may seem identical at first glance if you look closely at the white car near the right edge of the camera frame you can see that the output image has been corrected.

#### 2. Has a binary image been created using color transforms, gradients or other methods?

I used a combination of color and gradient thresholds to generate a binary image in code cell [4].  Here's an example of my output for this step.

![alt text][image5]

#### 3. Has a perspective transform been applied to rectify the image?

The code for my perspective transform includes `get_trapezoid()` and `get_rectangle()` functions from code cells [5] and [6] of the IPython notebook.  Both functions take an image shape (`shape`) as input.  `get_trapezoid()` generates the source (`src`) points `get_rectangle()` generates the destination (`dst`) points.  I then compute the perspective transform using `cv2.getPerspectiveTransform()`.

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a binary image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image6]
![alt text][image7]

#### 4. Have lane line pixels been identiﬁed in the rectiﬁed image and ﬁt with a polynomial?

I used the sliding window technique described in the lecture to fit my lane lines with a 2nd order polynomial like this:

![alt text][image8]

I defined a `fit_polynomial()` function in code cell [8] of the IPython notebook.  This function takes a bird's eye binary image as input and calls `find_lane_pixels()` to select the pixels for left and right lanes.  The hyperparameters of `nwindows`, `nmargin` and `minpix` can be passed in as arguments to the `find_lane_pixels()` function. The resulting left and right lane pixels are then fitted to separate polynomials using `np.polyfit()`.

#### 5. Having identiﬁed the lane lines, has the radius of curvature of the road been estimated? And the position of the vehicle with respect to center in the lane?

I calculated the radius of curvature and position of the vehicle with respect the center of the lane in code cell [9] of the IPython notebook. I used a `ym_per_pix` value of `27./720` and an `xm_per_pix` value of `3.7/900` based on the shape and resolution of the test images as well as information gleaned from the lecture notes. I observed by scrubbing through the output video that the radius of curvature shrinks when the vehicle enters a turn and grows when it gets on a straightaway.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell [10] of the IPython notebook by composing a `process_image()` function.  The reverse transform form bird's eye view back to perspective view happens in the `draw_lane()` function with a call to `cv2.warpPerspective()` using `Minv` from code cell [6]. I used masking and `cv2.addWeighted()` to make the lane lines and inner lane stand out.  Here is an example of my result on a test image:

![alt text][image9]

---

### Pipeline (video)

#### 1. Does the pipeline established with the test images work to process the video?

Here's a [link to my video result](./output_video.mp4)

I noticed that Firefox does not recognize the video format of this output `.mp4` but I can still open and play the video after I download it to my Windows machine.

---

### Discussion

#### 1. Where will your pipeline likely fail?  What could you do to make it more robust?

My implementation searches blindly for lane lines each frame of the video which is grossly inefficient.  A smarter approach would be to use results from previous frames to inform the search for the position of the lines in subsequent frames of video.  My lane lines seem to shake when the vehicle drives on overpasses with much lighter pavement.  Applying a low pass filter over successive frames would probably smooth that out.
