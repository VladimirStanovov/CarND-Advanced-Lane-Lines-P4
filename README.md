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
[image3]: ./output_images/test6_thresholded.jpg "Thresholded method 1"
[image4]: ./output_images/test6_thresholded2.jpg "Thresholded method 2"
[image5]: ./output_images/test6_lanes_boxes.jpg "Lines and boxes"
[image6]: ./output_images/test6_lanes_area.jpg "Lines search area"
[image7]: ./output_images/test6_final_visualisation.jpg "Final output"
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

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()`, which appears in lines 1 through 8 in the file `example.py` (output_images/examples/example.py) (or, for example, in the 3rd code cell of the IPython notebook).  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
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
| 1127, 720     | 960, 720      |
| 695, 460      | 960, 0        |

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

