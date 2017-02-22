
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
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

###Camera Calibration

####1. Camera calibration is the process of estimating intrinsic and/or extrinsic parameters. Intrinsic parameters deal with the camera's internal characteristics, such as, its focal length, skew, distortion, and image center. Extrinsic parameters describe its position and orientation in the world. The basic method in calculating the Camera calibration is to capture 
known pattern like chessboard grid  and comparing with captured images from multiple anges. OpenCV libray provides cv2.calibrateCamera API for this purpose.

The code for this step is contained  in lines #10 through #30 of the file called `image_process.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

Following are various stages in our pipeline(from #1 to #6) where input image is processed sequentially.
The implementation of pipeline can be found in `advanced_lanefind.py` from line #248 to #270. 


####1. The first step is to undistort the input image to pipeline. Image distortion occurs when a camera looks at 3D objects in the real world and transforms them into a 2D image; this transformation isn’t perfect and changes the shape and size of the object. With camera calibration, we have calculated distortion cofficients for our camera. I have  used OpenCV cv2.undistort() API to undistort the image.

The code for this step is contained  at lines #252 in `advanced_lanefind.py`. 

Following is the example of a distortion-corrected image from test set,

![alt text][image2]

####2. Next the image is processed to extract key features that can identify lanes clearly and removing unnecessary parts of the image using  color transforms, gradients  to create a thresholded binary image.  
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at lines #260 through #270 in `advanced_lanefind.py`). 

Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]

####3. Due to 2D nature of the image, objects appear smaller the farther away and bigger as they are close to the camera. In our case we have lanes becoming smaller with the distance and appear to almost converge even though they are really parallel lines in the real world. To correct this, we need to  apply perspective transform and get a birdes eye view of the lanes.

The code for my perspective transform includes a function called `perspective_unwarp()`, which appears in lines #225 through #255 in the file `image_process.py`. The `perspective_unwarp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the extract the source points on the lanes manullay using small script provide in `./lane_select/lane_select.py` and destination points are calculated with offset to map source points back to orinal image size.

Following are source and destination points hardcoded in file `advanced_lanefind.py`  at line #308 and passed as argument to pipeline exectution. Points are  ordered left-top, right-top, bottom-right, bottom-left respectively.

```
vertices = np.array([[572,461],[709,459],[1052,674],[267,678]])
src = np.float32(vertices)

h_offset = 0 
w_offset = 200

dst = np.float32(
    dst = np.float32([[w_offset, h_offset],
    [w-w_offset, h_offset],
    [w-w_offset, h-h_offset],
    [w_offset, h-h_offset]]))

```
This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 572, 461      | 200, 0        | 
| 709, 459      | 1080, 0       |
| 1052, 674     | 1080, 720     |
| 267, 678      | 200,  720     |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Then I did some other stuff and fit my lane lines with a 2nd order polynomial kinda like this:

![alt text][image5]

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in lines # through # in my code in `my_other_file.py`

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

