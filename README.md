## Advanced Lane Finding Project
---

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

[image1]: ./output_images/undistort-1.png "Undistorted"
[image2]: ./output_images/undistort.png "Road Transformed"
[image3]: ./output_images/threshold_binary.png "Threshold Binary"
[image4]: ./output_images/checkTranfsform.png "Checking Transform"
[image5]: ./output_images/detect_lane_lines_1.png "Fit Visual"
[image6]: ./output_images/detect_lane_lines2.png "Fit Visual"
[image7]: ./output_images/output_metrics.png "Output"
[video1]: ./output_images/project_video.mp4 "Video"
[video1]: ./output_images/challenge_video.mp4 "Challenge Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  
You're reading it!

### Camera Calibration

The code for this step is contained in the function called 'calibrate' in the  `Utility functions` code cell of the IPython notebook located in "./mywork/project4.ipynb"   

First I have object points (x,y,z) in the three dimensional world, where z=0, as I assumed the chessboard is on a fixed plane.

For each image, when I successfully detect corners, I collect the `objp` array into the `objpoints`. The detected corners are collected in the `imgpoints`. I use the `cv2.calibrateCamera` to compute the camera calibration and distortion coefficients.  

You can see below the result of running the `cv2.undistort()` function on this image: 

![alt text][image1]

#### 1. an example of a distortion-corrected image.

In order to see try the above distortion coefficients, see the road image below:
![alt text][image2]

#### 2. Description of how color and gradient thresholds were used to create a thresholded binary image

I used a combination of color and gradient thresholds to generate a binary image , see function `binary_image_transform` in the code cell called `Utility functions` located in "./mywork/project4.ipynb"

Here's an example of my output for this step.

![alt text][image3]

#### 3. Perspective transform and an example

The code for my perspective transform includes a function called `unwarp`, which you can see in the code cell called `Utility functions` located in "./mywork/project4.ipynb"

`unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the following hardcoded source and destination points:

```python
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 585, 460      | 320, 0        | 
| 203, 720      | 320, 720      |
| 1126, 720     | 960, 720      |
| 695, 460      | 960, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Identification of lane-line pixels polynomial fit

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:


See function `detect_lane_lines` uses sliding window to detect the lane lines, then fit them using a 2nd order polynomial, see output result:

![alt text][image5]

When I already have detected lane lines, I use `detect_lane_lines_subsequent_images` to look into the previously detected area, it looks like this:

![alt text][image6]

#### 5. Radius curve calculation curvature of the lane and the position of the vehicle with respect to center

I did this in function `curvature`

#### 6. An example image of result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in function `warp_detected_lines_onto_original`.  Here is an example of my result on a test image:

![alt text][image7]

---

### Pipeline (video)

Here's a [video1]

---

### Discussion

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and 
how I might improve it if I were going to pursue this project further.
  
I do the following steps in my pipeline:

* I keep a window size n of last known calculation, which I average out to generate the lane lines
* Every time I read an image, I first run through a sanity-check, see `sanity_check_update_lines`, that will filter out lines that are too different in curvature, are too far apart, or are not roughly parallel
* If I have already detected lane lines before, I do not need to search the whole image, but I can search in the round about area of the previous lane lines.
* I can keep searching in the prev lane line area, but when there are too many losses of detecting lane lines I reset back to searching from scratch.

My algorithm works fine on the provided video, it is still not quick to respond to the changing lane curvature. I tried my algo on the challenge videos, the output is not great, it got tricked by the shadow of the bridge.  My sanity check needs improvement, currently it is not doing a good enough job of filtering out.

