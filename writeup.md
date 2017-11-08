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

[image1]: ./test_images/test1.jpg
[image2]: ./output_images/test1_undistorted.jpg
[image3]: ./output_images/test1_s_channel.jpg
[image4]: ./output_images/test1_thresholded.jpg
[image5]: ./output_images/test1_warped.jpg
[image6]: ./output_images/test1_sliding_window.jpg
[image7]: ./output_images/test1_pipeline.jpg
[video1]: ./project_video.mp4 "Video"

## Code location
All code is located at the IPython notebook `./notebook.ipynb`.

## Rubric Points

### Camera Calibration

#### Calculate camera matrix and distortion coefficients

The code for this step is contained in the code cell below the section `4. Camera Calibration`. Here is a brief description of how I did it:

1. Prepare object points like `(0, 0, 0)`, `(0, 1, 0)`, ..., `(0, 8, 0)`, `(1, 0, 0)`, ..., `(5, 8, 0)` (48 points) representing those inner lattices of the chessboard in the real world.
2. For each image for calibration, find the same lattices in each image by calling `cv2.findChessboardCorners()`.
3. Use `cv2.calibrateCamera()` to calculate the camera matrix and distance coefficients.

### Pipeline (single images)

#### 0. Image example
Throughout this section (Pipeline), I will use image `test1.jpg` as example for each step.

![test1.jpg][image1]

#### 1. Distortion correction

In the code cell below the section `5. Image Undistortion` I defined a function to distort any image produced by this camera, by using `cv2.undistort()` function. Below is an example of an original image and undistorted one.

![test1_undistorted.jpg][image2]

#### 2. Color transform

I only take the `S` channel in HLS color space. See the code cell below section `7. HLS Color Space`. Basically I use `cv2.cvtColor(image, cv2.COLOR_RGB2HLS)` to transform the image into HLS color space, and takes the `S` channel by `hls_image[:,:,-2]`. Below is an example of `test1_undistorted.jpg` transformed to `S` channel:

![test1_s_channel.jpg][image3]

#### 3. Thresholding

I use pixel value thresholding and a combination of direction and magnitude sobel thresholding to get a binary image, that is, a pixel is 1 if and only if `color_thresholded | (direction_thresholded & magnitude_thresholded)`. See the code cell below `9. Thresholding Lane Lines` for details. Below is an example of thresholded binary image:

![test1_thresholded.jpg][image4]

#### 4. Perspective transform

The 2 code cells below the section `11. Perspective Transform` does the perspective transform from the thresholded binary to a birds-eye view. Here are the steps I took:

1. Undistort the image `straight_lines1_undistorted.jpg`
2. Pick a trapezoid on the undistorted image aligning with the straight road lines.
3. Define a rectangle within the image shape.
4. Use `cv2.warpPerspective()` function to calculate a transform matrix (and its inverse)

Below is an example of transformed image:

![test1_warped.jpg][image5]

#### 5. Find Lane Lines Pixels

The code cells below `13. Find Lane Lines` section finds the lane lines. Here are the steps I took:

1. Define a `Line` class to represent one lane line, and `Lanes` class to represent the 2 lane lines (with some helper functions to perform actions on both lines at the same time).
2. When we are not confident about the lane lines on the previous frame (or when it is the first frame), we do a sliding window search. Basically we divide the area into `n` windows vertically, and search from the bottom. On the bottom window, we use a histogram to find the 2 lane lines, then searching upwards, each time we search the pixels near previous window.
3. When we are confident about the lane lines found on the previous frame, we search the pixels near the lanes lines found on the last frame
4. After either `2` or `3`, we get 2 lists of lane line pixels for left lane and right lane, then use `np.polyfit()` to fit a polynomial for each lane.

Below is an example of lane lines found by sliding window search:

![test1_sliding_window.jpg][image6]

#### 6. Curvature

Curvature is implemented in the `Lanes.get_curvature()` method (in `13.2 Line Class`).

#### 7. Vehicle position in the lane

Vehicle position is implemented in the `Lanes.get_position()` method (in `13.2 Line Class`). Basically I did 6 steps:

1. Calculate both lanes' horizontal position from the polynomials of them
2. Average them to get the lane center
3. Take vehicle center `1280 / 2 = 640`
4. Calculate vehicle position in pixel value by taking the difference between vehicle center and lane center
5. Calculate scale `m / pixel` by `lane width in meter / lane width in pixel`
6. Multiply lane position by scale.

Below is a picture of the output through the whole pipeline:

![test1_pipeline.jpg][image7]

---

### Pipeline (video)

#### Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### One issue I was facing was to threshold the image. There are too many options, we can use any color space, any combinations of the channels, sobel values on each axis, direction, magnitude, and any threshold values, also there are a lot of different ways to combine different thresholds. It was hard for me to find the right combination to filter out the lane lines just right. Also, perhaps that's just for this particular video. I think my threshold depends a lot on the color of the lane lines and the color of the road.

#### If there are other cars on the road, I think the thresholding let those objects pass through and therefore the sliding window search might not be able to correctly find two separate lanes.
