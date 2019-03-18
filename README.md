## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

The code I used for this project can be found in `Udacity_Project4_Submission.ipynb`. The cell numbers referenced here is from that file.

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

[image1]: ./output_images/undistort_chessboard.png "Undistorted"
[image2a]: ./test_images/test4.jpg "Road Transformed"
[image2b]: ./test_images/straight_lines1.jpg "Road Transformed"
[image3a]: ./output_images/combine_gradient.png "Binary Example1"
[image3b]: ./output_images/color_spaces.png "Binary Example2"
[image3c]: ./output_images/final_combined.png "Binary Example3"
[image4a]: ./output_images/perspective_straight.png "Warp Example1"
[image4b]: ./output_images/perspective_curved.png "Warp Example2"
[image5a]: ./output_images/lane_lines_window.png "Lane line fit"
[image5b]: ./output_images/histogram.png "Histogram"
[image5c]: ./output_images/polyfit_prev.png "Lane line fit using previous fit"
[image6a]: ./output_images/drawing.png "Output1"
[image6b]: ./output_images/drawwrite.png "Output12"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

#### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first four code cells.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. A number of chessboard images are provided. It is assumed that the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. OpenCV functions `findChessboardCorners` is used here.

I then created function `cal_undistort(img, objpoints, imgpoints)` used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to two test images as following:
![alt text][image2a]

This image 'test4' is choosen because image with curved lines, tree shadows and yellow lines, most complicated one).


![alt text][image2b]

This image 'straight_line1' is choosen because stright line helps determine destination point and source point later for perspective transform, and yellow lines is where I have problem detecting in Project 1

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at 'Step 2: Appky Binary Thresholds' part in `Udacity_Project4_Submission.ipynb`.  

* Gradient thresholds: `combine_thresh()` combines sobel x or y threshold (`abs_sobel_thresh()`), magnitude thresholds (`mag_thresh()`) and direction thresholds(`dir_threshold()`)
* Color space threshold: defined in `color_thresh()`, used RGB and HLS color spaces.
* Combined color and gradient: defined in `color_grad_thresh()`. By observation a lane should either be yellowish or whiteish and it should also have gradient.

Here's series of images shows the results on the test images I choose:


![alt text][image3a]

![alt text][image3b]

![alt text][image3c]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in 'Step 3: Perspective Transform' in `Udacity_Project4_Submission.ipynb`.  The `corners_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32([[565, 470],[722, 470], [1120, 720],[190, 720]])
```

```
dst = np.float32([[320, 1], [920, 1], [920, 720],[320, 720]])
```

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image on both straight lines and curved lines.

![alt text][image4a]
![alt text][image4b]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

This part can be found in 'Step 4: Finding Lanes' in `Udacity_Project4_Submission.ipynb`. The functions `sliding_window()` and `polyfit_using_prev`, which identify lane lines and fit a second order polynomial to both right and left lane lines. `sliding_window()` computes a histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. These locations were identified from the local maxima of the left and right halves of the histogram. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

![alt text][image5a]

The image below depicts the histogram generated by `sliding_window()`. The resulting base points for the left and right lanes - the two peaks nearest the center - are clearly visible:

![alt text][image5b]

The `polyfit_using_prev` function performs basically the same task, but only searching for lane pixels within a certain range of the previous fit. The image below demonstrates this

![alt text][image5c]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This can be found in 'Step 5: Measuring Curvature' in `Udacity_Project4_Submission.ipynb`. The detailed function can be found in `meaure_curvature_center()`. An example result is provided below:

'Radius of curvature for example: 1321.01990855 m, 1026.86607422 m
Distance from lane center for example: 0.133339362526 m'

The curvature of 1.0 km and 1.3 km is close to the 1 km mentioned in the course material


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the code cells titled "Drawing" and "Write Curvature radius and distance from center data to img" in `Udacity_Project4_Submission.ipynb`. A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results of the `drawing` function:


I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6a]

Below is an example of the results of the `write_data` function, which writes text identifying the curvature radius and vehicle position data onto the original image:

![alt text][image6b]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The major problem I encountered are due to lighting conditions, shadows, discoloration, missing lanes and so on. To summaries, these are problems that causing the lines hard to detect. It is easy to using binary threshold to get clear lane lines of the 'project_vedio.mp4'. However, the images in 'challenge_vedio.mp4' are much harder. There continuous cracks on the road, the shadows from the bridge all made this problem harder.

I think the main approach for this problem is to find a better threshold parameters to make the lines clearly visible first.

