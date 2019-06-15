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

[image1]: ./output_images/camera_calibration.jpg "Undistorted"
[image2]: ./output_images/test1.jpg "Road Transformed"
[image3]: ./output_images/color_binary.jpg "Binary Example"
[image4]: ./output_images/warped_image0.jpg "Warp Example"
[image5]: ./output_images/find_lane.jpg "Fit Visual"
[image6]: ./output_images/test3.jpg "Output"
[video1]: ./output_images/project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Camera Calibration
#### 1. The camera matrix and distortion coefficients.

The code for this step is contained in the first and second code cell of the IPython notebook located in "./AdvancedLaneLines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. An example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Create a thresholded binary image using color transforms and gradients.

I used the HLS color space. The S channel is divided by the center value because it highlights the noticeable parts well. Then, I added the edge extraction using the L channel edge extraction that is not affected by the brightness. Edge extraction was set by narrowing down to a narrow range of low values because the wider the threshold, the larger the noise.(thresholding steps at lines 7 through 26 of 3rd code cell in `AdvancedLaneLines.ipynb`).  
Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

#### 3. Perspective transform.

The code for my perspective transform includes a function called `warped()`, which appears in the 4th code cell of the IPython notebook.  The `warped()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
width    = 0.143
height   = 0.628
offset_x = 200  # margin for curve road
imshape  = img.shape
img_size = (img.shape[1]+2*offset_x, img.shape[0])

# set polygon shape
apex_c     = [int(imshape[1]/2), int(imshape[0]*height)]
apex_w     = int(imshape[1]*width)
apex_bx    = int(imshape[1]*0.0)-offset_x
apex_by    = int(imshape[0]*0.0)

src        = np.float32([[apex_bx,imshape[0]-apex_by],
                        [apex_c[0]-apex_w/2, apex_c[1]], 
                        [apex_c[0]+apex_w/2, apex_c[1]],
                        [imshape[1]-apex_bx,imshape[0]-apex_by]])
dst        = np.float32([[0,imshape[0]],
                        [0, 0], 
                        [imshape[1]+2*offset_x,0],
                        [imshape[1]+2*offset_x,imshape[0]]])
```
The parameter `width` and `height` were determined by searching using the function `hist()` in the 5th code cell of the IPython notebook.
The parameter `offset_x` is used to expand the image. This is because the window is set to a large trapezoid so that the lane does not extend beyond the window even in the case of a curve.


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| -200,   720   |  0,    720    | 
| 548,    452   |  0,      0    |
| 731,    452   | 1680,    0    |
| 1480,   720   | 1680,  720    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]


#### 4. Identified lane-line pixels and fit their positions with a polynomial.

 I used the sliding window method to extract the lane marker pixels.
First, we created a histogram that integrates the number of points per binary image x-axis over the bottom half of the image. And I estimated the position of the center of the lane by selecting the largest value in the histogram. I processed the right lane and the left lane by dividing this process into the right side and the left side of the image.

 Starting from there, I set a window in the image and extracted the non-zero pixels in that window to detect the white line part.
In this process, hyperparameters are the window width and height and the number of windows on the Y axis, but it was tuned while looking at the detection results.
Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

#### 5. Calculated the radius of curvature of the lane and the position of the vehicle with respect to center.


The curvature and the position of the vehicle were calculated by simple polynomial approximation. the conversion parameters from pixel to real length are follows:
 Y dimension: 30/720 (meters per pixel)
 X dimension: 3.7/700 (meters per pixel)

I did this in lines 6 through 25 of 9th code cell in `AdvancedLaneLines.ipynb`


#### 6. Example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 1 through 41 of 11th cell in my code in `AdvancedLaneLines.ipynb` in the function `draw_detected_lane_boundaries()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Final video output.

My pipeline should perform reasonably well on the entire project video.
Here's a [link to my video result](./output_images/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues in the implementation of this project.

One of my ideas is that in the Perspective transform, I extended the window to the outside of the image and took the top of the lane widely. This is to prevent the top of the lane from escaping the window in the curve.
It works as intended, so there are no major issues. However, because the upper end of the lane will be extended greatly, the accuracy may be aggravated.

In this regard, it can be considered to use weights when fitting a sequence of points. The upper part of the lane is less accurate per pixel, so it might be better to reduce the contribution to the quadratic function fitting.

In addition, I think that it could be solved if the shape of the window I can be changed according to the curvature of the road. (For example, use the previous curvature to change the trapezoidal shape)
However, in this case, it is necessary to devise the correction since the image is distorted.

In order to make the output more robust, I think it is better to use the results of the previous frame.

The curvature and the position of the vehicle generally do not change rapidly, so the accuracy of the result of the previous frame may be increased by taking into account the movement of the vehicle.

Similarly in detection of lane pixels, overlapping the warped image with the previous frame may increase robustness. At that time, the previous frame needs to be translated and rotated in consideration of the movement of the vehicle.