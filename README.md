This project is a fork of the lane marking detection project within Udacity's Self Driving Car Engineer Nanodegree Program. A decription of my implementation can be found below the original project decription.
The code is included in the IPython notebook P1.ipynb

# **Finding Lane Lines on the Road** 
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

Overview
---

When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project you will detect lane lines in images using Python and OpenCV.  OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.  

To complete the project, two files will be submitted: a file containing project code and a file containing a brief write up explaining your solution. We have included template files to be used both for the [code](https://github.com/udacity/CarND-LaneLines-P1/blob/master/P1.ipynb) and the [writeup](https://github.com/udacity/CarND-LaneLines-P1/blob/master/writeup_template.md).The code file is called P1.ipynb and the writeup template is writeup_template.md 

To meet specifications in the project, take a look at the requirements in the [project rubric](https://review.udacity.com/#!/rubrics/322/view)



# **Finding Lane Lines on the Road** 

## Writeup

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report
---

### Reflection

### 1. Description of the image processing pipeline. 

The pipeline consists of 5 steps to detect lane markings in an original image (cf. Fig. 1). 

<figure>
 <img src="./test_images/solidYellowCurve2.jpg" width="380" alt="original image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 1: original image </p> 
 </figcaption>
</figure>
 <p></p> 

First, the images are converted to grayscale (cf. Fig. 2), then edges are detected with a canny edge detection (params: low_thresh=70, high_thresh=240) (cf. Fig. 3)

<figure>
 <img src="./test_images_output/gray/solidYellowCurve2.jpg" width="380" alt="grayscale image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 2: grayscale image </p> 
 </figcaption>
</figure>
 <p></p>
 
 <figure>
 <img src="./test_images_output/canny/solidYellowCurve2.jpg" width="380" alt="canny image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 3: edges detected with canny algorithm </p> 
 </figcaption>
</figure>
 <p></p> 


Afterwards, only lines within the region of interest ([[(580, 340), (420, 340), (140, 540), (900, 540)]]) are selected (cf. Fig. 4).

<figure>
 <img src="./test_images_output/roi/solidYellowCurve2.jpg" width="380" alt="roi image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 4: edges within the region of interest </p> 
 </figcaption>
</figure>
 <p></p>

In order creat smoother and also longer line segments a gaussian blurring (kernel size=3) is applied to the edge image (cf. Fig. 5).

<figure>
 <img src="./test_images_output/blurred/solidYellowCurve2.jpg" width="380" alt="blurred image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 5: blurred edges within the region of interest </p> 
 </figcaption>
</figure>
 <p></p>
 
 
Afterwards, the created edges are tranformed to hough space (rho=2, theta=3*(np.pi/180), threshold=30, min_line_len=20, max_line_gap=4), where the longest line belonging to the left and right marking respectively is determined (cf. Fig. 6). Afterwards, the determined lines are exctended up to the lower edge of the picture and the upper border of the region of interest (or in the special case of intersection lines up to the intersection point). In addition, the lateral extend of the markings is estimated and the entire constructed marking is drawn.

To do so, the original draw_lines() function was modified as follows:
* the actual calculation of the lines was moved to a seperate function (calc_lines()) in order to be accesible from another function draw_markigns() as well
* the calc_lines() function expects the image and the hough lines as input and determines the longest line belonging to the left and right border respectively and returns it
* the draw_markings() and hough_markings() functions were defined - they draw the markings as polygons instead as lines


<figure>
 <img src="./test_images_output/hough/solidYellowCurve2.jpg" width="380" alt="lines image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 6: extended longest line belonging to the left and right marking (red); estimated outer borders of the markings (blue) </p> 
 </figcaption>
</figure>
 <p></p>

Back in the original image, the constructed lines can be shown as overlay (cf. Fig. 7).

<figure>
 <img src="./test_images_output/weighted/solidYellowCurve2.jpg" width="380" alt="combined image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 7: original image overlayed with estimated lane markings as lines </p> 
 </figcaption>
</figure>
 <p></p>

The constructed lines are converted to a polygon spanning over the full extend of the markings (cf. Fig. 8).

<figure>
 <img src="./test_images_output/final/solidYellowCurve2.jpg" width="700" alt="combined image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;"> Fig. 8: original image overlayed with estimated lane markings as polygons </p> 
 </figcaption>
</figure>
 <p></p>

### 2. Shortcomings with the current pipeline

The current pipeline shows some minor shortcomings:

* The detection of dashed lines is not as stable as the one of solid lines. The reasons is that the approach uses the longest line as reference and as the car is moving along the dashed line, the reference is continously changing.
* The approach uses the longest line as reference and extends it with a small amount of pixels to the left and to the right. But as it is not known, wether the current line belongs to the right or left border of the considered marking it is not clear, into which direction, the line should be extended more than to the other. In some rarely occuring situations, this also leads to instabilities.
* In the challenging example, it is hard to detect lines in the shadowed region. In addition, the road rather curvy - therefore, it is a bad idea to estimate the the markings with single lines.

### 3. Possible improvements to the pipeline
The described shortcoming could be fixed as follows:

* In order to create smoother lane markings, a smoothing over consecutive pictures of the video stream could be applied.
* To get better line approximation in shadowed regions, the parameters could be tuned further or other color spaces could be tried out.
* For curvy roads, for exaymple, a polynom of third order could be used to approximate the lane markings.

