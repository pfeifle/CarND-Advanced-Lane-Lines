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

[image1]: ./output_images/calibration_result.jpg "Calibration via chessboard"
[image2]: ./output_images/calibration_result_highway.jpg  "Road Transformed"
[image3]: ./output_images/ColorTransformation_Gradient.jpg  "Color Transformation and Gradient Processing "
[image4]: ./output_images/warp1.jpg "Definition of Warp Points"
[image5]: ./output_images/warp2.jpg "warp points for perspective transformation"
[image6]: ./output_images/warp3.jpg "Applying perspective transformation"
[image7]: ./output_images/warp4.jpg "Applying perspective transformation and binary thresholding"
[image8]: ./output_images/polynomial.jpg "2nd order Polynomial"
[image9]: ./output_images/test.jpg "test image for lane detection"
[image10]: ./output_images/final.jpg "final image of the project"
[video1]: ./project_video.mp4 "Video Input"
[video2]: ./project_result.mp4 "Video Output"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  
 
---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code `cell #6 abd #8` of the IPython notebook located in `./P4.ipynb`.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. `imgpoints` are detected by using the open CV function `cv2.findChessboardCorners(gray, (9,6), None)` (the chessboard has 9x6 inner nodes).

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

The resulting camera matrix and the distortion values are stored in a pickle file for further usage (cf. code `cell #8` of `P4.ipynb`)


###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one. The code generating the image below can be found at `cell #1` of `P4.ipynb`. To achieve this the camera matrix and calibration values are loaded from a pickle file and applied to real camera images by using the function `dst = cv2.undistort(img, mtx, dist, None, mtx)`

![alt text][image2]


####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at `cell #56` of `P4.ipynb`).  Here's an example of my output for this step. I applied the the color and gradient thresholds to an undistoreted image by using threshold values for the absolute x derivates from the `Sobel operator` and by using the `s-channel` from an HSV image. 

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform was developed by looking at an image containintg straight lines (cf. `test_images/straight_lines1.jpg`). Next, for points on the straight lines were marked (cf. image below) 


![alt text][image4]

As the image contained straight lines, the transformation values for the warp function were simply defined by assingning the same x-value to the top points as the ones of the bottom points (cf. code in `cell #75` of `P4.ipynb`). The code cell contains a warp function as well as the definition of an inverse perspective transformation `Minv`. The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:
 
![alt text][image5]

  
I verified that my perspective transform was working as expected by testing visually whether straight and paralles lanes are parallel in the warped image (cf. image below and  
 `cell #62` of `P4.ipynb`).

![alt text][image6]


In  `cell #70` of `P4.ipynb`) all techiques applied so far, i.e. calibration, color transforms, gradients and perspective transformation, were combined (cf. image below).

![alt text][image7]


####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In  `cell #95` of `P4.ipynb`) histograms of the binary warped image were applied to find the relevant lane pixels. in order to do this, the image was examined by 9 sliding windows.
The image below shows the detected left and right lane pixels in red an blue color. This information was used to find the 2nd order polynomial for each lane boundary. The polynomial was derived by using the numpy function `np.polyfit` (cf. image below)

![alt text][image8]

In order to check the functionality of the code developed so far, the lane was drawn on top of the undistorted image (cf. `cell #96` of `P4.ipynb` and image below)

![alt text][image9]
 

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

This was done in code cells `cell #59` and  `cel #48` of `P4.ipynb`). To get from pixel space to real world space the transformation provided by udacity were used, i.e.

`ym_per_pix = 30/720 # meters per pixel in y dimension`

`xm_per_pix = 3.7/700 # meters per pixel in x dimension`


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

As already described above, this was done in  `cell #95` of `P4.ipynb` based on histograms.
The image below shows the detected left and right lane pixels in red an blue color. The resulting 2nd order polynomial is drawn in yellow (cf. image below)

![alt text][image8]
 

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

In the code cells `cell #61`, `cell #62`, `cell #63` and `cell #75` all the above described algorithms were wrapped into functions which could be used from the function `process_images` in `cell #69`. The [input video](./project_video.mp4) was mapped frame by frame into the [output video](./project_result.mp4) (cf. `cell #76`) .


Here's a [link to my video result](./project_result.mp4)


The image below shows the final image of the video frame and gives an overall impression of what was done in this project.

![alt text][image10]



---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

All the techniques presented in the udacity lesson could be applied in a straight-forward way. To me the definition of the warp function seems rather crucial. I put quite some effort into defining the source and destination points rather carefully (cf. image below)

![alt text][image5]

small changes in the definition of these values seem to have a big difference.
The images were processed frame by frame without taking the result of former frames into consideration. In order to stabilize the lane detection algorithm further, a kind of low-pass filter could be implemented based on the results of the last frames which could filter out exceptional frames. 
As I was not quite sure how to generate information valid accross frames, i.e. something like static variables, I didn't pursue this approach as the achieved result seems to be good enough at least for the project video. :-).


