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
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"


[image_a1]: ./examples/orig_ud_persp.png "Orig/Undistorted/Warped 1"
[image_a2]: ./examples/orig_ud_persp2.png "Orig/Undistorted/Warped 2"
[image_a3]: ./examples/orig_ud_persp3.png "Orig/Undistorted/Warped 3"

[image_b1]: ./examples/wrp_filt_hist.png "Warped/Filtered/Histogram 1"
[image_b2]: ./examples/wrp_filt_hist2.png "Warped/Filtered/Histogram 2"
[image_b3]: ./examples/wrp_filt_hist3.png "Warped/Filtered/Histogram 3"

[image_c1]: ./examples/filt_windows.png "Filtered/Windows 1"
[image_c2]: ./examples/filt_windows2.png "Filtered/Windows 2"
[image_c3]: ./examples/filt_windows3.png "Filtered/Windows 3"

[image_d1]: ./examples/result.png "Result 1"
[image_d2]: ./examples/result2.png "Result 2"
[image_d3]: ./examples/result3.png "Result 3"

[video1]: ./project_video_out.mp4 "Result Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook located in [P4.ipynb](https://github.com/cscsatho/CarND-Advanced-Lane-Lines-P4/blob/master/P4.ipynb)  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image_a1]
![alt text][image_a2]
![alt text][image_a3]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like the ones in the previous paragraph.

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (can be seen in the jupyter notebook, cell 4, function filter() of class LaneDetector).  Here's an example of my output for this step.
```
gradx = self.filtering.abs_sobel_thresh(gray, orient='x', sobel_kernel=3, thresh=(30, 100), title='gradx')
mag_binary = self.filtering.mag_thresh(gray, sobel_kernel=3, mag_thresh=(30, 100), title='mag_binary')
s_binary = self.filtering.get_color_channel(hls, chn=2, thresh=(100, 255), title='s_binary')
mag_binary_l = self.filtering.mag_thresh(hls[:,:,1], sobel_kernel=13, mag_thresh=(30, 125), title='mag_binary_l')
mag_binary_s = self.filtering.mag_thresh(hls[:,:,2], sobel_kernel=13, mag_thresh=(30, 125), title='mag_binary_s')
dir_binary_s = self.filtering.dir_threshold(hls[:,:,2], sobel_kernel=5, thresh=(0.7, 1.3), title='dir_binary_s')
        
img_comb[(((gradx == 1) | (s_binary == 1) | (mag_binary == 1) & (dir_binary_s == 1)) & ((mag_binary_l == 1) | (mag_binary_s == 1)))] = 1        
```

However, I!ve implement a lot more filters that these six and experimented with them (see in code).

![alt text][image_b1]
![alt text][image_b2]
![alt text][image_b3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in the 4th cell of the notebook (class LineDetector).  The function uses the already created transformation matrix M.I chose the hardcode the source and destination points in the following manner:

```
src_ref_points = [[537, 490], [742, 490], [1087, 720], [193, 720]]
dst_ref_points = [[436, 350], [844, 350], [850, 720], [430, 720]]
```

I verified that my perspective transform was working as expected by checking landmarks on the `src` and `dst` image.

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image_c1]
![alt text][image_c2]
![alt text][image_c3]

I used the sliding window technique with 10 vertical shifts. The higlightLaneLines() function along with the windowSearch() is responsible for rendering the example images above.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in higlightLaneLines() using np.polyfit() on filtered window pixels.
Example radius calculations:
```
Processing test_images\straight_lines2.jpg...
 left_fit [  2.20364962e-05  -1.20024652e-02   4.39384579e+02] [  6.16315063e-05  -1.86491133e-03   3.79280085e+00]
right_fit [  2.20364962e-05  -1.20024652e-02   4.39384579e+02] [  6.16315063e-05  -1.86491133e-03   3.79280085e+00]
radius r/l/avg =  8112/36440/22276 m, lane width = 3.59 m, offset in lane =  -8 cm
Processing test_images\test1.jpg...
 left_fit [  6.86606667e-05  -1.02159914e-01   4.88846894e+02] [  1.92029634e-04  -1.58733376e-02   4.21976328e+00]
right_fit [  6.86606667e-05  -1.02159914e-01   4.88846894e+02] [  1.92029634e-04  -1.58733376e-02   4.21976328e+00]
radius r/l/avg =  2603/ 1858/ 2230 m, lane width = 3.81 m, offset in lane = -27 cm
Processing test_images\test2.jpg...
 left_fit [ -1.67542028e-04   2.16592025e-01   4.03442102e+02] [ -4.68580279e-04   3.36534967e-02   3.48254267e+00]
right_fit [ -1.67542028e-04   2.16592025e-01   4.03442102e+02] [ -4.68580279e-04   3.36534967e-02   3.48254267e+00]
radius r/l/avg =  1067/ 1383/ 1225 m, lane width = 3.68 m, offset in lane = -39 cm
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

Lane area is marked in function addOverlay() using cv2.fillPoly() and other cv2 image manipulation functions. Here is an example of my result on a test image:

![alt text][image_d1]
![alt text][image_d2]
![alt text][image_d3]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_out.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

There are minor problems when no suitable curve was found - interpolating could be improved.
Filtering dark and very bright areas could be improved as well.

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  
