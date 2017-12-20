# **Advanced Lane Lines**

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

[image1]: ./output_images/undistort_test_image_calibration.png "Undistorted"
[image2]: ./output_images/undistort_test_image.png "Road Transformed"
[image3]: ./output_images/threshold.png "Binary Example"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/lane_detection.png "Fit Visual"
[image6]: ./output_images/view.png "Output"
[video1]: ./project_video.mp4 "Video"

## Rubric Points

Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/571/view) individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.   

You're reading it! and here is a link to my [project code](./P4.ipynb)

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first section of the IPython notebook located in "./P4.ipynb" (Camera calibration and distortion correction)

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

It is important to mention that the used chessboard images have 9x6 points to detect. This contrast with the image used during the lesson (8x6).

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps in the section 2 of the IPython notebook located in "./P4.ipynb").  Here's an example of my output for this step:

![alt text][image3]

Regarding color threshold, I did threshold of the S component of the HLS space (145, 255) for detecting lines and add robustness when shadows are present. I also used the H component (15, 105).

Regarding gradient threshold, I used gradient direction (0.6, 1.5 rad) combined with gradient over X axis (12, 90). In order to avoid the detection of dark lines (of both lane and road border) of the challenge video, I filtered the obtained result of gradient threshold by using the V component of the HSV space (145, 255).

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the section 3 of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose four source points contained in the lane lines of a selected test image. I selected these points to be represented at the top and bottom of the resulting image:

| Source        | Destination   |
|:-------------:|:-------------:|
| 206, 720      | 320, 720        |
| 1097, 720      | 960, 720      |
| 580, 460     | 320, 0      |
| 700, 460      | 960, 0        |

Here is an example of a warped image:

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial

The lane-line pixels were identified by using the sliding window method over the warped image (section 4 of notebook). Firstly, the histogram of the image was calculated to detect at which points of the X axis the concentration of lane points bigger is (base of lane line). Then I applied the sliding window taking the calculated lane line base as starting point to track the areas with more nonzero pixels.

After having detected the hot regions, I used the polyfit function to obtain two second grade polynomials that characterize both lane lines.

From the second and following frames, I avoided the sliding window method to detect the lines faster, by taking the result for the previous frame as starting point for the detection.

Here is an example of the detected lines and the polynomials (red pixels for left line and blue ones for right line):

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in the section 5 of the IPython notebook. Polynomials were built (coefficients stored in `left_fit_cr` and `right_fit_cr`) using the left and right lane positions (`leftx`,`lefty`,`rightx`,`righty`) calculated in the previous step and conversion factors for obtaining the calculation in meters (`ym_per_pix`, `xm_per_pix`). The obtained coefficients are used to evaluate the radius of curvature equation at the top of the Y axis (`y_eval`). The results are the radius of curvature for the left and right lanes (`left_curverad`, `right_curverad`).

Regarding the offset with respect to the center, I initially built polynomials without using the meter conversion (coefficients in `left_fitx` and `right_fitx`). I evaluated both polynomials in the middle of the Y axis and calculated the middle position between lines (`dist`). I calculated the difference between this value and the center of the X axis (640) and converted the difference to meters.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the section 6 of the IPython notebook. Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result](./output_videos/project_video.mp4)
And here's a [link to my challenge video result](./output_videos/challenge_video.mp4).

---

### Discussion


During the realization of the project it was clearly seen that the key for a good lane detection is an optimized application of color and gradient threshold to obtain the best possible binary image with the identified lines and as less as possible additional forms. Regarding color, as mentioned in the lesson the S component of the HLS representation does a good job in presence of shadows, so it was firstly used. The H component also was proved to obtain decent results, so it was also included to increase the robustness of the color threshold. Regarding gradient threshold through Sobel, gradient direction and gradient over X axis where selected, and, since this configuration made that dark lines on the road and road borders were detected (challenge video), an additional filter using the V component of the HSV space was used over the gradient results.

With the mentioned configuration the lines of both project and challenge videos are detected, except for some wobbly lines. I could see that I could make these wobbly lines disappear for the project video by using a tighter configuration of the threshold limits, but in this case there were more problems to detect the lines for the challenge video. Finally I took the current configuration, but maybe it is possible to find a better threshold limits.

Another possible improvement, and maybe this is an area where I could have put more efforts, is to perfect the sliding window method to detect the lines once the binary image is calculated.
