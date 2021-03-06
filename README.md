## Advanced Lane Lines - Kevin Lee



### Advanced Lane Finding Project



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



[image1]: ./writeup_imgs/chessboard.png "Chessboard Undistorted"

[image2]: ./writeup_imgs/distort.png "Undistorted Road Image"

[image3]: ./writeup_imgs/s_channel.png "S Channel"

[image4]: ./writeup_imgs/combined.png "Combined Thresholds"

[image5]: ./writeup_imgs/warped.png "Perspective Transform"

[image6]: ./writeup_imgs/binary.png "Warped Binary"

[image7]: ./writeup_imgs/sliding_win.png "Sliding Windows"

[image8]: ./examples/color_fit_lines.jpg "Fit Visual"

[image9]: ./examples/example_output.jpg "Output"

[image10]: ./writeup_imgs/lines_filled.png "Lines Filled"

[image11]: ./writeup_imgs/l_channel.png "L Channel"

[image12]: ./writeup_imgs/b_channel.png "B Channel"

[video10]: ./project_video.mp4 "Video"



---



### Camera Calibration



#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.



The code for this step is contained in the 4th code cell of the IPython notebook called `camera_calibration.ipynb`.



I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.



I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:



![alt text][image1]



I then stored the calibration in a pickle file called `cam_calibration.p`.



### Pipeline (single images)



#### 1. Provide an example of a distortion-corrected image.



I loaded the pickled camera calibration data and applied "cv2.undistort" to the test images.  An example of one of the images (original and undistorted) can be seen below.  It is difficult to notice too much of a difference, but you can see a change in the hood of the car.



![alt text][image2]



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.



I first played around with some of the color spaces to see which were effective in isolating the lane lines.  After a bit of exploration, I decided to use the L channel from LUV to detect the white lanes and the B channel from LAB to detect the yellow lanes.



**L Channel**

![alt text][image11]



**B Channel**

![alt text][image12]



I also took a look at gradient options, including Sobel X and Y, magnitude, and directional.  After initially trying to use these as lane detectors, I found that color thresholding was sufficient.  Therefore I decided to not use gradient thresholding at all.  I combined the L Channel and B Channel binary image outputs together and resulted in the following combined images.



![alt text][image4]



#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.



The code for my perspective transform includes a function called `warper()`, which appears in the 3rd code cell of the IPython notebook `perspective_transform.ipynb`.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner based on mapping out the trapezoid on a test image:



This resulted in the following source and destination points:



| Source        | Destination   | 
|:-------------:|:-------------:| 
| 205, 720      | 205, 720       | 
| 1120, 720      | 1120, 720      |
| 745, 480     | 1120, 0      |
| 550, 480      | 205, 0       |



I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.



![alt text][image5]



I then ran the warped test images through the color/gradient transform and resulted in the following images.



![alt text][image6]



I stored the perspective transform in a file called `perspective_transform.p`.



#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?



In `advanced_lane_lines.ipynb`, I have the function "sliding_windows_fit" in code cell 9 that fits my lane lines with a 2nd order polynomial and uses sliding windows.  It essentially uses a histogram to determine the peaks of the warped images which should correspond to the lane lines.  Windows around these areas are used to highlight the points used for polynomial fitting.  I had to increase with the width of the sliding windows a bit to capture some of the dotted white line in the extreme top of the warped image.



![alt text][image7]



#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.



I did this in code cell 13 (get_curvature function) in `advanced_lane_lines.ipynb`.  I used the formulas from the lectures on "Measuring Curvature" after mapping the back to the real world scale using the approximate meters per pixel in the x and y dimensions.  To find the position of the vehicle, I specified the middle of the image (where the camera is), and then found the center of the lane by using the fit of the lines.  Finally, I found the difference between these two and multiplied by the meters per pixel in the x direction.



#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.



I implemented this step in code cell 12 in `advanced_lane_lines.ipynb` in the function `draw_lane()`.  Here is an example of my result on a test image:



![alt text][image10]



---



### Pipeline (video)



#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).



Here's a [link to my video result](./project_video_output.mp4)



---



### Discussion



#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?



I approached this project by following the rubric and the lecture portion.  I tried to break up the project into parts (camera calibration, color/gradient thresholding, perspective transform, curvature, etc.).  In general, most of my issues involved playing with parameters to get the desired result.  I had to play with the threshold values, the sliding window parameters, the perspective transform source, destination, and offset values.



When I first ran the pipeline on the video, I noticed there were a few problematic frames that messed up the lane tracking.  I implemented the Line object and a queue that stores the last 20 fits and used the average of those fits for drawing the lanes back onto the images.  This smooths out the error caused by the erratic frames.  I also added a method that checks if the fit is bad by signaling if the respective fitted lines crosses the center threshold (it shouldn't).



**Resubmission Edit Start**



Fine tuning thresholds were probably the biggest challenge I had in this project.  I did quite a bit of experimenting with different color spaces as well as the gradient options, and finding robust threshold ranges to extract just enough information took a decent amount of trial and error.


In terms of problems, the method I used to implement detecting a bad fit (whether the lines crossed a certain threshold) limits the window for a line to be detected, so if there is a sharp curve, the line detected might be labeled as a bad fit.



1. To possibly remedy this, I could try other means of detecting a bad fit (using polynomial coefficients, testing for paralellism, testing if the curve differs too much from previous fits, etc.) that would be more robust of a solution.



2. I could also implement the feature of using previous successful fits as a window for future fits, which could also boost performance as the algorithm would have less searching to do through the images.



3. I could also extract more information from the fit/line detection from frame to frame to better help the smoothing process and limit the effect of bad fits.



The pipeline could also fail if unforeseen objects obstruct the warped lane detection area.  This could occur if there are vehicles in the lane where detection is occuring, as the color/gradient thresholds will pick up the vehicle outlines as well as the lane lines so the sliding windows fit might be confused and use part of the vehicle as the best fit.



Other examples of potential issues could be nightime driving, different color pavement, lack of actual lane lines, or too many shadows for an extended period.



**Resubmission Edit End**



I believe my pipeline could fail in areas with much more shaded regions.  To combat this, I would need to make the color/gradient thresholding more robust to these areas, perhaps by adding in more color channels that are better at detecting lane lines in darker images.  There are also a few frames where the lane line detection is quite a bit off, the smoothing I added help with this (taking the average of the last 10 fits), but I could possibly add some more checks that throw away fits if they do not result in a certain amount of "parallelism".

