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

[image1]: ./camera_cal/calibration1.jpg "Initial image"
[image2]: ./output_images/calibration1_undistorted2.jpg "Undistorted"
[image2_2]: ./output_images/straight_lines1_undistorted.jpg "Undistorted 2"
[image3]: ./output_images/test4_thresholded.jpg "Thresholded method 1"
[image4]: ./output_images/test4_thresholded2.jpg "Thresholded method 2"
[image4_2]: ./output_images/straight_lines2_thresholded2.jpg "Thresholded method 2"
[image5]: ./output_images/test4_lanes_boxes.jpg "Lines and boxes"
[image6]: ./output_images/test4_lanes_area.jpg "Lines search area"
[image7]: ./output_images/test4_final_visualisation.jpg "Final output"
[video1]: ./processed_project_video.mp4 "Video"
[video2]: ./processed_challenge_video.mp4 "Video"
[video3]: ./processed_harder_challenge_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

###Camera Calibration

####1. Camera matrix and distortion coefficients.

The code for this step is contained in the first code cell of the IPython notebook located in "Project4.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Initial image
![alt text][image1] 
Undistorted image
![alt text][image2]

###Pipeline (single images)

####1. Distortion-corrected test image.
After testing on chessboard images, I applied the distortion correction to one of the test images like this one:
![alt text][image2_2]

####2. Color transforms, gradients or other methods to create a thresholded binary image.

Cells 4-8 in Project4.ipynb are responsible for functions performing Sobel thresholding, gradient calculation, and using HLS (S channel used) & HSV (V channel used) color maps. Cell 9 combines individual thresholding methods with certain parameters. 

```
ksize = 5
gradx = abs_sobel_thresh(img, orient='x', sobel_kernel=ksize, thresh=(25, 100))
grady = abs_sobel_thresh(img, orient='y', sobel_kernel=ksize, thresh=(25, 100))
mag_binary = mag_thresh(img, sobel_kernel=ksize, mag_thresh=(25, 100))
dir_binary = dir_threshold(img, sobel_kernel=ksize, thresh=(0.75, 1.25))
hls_binary = hls_select(img, thresh=(150, 255))
hsv_binary = hsv_select(img, thresh=(150, 255))
```

####3. Perspective transform.

Cell 10 contains basic image transformation functions, which include getting the perspective transformation M, thresholding, and getting the bird's eye view thresholded image. The following vertexes were used to get perspective:

```
[[[img.shape[1]*0.41,img.shape[0]*0.65]],[[img.shape[1]*0.59,img.shape[0]*0.65]],
[[img.shape[1]*0.0,img.shape[0]*0.95]],[[img.shape[1]*1.0,img.shape[0]*0.95]]]
```
![alt text][image3]

On this image you may see that lines get wider when they are further from the camera. Although this could be a good option, this happens due to the fact that we first perform binary thresholding, and then make perspective transform:
```
newimg = get_thresholded(img)
finalimg = getperspectiveimage(newimg,M)
```
However, we could first make perspective transform, and then threshold the image:
```
newimg = getperspectiveimage(img,M)
finalimg = get_thresholded(newimg)
```
...which makes a significantly different result:

![alt text][image4]

Further is this project, I used the second option (get perspective -> threshold).

I verified that my perspective transform was working as expected by looking at the transformed binary image and making sure that both straight lines are parallel to y axes.

![alt text][image4_2]

####4. Identifying lane-line pixels and fitting their positions with a polynomial

Identifying line-line pixels is split in two functions: getinitiallines (cell 11) finds the lines with 9 sliding windows, after that the 2nd order polynomials are fit to both lines' pixels. It also draws rectangles around found windows.
The getnextlines function (cell 12) uses fitted polynomials to find new pixels, and fit new polinomials to them. Is also drwas the "search area" around fitted lines.
You may see the resluts of these function on the followin images:

![alt text][image5]
![alt text][image6]

####5. Calculateing the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for calculating the curvature is provided in cell 16. It simply uses the known equation, and applies known standard lane width.
An example of vehicle bias calculation (to the left lane) is performed in cell 19. It simply uses 3rd parameter of the fitted polynomial as it appears to be a relatively good estimation of the lane position. However, video processing pipeline uses different approach.

####6. Example image of result plotted back down onto the road such that the lane area is identified clearly.

Cell 19 contains getting the lanes positions and plotting it back on the original image, as well as putting text with curvature and bias values on the image.

![alt text][image7]

---

###Pipeline (video)

The video pileline uses functions processnewimage and processnextimage in cells 17 and 18. These are similar to cells 11 and 12, but return a whole set of image processing parameters. Cell 20 defines the Line class used in the pipeline. Cell 21 contains the main pipeline of video processing, i.e. the processimage function, which gets the raw image and returns the processed one.
First, this function calls processnewimage, and using the returned values calculates averaged line positions on the image (bestx) over last 5 steps.  Moreover, it keeps track of fit parameters using exponential smoothing:

```
LeftLine.best_fit[0] = LeftLine.best_fit[0]*0.8 + 0.2*LeftLine.current_fit[0]
LeftLine.best_fit[1] = LeftLine.best_fit[1]*0.8 + 0.2*LeftLine.current_fit[1]
LeftLine.best_fit[2] = LeftLine.best_fit[2]*0.8 + 0.2*LeftLine.current_fit[2]
RightLine.best_fit[0] = RightLine.best_fit[0]*0.8 + 0.2*RightLine.current_fit[0]
RightLine.best_fit[1] = RightLine.best_fit[1]*0.8 + 0.2*RightLine.current_fit[1]
RightLine.best_fit[2] = RightLine.best_fit[2]*0.8 + 0.2*RightLine.current_fit[2]
```

Bias calculation is performed using 25% of the lane height (because the lanes may have curvature!) and uses averaged bestx parameters of each line. For curvature calculation, the averaged parameters are used as well, this apperas to make estimations more stable.
At the end, we perform a check wheather the parameters of both lines are close enough (lines are parallel if they are!), and if they are not, we perform lanes search again. Otherwise, for the next frame, the processnextimage function is called, which uses averaged known lanes positions. If this part fails to find lanes, the known parameters are returned, and the next step again searches for the lanes using sliding window in processnewimage function.

Here's a [link to my video result for project video](./processed_project_video.mp4)

There are also videos for both challenges, which, unfortunately, don't show good results.

Here's the [challenge video](./processed_challenge_video.mp4)
Here's the [harder_challenge video](./processed_harder_challenge_video.mp4)
---

###Discussion

To succesfully solve the challenges one may think of making dinamic change of the vertexes that are cutting the part of the image with the road. Moreover, non-linear transformations of the cutted part may be helpfull, especially those which are based on the curvature value.
