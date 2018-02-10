## Writeup

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

[image1]: ./examples/calibration17_undistorted.png "Undistorted"
[image2]: ./examples/straight_lines2_undist.png "Road Transformed"
[image3]: ./examples/challenge_video_37_thresholding.png "Binary Example"
[image4]: ./examples/straight_lines2_binary_warped.png "Warp Example"
[image7]: ./examples/straight_lines2_warp_matrix_points.jpg "Points Example"
[image5]: ./examples/straight_lines2_fit.png "Fit Example"
[image6]: ./examples/straight_lines2_lanes.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 4th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb).

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the 5th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb). I've applied the distortion correction to one of the test images below:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the 8th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb).
I used a combination of color and gradient thresholds to generate a binary image. The first was a combination of thesholding of specific channels in different color spaces:
- The Red channel in RGB color space
- The Value channel in HSV color space
- A blend of all channels in HSV color space
- The Saturation channel in HLS color space

In ideal lighting and road conditions, this thresholding was very good at identifying lane lines. However, it suffered from identifying excess, non-lane information.

To alleviate this, I combined it with an x-gradient threshold which, by itself, was good at finding lines. But alone suffered by identifying shadows and tar strips. The two together worked generally well at identifying most lines, especially white lane markings.

Through experimentation I found that the lightness channel in HLS color space was incredibly robust at identifying yellow lane markings. By taking the magnitude gradient of this channel and combining it with a directional gradient I came up with an impressive yellow lane finder.

By combining both, I was able to generate a binary output that excelled in identifying yellow and white lane lines under a variety of conditions.

Here's an example of my output for this step on a particularly difficult frame from the challenge video:

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the 11th code cell of the IPython notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

![alt text][image7]

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 714, 466      | 960, 0        |
| 1107, 720     | 960, 720      |
| 219, 720      | 320, 720      |
| 573, 466      | 320, 0        | 

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the 13th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb).
First I took a histogram of the lower half of the columns in the binary image. The peaks of this histogram represent the general area of the left and right lanes. I then implemented a sliding window search which travels from the bottom of the image upward. In each successive step, the new window searches a predefined area centered over the midpoint of the nonzero pixels in the previous window. I found that making the first window wider than the rest helped guard against the window accidentally merging with non-lane markings.

For each cluster of pixels identified as a lane, I fit a 2nd order polynomial to the line as seen below:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the 15th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb).
I first took the curve fit found in the previous step and converted it from pixels to meters. I then used the new polynomials and applied them to the following equation found [here](https://www.intmath.com/applications-differentiation/8-radius-curvature.php):

R = (1 + (2Ay + B)^2)^(3/2) / |2A|

The position of the vehicle was calculated using the follow equation:

C = Image_Center - (X_Position_L_Lane + X_Mid_Between_Lanes)


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the 17th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb). Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my project video result](./project_video.mp4)

And here's [my challenge video result](./challenge_video.mp4)

And lastly, [my harder challenge video result](./harder_challenge_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Most issues encountered for this project revolved around an inibility to identify lane markings in all environmental and road conditions. I spent a significant amount of time experimenting with different color spaces and thresholding techniques to get it tuned as well as I could. As I tested, I put together a selection of frames which captured a wide range of road conditions. As I experiemented, checked that each change did not have a negative imact on any of these frames.

Once I had made the thresholding as robust as possible, I utilized a running average of the lane information for the last 7 frames to help smooth out the jumps between frames. Additionally, I included plasubility checks on the lanes to filter out lanes which had unrealistic features. This included:
- Checking for large jumps in the base position of the lane
- Comparing the distance between the lanes
- Checking the positions of the lanes
- Checking for large changes in the curvature of the lane
- Comparing the curvature of the left and right lanes
If the lanes did not pass the plausibility check, then that frame was tossed out. If no lanes were detected for 7 frames then I considered the lane "lost" and did not show anything.

The code for this step is contained in the 19th code cell of the IPython notebook located [here](./Advanced_Lane_Lines.ipynb). 

One shortcoming of my pipeline is with the thresholding. More robust thresholding techniques would likely improve the lane detection under dynamic lighting and poor road conditions. Another is a limitation of the sliding window search. By limiting the lateral movement of the window, sharp turns will not be detected. If information about the steering angle was known, then a dynamic parameter could be used to improve the lateral search tolerance.

Another failure mode would be for onramps, offramps, and construction zones. By adding in map data along with a preplanned navigation and GPS, we could know if an exit needed to to taken or ignored, and know to ignore onramps alltogether. Construction zones could be detected using traffic sign recognition, which could temporarily disable the system while in that zone (if needed).