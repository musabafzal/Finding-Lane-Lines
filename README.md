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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test5.jpg
[image7]: ./test_images/undistorted.jpg
[image8]: ./test_images/binary.jpg
[image9]: ./test_images/src.jpg
[image10]: ./test_images/dst.png
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./test_images/detected.png "Fit Visual"
[image6]: ./test_images/result.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "lane_lines_notebook.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will show you the images of distorted image and distortion-corrected image as follows:

Distorted image:

![alt text][image2]

Distortion-corrected image:

![alt text][image7]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. I worked in HLS space for color thresholding. Only S channel was used. I used the s channel for a gradient filter along x and saturation threshold. The code for these filters is present in the function `pipeline` present in the third code cell of `lane_lines_notebook.ipynb`. The binarized version of the image above looks as follows:

![alt text][image8]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in the fourth code cell of `lane_lines_notebook.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([(205, 720), (1075,720),
      (704,454), (576,454)])

dst = np.float32([[offset, 0],
      [img_size[0]-offset, 0],
      [img_size[0]-offset, img_size[1]],
      [offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 576, 454      | 150, 0        |
| 205, 720      | 150, 720      |
| 1075, 720     | 1130, 720      |
| 704, 454      | 1130, 0        |

Example:

Undistorted image with source points drawn:

![alt text][image9]

Warped image:

![alt text][image10]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I implemented both the codes, firstly the sliding window search code will run and detect the left and right lane line pixels, in the function `window_search`.

Then there is code, in the function `detect_lane_lines`, which just searches for lane line pixels within a margin of 100 pixels from the lane lines that were detected by the sliding window search.

The code for these functions appears in the fifth code cell of `lane_lines_notebook.ipynb`.

The looks like this:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature and the position of the vehicle with respect to center is calculated upon calling the `radius_curvature` in sixth code cell `lane_lines_notebook.ipynb`.
For a second order polynomial f(y)=A y^2 +B y + C the radius of curvature is given by R = [(1+(2 Ay +B)^2 )^3/2]/|2A|.

The distance from the center of the lane is calculated by measuring the distance to each lane and calculating the position assuming the lane has a given fixed width of 3.7m.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in seventh code cell of `lane_lines_notebook.ipynb` in the function `result_image()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I faced problem implementing the sliding window search in the first frame of the video and then margin search in the subsequent frames but was able to resolve it using static variables. The pipeline was having difficulty detecting lane line where there were shadows and also yellow lane lines.  The pipeline will likely fail as more and more false lines detected due to shadows are on the same lane. This could be solved by building a robust lane detector which takes into account both yellow lane lines and white lane lines and can distinguish between them.
