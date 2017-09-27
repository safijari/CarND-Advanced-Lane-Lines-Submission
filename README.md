# Advanced Lane Following Submission
The goals / steps of this project are the following:


- Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
- Apply a distortion correction to raw images.
- Use color transforms, gradients, etc., to create a thresholded binary image.
- Apply a perspective transform to rectify binary image ("birds-eye view").
- Detect lane pixels and fit to find the lane boundary.
- Determine the curvature of the lane and vehicle position with respect to center.
- Warp the detected lane boundaries back onto the original image.
- Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

**Note:** All code is in the `project_notebook.ipynb` file. I’ll reference cells below.

# Rubric Points:

**Camera Calibration**
**1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.**

The code for this is under the **Camera Calibration** heading in the notebook. This is essentially the same as the lecture, we create “idealized” object points in 3D (z is zero), for each of the internal corner in a calibration target. Then, using opencv’s `findChessboardCorners` we discover the same points in a real image as shown below:

![Corner points found in 2D on an image](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506480964158_file.png)


The  `calibrateCamera` function takes as an input these points and the idealized points, it then tries to discover the camera matrix as well as distortion coefficients for a radial distortion model. These can be passed to the `cv2.undistort` function to rectify the image. One example of a rectified image is shown below:

![Note how the lines at the top are "straight" in the undistorted image](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506481086966_file.png)


**Pipeline (single images)**
All code for this is under the **Pipeline** heading in the notebook
**1. Provide an example of a distortion-corrected image.**


![Original](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506481349242_file.png)
![Undistorted](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506481356353_file.png)


These cameras have very little distortion overall, so it’s difficult to tell the effect in a natural image.

**2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.**
**3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.**
Note: these are too intermingled in my code to separate…

All of this is under the `process_image`  function. I first convert the image into HLS color space and then do a perspective transform, as shown pictorially below:


![I transform this quadrangle to ...](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506481957021_file.png)
![...this image shape](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506481964732_file.png)


Note that I made a different height and width for the transformed image, this was to keep some semblance of an aspect ratio after the transform so that a pixel in the x direction should roughly be the same size as a pixel in the y direction.

I then do two thresholds, one on the `s` channel and one on the `l` channel. Then I add them to get a final binary mask.


![Note that S channel is better at picking up the yellow line and the L channel is better at picking up the white line, this makes sense as yellow would have a high saturation value, whereas white has a high intensity](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506482224953_file.png)


**4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?**
**5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.**

I then chunk the combined mask to 5 pieces in the vertical direction, split it at a horizontal pixel coordinate of 500 (for the project video, the two lines stay always separated at around 500), and then locate boxes at the maximum values of a “vertical sum” for each vertical chunk. I also reject boxes that meet certain criteria (e.g. outside image). The result looks something like the left image:

![Red rectangles for left line, green for right](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506482511730_file.png)


The `fit_line_and_find_curvature` function takes in a split binary mask created using the boxes in the above image as well as past polynomial coefficients and fits a polynomial to them. The fit is discarded if the mean squared error between the current and the past fit exceeds an empirically discovered threshold. It also finds the curvature. Note that I do numerical derivatives to find the curvature. 

Outside of this function call, I compute the lane deviation by using the points at the bottom of the above transformed image. I compare the center of these two points with an empirically determined center value of `622`. 

**6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.**

![ Note that the curvature values are for either line in meters, as is the deviation. I trust the left line's curvature value more than the right line's, the sparsity of the data on the right makes that estimate a tad suspect!](https://d2mxuefqeaa7sj.cloudfront.net/s_E54D41902493E368C26A370C8415F0AFD499BCF17C7C3184751CAA863F062DC9_1506483233929_image.png)


**Pipeline (video)**
**1. Provide a link to your final video output. Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).**

Find it on [youtube](https://youtu.be/G12dVrJGXtI). It’s also in the repo.

**Discussion**
**1. Briefly discuss any problems / issues you faced in your implementation of this project. Where will your pipeline likely fail? What could you do to make it more robust?**

So, lots of stuff in my pipeline breaks down on the challenge video. It wouldn’t really generalize to varied road textures or lighting conditions. It also wouldn’t work if the car is changing lanes or the deviation or curvature is really high (this goes back to some hand tuned parameters). This said, my binary image is really clean, to the point that I actually don’t have to do any smoothing! I also fare well doing the full search for the lines rather than the relative search (it doesn’t take that long so whatever).

I think using Maximally Stable Extremal Regions would make a lot of sense for a situation like this. These fire well on text and text like objects in natural scenes and would likely work for lane lines as well. Beyond this the code is a bit of a mess but that neither here nor there.

