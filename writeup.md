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


There are total three files to finish all CRITERIAs
### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Camera Calibration.ipynb" (or in lines # through # of the file called `Camera Calibration.ipynb`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  


`imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. Python opencv has a convenient method called "findChessboardCorners" that will identify the points where black and white squares intersect and reverse engineer the distorsion matrix this way. I save the found `corners` to save into `imgpoints`.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]:"./output_images/undistort_output.jpg"

I apply the distortion correction to one of the test car images like this one:
![alt text][image2]:"./output_images/car_undistort.jpg"


### Colorspace threshold and perspective

First, I practive the kinds of Colorspace threshold and get the feeling of the difference color space and gradient thresholds.
Then , I use one image of test images with the straight line to define the perspective transform matrix. 

the source (`src`) and destination (`dst`) points that I chose in the following manner:

```python
src = X0,y0=195,image.shape[0]
      x1,y1=1120,image.shape[0]
      x2,y2=689,450
      x3,y3=592,450

h=720 #height
w=750 #width
o=(copy.shape[1]-w)/2 #origin point,suppose the camera is located at center

dst = np.float32([[o,h],
                [w+o,h],
                [w+o,0],
                [o,0]])
```

The perspective transform produces the following type of images:
![alt text][image2]:"./output_images/car_perpective.jpg"

and generate to the matrix of M and inversed M to use then.


### Pipeline (images process)

Here is the writeup_up for the main project `Advanced lane_finding.ipynb`

#### Step1. Show the all images that I want to deal with.

Loading the images and built the useful functions steps at lines through # [2]-[4] #

#### Step2. apply the undistortion function to the images

Basically, here function is as same as the `Camera Calibration.ipynb` to use.

#### Step3. apply the combination of kinds of thresold to generate binary images

I apply color and edge thresholding in this section to better detect the lines, and make it easier to find the polynomial that best describes our left and right lanes later. Here I select the s-channel at hls color space a and x-direction gradient thresholds to generate the binary images. And then, I combined them though the function of "combined".

![alt text][image3]:"./output_images/Thresh_combined.jpg"

#### Step4. apply perspective transform to the binary images

I apply the M matrix generated from `Colorspace_threshold and perspective.ipynb` to the above binary images.

![alt text][image4]:"./output_images/Perspective_trans.jpg"


#### Step5. fitting the polylines, calculate the curvature and do some units translation.

I then compute a histogram of our binary thresholded images in the y direction, on the bottom half of the image, to identify the x positions where the pixel intensities are highest. Since we now know the starting x position of pixels (from the bottom of the image) most likely to yield a lane line, I run a sliding windows search in an attempt to “capture” the pixel coordinates of our lane lines.

From then, I simply compute a second degree polynomial, via numpy’s polyfit, to find the coefficients of the curves that best fit the left and right lane lines

I also compute the lane curvature by calculating the radius of the smallest circle that could be a tangent to our lane lines as following images:

![alt text][image5]:"./output_images/Find_fitting_curve.jpg"

#### Step6. Apply all the process to images once and write down the information on images.

Finally, I apply all the built process to images once time that is convenient to veiw all the algorithms, and unwarp the dealed images. I draw the inside the of the lane in green and unwarp the image. I also add textual information about lane curvature and vehicle’s center position:


![alt text][image5]:"./output_images/images_test.jpg"

---

### Pipeline (video)

Basically, the need functions are almost same as the Pipeline (images process), but I add two more functions for video process.
One is the fitting of searching around existed polylines. I supposed the pipelines are changing slowly so that can be easier to define the area from the last fitting to the next fitting.
Another is the iteration function that is used to smooth the lanes area to give the better performance to some tolerance. I use the 7 samples to average the fitting coefficents

Here's a [link to my video result](.Output_video//project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found two shortcoming of my project.
For one thing, in the challenge video, if the road have some patchs or slits within the sides of pipelines, and maybe just in the dark shadow environment, it would cause some error to judge the pipelines. The possbile solution may be to change the histogram method for find the bottom postion of pipelines in this project, but I still not yet think out what's a solution can use to substitute the current method.

For another thing, the fitting function is not enough dynamic. If there is suddenly an extreme curve of road, the fitting is hard to change fast enough to fit the change in the real world.
