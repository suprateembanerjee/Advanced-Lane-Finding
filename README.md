## Advanced Lane Finding Project

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

[image1]: undistorted.PNG "Undistorted"
[image2]: thresholding.PNG "Thresholding"
[image3]: warp.PNG "Warp"
[image4]: poly.PNG "Poly"
[image5]: final.PNG "Result"
[video1]: test_videos_output/ProjectVideoOutput.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. 

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "./P2.ipynb" (or in lines # through # of the file called `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the distortion corrected chessboard images.

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image1]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at code block In[6] in P2.ipynb). I have experimented with a number of colour channels and gradients. Three of them were clearly better than the rest: (S(HSL) or Sobel X), (S(HSL) or Sobel X or R(RGB)), and ((S(HSL) or Sobel X) and L(HSL)). While the R channel was marginally better at annotating white lane lines it induced unwanted noise in some others. Upon further experimentation among these three threshold choices, I went with the third threshold option as it removed most noise while producing reasonably consistent outcomes. Here's an example of my output for this step. 

![alt text][image2]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in code block In[10] in P2.ipynb. The `warp()` function takes as inputs an image (`img`). I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32(
    [[680,445],
    [1059,665],
    [248,665],
    [590,445]])
    
dst = np.float32(
    [[850,60],
    [850,680],
    [450,680],
    [450,60]])
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 680, 445      | 850, 60       | 
| 1059, 665     | 850, 680      |
| 248, 665      | 480, 680      |
| 590, 445      | 450, 60       |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I fit a 2nd order polynomial using a function called 'fit_polynomial' in code block In[50] in P2.ipynb.

![alt text][image4]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in code block In[21] of P2.ipynb Notebook. The function 'measure_curvature_real' returns two values, radius of Curvature for the left line as well as radius of Cuvature for the right line. While displaying this curvature, I have averaged these two radii. In code block In[23], I have defined a function called 'calculate_lane_stats' which return the width of the lane as well as the midpoint. Using these values, I have calculated the position of the vehicle from the center of the lane.


#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

In code block In[26] of P2.ipynb, I implemented a function called 'warpback', which shifted the perspective of my annotated lane back onto the camera-view as viewed from the car. Stacking the annotated image over the original resulted in the desired output, as such: 

![alt text][image5]

---

### Pipeline (video)

Please find the output videos in the directory titled 'test_videos_output' in the same repository.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.

While the present version of the project does a decent job at finding the lanes in the project video, it fails for the harder challenge videos. Not only does the detections make no sense in the challenge video, the code fails to consistently detect similar lanes as in the project video. My assumption is, it has something to do with better thresholding, although I have not come across a better thresholding metric yet. I also feel the code is sensitive to shadows, although usage of Saturation and Lighness metrics has helped me stabilize this issue to a certain degree. If I were to revisit and refactor this project again in the future, I would be exploring avenues to increase tolerance to real world scenarios like uneven roads and shadows, hopefully having known a few more techniques that could help me, by then.
