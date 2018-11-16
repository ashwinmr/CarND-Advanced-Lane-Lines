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

[image1]: ./output_images/undistorted_output.png "Undistorted output"
[image2]: ./output_images/road_transformed.png "Road Transformed"
[image3]: ./output_images/binary_combo_example.png "Binary Example"
[image4]: ./output_images/warped_straight_lines.png "Warp Example"
[image5]: ./output_images/color_fit_lines.png "Fit Visual"
[image6]: ./output_images/final_output.png "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in code cell number 2 in the IPython notebook located in "./Main.ipynb"

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I created a function that would do this for each image and then called this function on every camera calibration image and accumulated all object and image points.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image in code cell number 4.  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform is in code cell 5. I hard coded the source and destination points by observing the test images. Then I use the "cv2.getPerspectiveTransform" function to get the transformation and apply it using "cv2.warpPerspective" function.

The source and destination points I chose were:

```python
src = np.array(
    [[(220,height),
      (595,450),
      (690,450),
      (1110,height)]],dtype=np.int32)

dst = np.array(
    [[(src[0][0][0],height),
      (src[0][0][0],0),
      (src[0][3][0],0),
      (src[0][3][0],height)]],dtype=np.int32)

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
|  220, 720     |  220, 720     | 
|  595, 450     |  220, 0       |
|  690, 450     | 1110, 0       |
| 1110, 720     | 1110, 720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this is in cell 6. I created a function that would use a histogram and windowing to get all the points corresponding to the lane and then fit a second order polynomial to it.

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I use the bottom pixels of the fitted curve and the equation to calculate radius of curvature. This is after, transforming from pixel space to world space. The code can be found in cell 7.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in code cell 8. I use the inverse perspective transformation matrix to transform the fitted curve back to the road. I then use the "fillPoly" function to draw it on the road.
I put it all together with a text display in code cell 9.

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I put everything together in a pipeline to process the video. It did reasonably well on the project video without the need for a class to track previous frames or other measures.

Here's a [link to my video result](./output_videos/project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I did not face many issues on the project video. My pipeline will fail on the challenge and harder challenge videos because there is no implementation of smoothing or filtering using past video frames. There is also no adaptation based on lighting changes. It also fails when a second order polynomial is not enough to match snaky roads. It is also very processor intensive since I did not make use of the "search around poly" algorithm. I could add filtering and optimization to make it more robust using a class to store prior information.