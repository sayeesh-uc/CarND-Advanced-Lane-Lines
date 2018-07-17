## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./output_images/undist-caliberated.png "Undistorted Caliberated"
[image2]: ./output_images/road-undist.png "Road Transformed"
[image3]: ./output_images/threshold.png "Binary Example with direction and magnitude"
[image31]: ./output_images/threshold-lane.png "Binary Example with direction only"
[image4]: ./output_images/lane-perspective-transform.png "Warp Example"
[image5]: ./output_images/lane-find-using-window.png "Fit Visual"
[image51]: ./output_images/warped-lane-all.png "Warp Example"
[image6]: ./output_images/final-output.png "Output"
[image7]: ./output_images/lanefind-with-diag.png "video-Output-sample"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is in IPython notebook located in "./Calibrate Camera.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 
Here is the result
![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in `Color and Gradient Threshold.ipynb`.  Here's an example of my output for this step.

Binary Example with direction and magnitude
![alt text][image3]

Binary Example with direction only
![alt text][image31]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in the file `Perspective Transform.ipynb` (7th code cell of the IPython notebook).
In the sample, I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([[580, 460], 
                  [700, 460], 
                  [1100, img_size[0]],
                  [210, img_size[0]]])
dst = np.float32(
    [[offset, 0],
     [img_size[0]-offset, 0], 
     [img_size[0]-offset, img_size[1]], 
     [offset, img_size[1]]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 580, 460      | 250, 0        | 
| 700, 460      | 470, 0      |
| 1100, 720     | 470, 1280      |
| 210, 7200      | 250, 1280        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I took the warped binary image like below

![alt text][image51]

Ran sliding window search(with 9 windows) looking for spikes in white spots using histogram.
The positions/windows where nonzero values are found are stored in a variable(for both x and y values), 
A margin is used while searching in the next window.
if we find more white pixes next window, then we will recenter the lane markers appropriately.
the final values are passed to np.polyfit setting the polynomial to 2nd order.

![alt text][image5]
I did this in lines in cell block of 10 and 12 in  `Lane Find.ipynb`

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines in cell block of 14 in  `Lane Find.ipynb`
convert the x,y points from the image grid to real world grid using the ym_per_pix and xm_per_pix values given.
we get the real world grid by multiplying the x,y points with ym_per_pix and xm_per_pix.

We will np.polyfit using the new points with a polynomial of 2nd order.
with the given coefficients of 2nd order polinomial, we can find the radius/curvature.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in cell block of 15 and 16 in  `Lane Find.ipynb`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./video_with_lanes.mp4)
Here's a [link to my video result with all diagnostics](./video with diagnostics.mp4)

[This is the youtube link](https://www.youtube.com/watch?v=W42CfISDhnk)

sample image below:

![alt text][image7]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I discarded the magnitude and direction thresholds, as i was able to get better output with just x and y sobels.
I used the apporach of finding lanes by searching within the margins of the previous found locations for the new lane points in the next frame to reduce the processing time.
I applied weighted average on the lane locations with higher weight given for new frames.
If we haven't found lane in any frame, the weighted factor will be increased so that when a new frame with lane is found, it gets higher preference and not get bogged down with lane locations from older frames.
