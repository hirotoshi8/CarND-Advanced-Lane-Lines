# Writeup
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

[image1]: ./output_images/camera_calibration_undistorted_image.jpg "Undistorted"
[image2]: ./output_images/undistorted_image.jpg "Road Transformed"
[image3]: ./output_images/feature_extracted_image.jpg "Binary Example"
[image4]: ./output_images/binary_image.jpg "Binary Example"
[image5]: ./output_images/warped_image.jpg "Warp Example"
[image6]: ./output_images/sliding_window_approach_image.jpg "Fit Visual"
[image7]: ./output_images/ROI_image.jpg "Fit Visual"
[image8]: ./output_images/pipeline_result_image.jpg "Output"
[video1]: ./output_images/project.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project as a guide and a starting point.  

### Camera Calibration

I briefly state how I computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the `Camera Calibration` cell of the IPython notebook called `P4_Camera_Calibration.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]


## Pipeline (single images)

#### 1. Distortion-corrected
I provide an example of a distortion-corrected image.
Based on the result of above Camera calibration, I correct the distorted images.
To demonstrate this step, I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Create binary of image
I describe how I tried color transforms, gradients or other methods to create a thresholded binary image. This step code is contained in `Sample: plot the example image with each features` in `P4.ipynb`. I provide an example of some binary image result. In this analysis, HLS color space (especially `S` channel) and Gradient X direction is good features to detect lane line.

![alt text][image3]


As a result, to make to binary image, I used a combination of color (`S` and `L` channel in HLS color space) and gradient X thresholds to generate a binary image (thresholding steps in `Extract a binary image` cell in `P4.ipynb`).  Here's an example of my output for this step. In the pipeline image below, the blue color is result from `S` color channel of HLS and the green one is gradient X.

![alt text][image4]

| Features                     | Thretholds             | 
|:----------------------------:|:----------------------:| 
| S channel in HSV Color space | Lower threthold = 200  |
|                              | Higher threthold = 255 |
| L channel in HSV Color space | Lower threthold = 230  |
|                              | Higher threthold = 255 |
| Gradient X                   | Lower threthold = 200  |
|                              | Higher threthold = 255 |


#### 3. Perspective transfomr
I describe how I performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in `Helper Functions` cell in the `P4.ipynb`. 

The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:


```python
src = np.float32(
    [[(img_size[0]  / 2) - 60, img_size[1] / 2 + 100],
    [((img_size[0]  / 4) - 120), img_size[1]-1],
    [(img_size[0]*3 / 4) + 150,  img_size[1]-1],
    [(img_size[0]   / 2  + 60), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0]  / 4 - 120), 0],
    [(img_size[0]   / 4 - 120), img_size[1]-1],
    [(img_size[0]*3 / 4 + 150), img_size[1]-1],
    [(img_size[0]*3 / 4 + 150), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 200, 0        | 
| 200, 719      | 200, 720      |
| 1110, 719     | 1110, 720     |
| 700, 460      | 1110, 0       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image5]


#### 4. Find the lane line
I demonstrate to identify lane-line pixels and fit their positions with sliding windows approach following the udacity's lecture

![alt text][image6]

Based on the position of lane lines detected by sliding window approach, 
I fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image7]

#### 5. Calculate the curavature of the lane line
I calculate the the radius of curvature of the lane following udacity's lecture and the position of the vehicle with respect to center.

I did this in `Calculate the curvature of the road and lane line` in `P4.ipynb`.

#### 6. Result of my pipeline for a test image
I provide an example image of my result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in `Build a Lane Finding Pipeline` section in `P4.ipynb`.  Here is an example of my result on a test image:

![alt text][image8]

---

### Pipeline for video stream

#### 1. Advanced lane line with my pipeline
I provide a link to my final video output.  My pipeline can perform reasonably well on the entire project video.

Here's a [link to my video result](./output_images/project.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I tried the `Challenge` video stream and my result is collappsed! 
In this project, I used HLS color space and gradient X and it's still not enough to only detect the lane lines. This result seems to be affected by the shadows of bridge in challeng video.
Addition to that, judging from the plot in `Sample: plot the example image with each features` cell in `P4.ipynb`, there are false-positive detections due to the shadows of trees. This is why HLS color space may not be the best robust choise.

To be robust and remove these noise, it's neccessary to adapt some additional logic.

- Change color space
- Add time sequential filter like moving average filter
 (In this project, I use only simple low pass filter).

