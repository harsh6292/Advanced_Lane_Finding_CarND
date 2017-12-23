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
[im1]: ./output_images/0_Chessboard/1_Original_Image.jpg "Chessboard Original"

[im2]: ./output_images/0_Chessboard/2_Draw_Corners.jpg "Chessboard Corners"

[im3]: ./output_images/0_Chessboard/3_Undistorted_img.jpg "Chessboard Undistorted"

[im4]: ./test_images/test6.jpg "Test Image Original"

[im5]: ./output_images/1_Undistorted_Test_Img/img_frame_0.jpg "Test Image Undistorted"

[im6]: ./output_images/2_Thresholded_Img/img_frame_0.jpg "Test Image Thresholded"

[im7]: ./output_images/3_Warped_Img/img_frame_0.jpg "Test Image Warped"

[im8]: ./output_images/4_Sliding_Window/img_frame_0.jpg "Test Image Sliding Window"

[im9]: ./output_images/5_Sliding_Window_Lane_Line/img_frame_0.jpg "Test Image Sliding Window With Lane Lines"

[im10]: ./output_images/6_Unwarped_Lines_On_Road/img_frame_0.jpg "Test Image Inverse Warped"

[im11]: ./output_images/7_Unwarped_Lines_Text_Fill/img_frame_0.jpg "Test Image Inverse Warped With Text and lane area filled"




The following are the steps I performed in order to display lane lines on images/videos.

All the code is written in an IPython notebook located
[here](https://github.com/harsh6292/Advanced_Lane_Finding_CarND/blob/master/Advanced_lane_Finding.ipynb) [File name: Advanced_lane_Finding.ipynb].

### Camera Calibration

The camera calibration code is located in the cells (4, 5, 6) of the above IPython notebook.

I start by creating 54 `object points` with each point having (x, y, z) coordinates. The chessboard has 9x6 inner corners due to which we have 54 points in total.

The chessboard is assumed to be put on a flat surface which will translate to having z=0. The object points are thus array of points starting at (0,0,0) and ending at (8,5,0).

The `image points` are on the other hand points which are detected by using the openCV function findChessboardCorners(). This method will find and return all the points found inside the chessboard. The object points are appended for each set of image points found in an image.

The `object points` and `image points` are then used to calibrate the camera using the openCV function calibrateCamera().

Following is the result of camera calibration applied on chessboard images.

Original Image
![alt text][im1]

Chessboard corners in the image
![alt text][im2]

Undistored chessboard image
![alt text][im3]



### Pipeline (single images)
Next, I created a Pipeline for processing images and drawing lane lines on test images.



##### Distortion Correction
An example of test image of how I applied distortion correction.

Original Test Image
![alt text][im4]

Undistored Test Image
![alt text][im5]



##### Thresholding
Next I applied color and gradient thresholding to the Undistorted images.

For gradient thresholding I applied Sobel operator from openCV and applied thresholding in both the 'x and 'y directions.

For color thresholding, I used L and S-channel from HLS converted image. I also used R & G channel from image and calculate the White and Yellow color channels from the combination of R & G.

I then created a combined binary of gradient and color thresholding.

(The code for thresholding is in cell '968 of the IPython notebook.)

Here's an example of my output for this step (showing same image as above).

![alt text][im6]


##### Perspective Transform
Next I applied perspective transform on the thresholded image by using openCV getPerspectiveTransform() and warpPerspective() functions.

The code for perspective transform is located in cell '969' of the notebook.

I chose the hardcode the source and destination points in the following manner:

```python
src = np.float32([
      [200, 720],
      [570, 470],
      [720, 470],
      [1130, 720] ])

offset = img.shape[1]/4

dst = np.float32([
      [350, 720],
      [350, 590],
      [980, 590],
      [980, 720] ])
```


Here's an example of my output for this step (showing same image as above).


![alt text][im7]


##### Drawing Lane Pixels and Windows
Next I used the perspective transformed image to find lane pixels in the image. In order to achieve this, I first found a window of likely pixels where the lane line is expected.

I found the window in the image by performing convolution of vertical slice of window with a signal value of one. Then I extracted the pixel locations where the combined convoluted signal value was maximum. These pixels formed my window of lane.

I then used these pixels to continue finding the lines around this area and store these pixel values.

Using the above pixel values found inside the window, I draw them on the warped image.

This is the result of finding pixels using a convolution signal.

![alt text][im8]



Then I fitted a 2nd order polynomial using these pixel values to create a continuous function for both left and right lane. Applying all the possible `y` values to the above 2nd order polynomial gives a curve that draws the line. The result looks with `y` values fitted looks like below:

![alt text][im9]

All the above result were obtained through code in cells (970, 971, 994) of the notebook.

##### Inverse Perspective Transform
Since the above points depicted the lane lines correctly, I used openCV method of warpPerspective() and inverse matrix transformation to transform warped image back to original.

I took the points drawn above and plotted them again black background. This image was then inverse transformed using above function. The resulting image was drawn over the original undistorted image to show lane lines.

The result looks like below:

![alt text][im10]


##### Measuring radius of curvature
The radius of curvature was calculated by first converting the lane pixels found above into the real world space values. That is, the pixel location values were converted to meters by multiplying the `x` and `y` coordinates with below values.

x_meter_per_pixel = 3.7/700
y_meter_per_pixel = 30/700

Following is the result of radius of curvature on above original image with lane lines drawn.

![alt text][im11]


---

### Pipeline (video)

The final pipeline was then applied to the project video to draw lane lines.

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion
During my time working with this project, I found finding the correct gradient and color threshold is most important as it will affect which part of the image does actually contain the lines and which part are just shadows or very bright spots from sunlight or reflection of other cars.

It took a long time for me to find a correct threshold values. I tried all thresholding mentioned in introduction like directional thresholding, magnitude thresholding and even using R-color and L-color channels in RGB and HLS color images, but they were not effective in isolating lane lines. In the end I used R & G color channels to make yellow and white binary representing yellow and white pixels. I used S color and L color channel too just to isolate shadows.

The second difficult part was that I first used sliding window approach to find the lane lines. This turned out to be bit less accurate and involved lot of tracking of variables. I then switched to convolution method of finding lanes which gave better results compared to sliding window.


I think the pipeline where it might fail would be areas where the objects around lane lines are very bright as compared to test images. In this case, the thresholding pipeline would certainly have various false positives and would make the car go off the road.

This would also create issues when finding the window centers as the convoluted signal would give more range of pixels where lane is found.

To make it more robust, I think I need to experiment with color and gradient thresholding more in order to better isolate lane lines from any abnormalities in the image caused by too much brightness etc.


### References
1. The project walkthrough provided by Udacity helped me a lot un understanding how to approach the problem and gave ideas when I got stuck in sliding window.
https://www.youtube.com/watch?v=vWY8YUayf9Q&index=4&list=PLAwxTw4SYaPkz3HerxrHlu1Seq8ZA7-5P
