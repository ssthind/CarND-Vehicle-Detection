## Writeup 

---

**Vehicle Detection Project**

The steps of this project are the following:

* Perform Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Also applied a color transform and append binned color features, as well as histograms of color, to HOG feature vector. 
* The data is normalized using `StandardScaler` from `sklearn.preprocessing` features and randomize a selection for training and testing using ` train_test_split` from `sklearn.model_selection`.
* Implemented sliding-window technique and used trained classifier to search for vehicles in images.
* Ran pipeline on a video stream on full project_video.mp4 and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/car_not_car.png
[image2]: ./output_images/HOG_car.jpg
[image21]: ./output_images/HOG_NOT_car.jpg
[image3]: ./output_images/sliding_windows_single1.jpg
[image4]: ./output_images/Test_frame_w_HeatMaps.jpg
[image5]: ./output_images/vid_frame.jpg
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one. [Here](https://github.com/ssthind/CarND-Vehicle-Detection/blob/master/writeup.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it! The completed project code is in Jupyter note book file [Vehicle_detection.ipynb](https://github.com/ssthind/CarND-Vehicle-Detection/blob/master/Vehicle_detection.ipynb). Also included a no-run version for easy download(as it has less size due to now images attached)

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the code cell of the IPython notebook.

I started by reading in all the [vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/vehicles.zip) and [non-vehicle](https://s3.amazonaws.com/udacity-sdc/Vehicle_Tracking/non-vehicles.zip) images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `YCrCb` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:

###### HOG transform for car image across different color channel
![HOG transform car image different color channel][image2]

 HOG-transform of Y-channel seems to be extracting better feature information


###### HOG transform for NOT car image across different color channel
![HOG transform Not car image different color channel][image21]

Although various other channel of different color space where also tried, but that channel did not help obtain better HOG feature. and few channel resulted in error while converting to other color spaces(eg error U and V channel of LUV). However Y-channel of color space 'YCrCb' resulted in better HOG related feature extraction. As Y-channel contain more information the other color channels for differnt color space (e.g R/G/B of RGB color space) and `YCrCb` was specifly created for digital tramission of visual information over eletronic medium.

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters on single channel and multi-channel across various color channels and finalized on HOG of Y-channel of color space 'YCrCb'

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using the feature extracted for the train data set, and also trained using the rbf kernal with on RGB color channel, but performace was not good. So final version consist of trained linear SVM on features vector consisting of features extracted from image in 'YCrCb' color space: Spatial Binning, Color Histogram, HOG features on Y-channel. Next saved it in pickle file using `pickle.dump(), pickle.load()` functions  for further reuse(also to save time against notebook/kernal crash and restore process)

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I had implemented 3 different size of windows with dimensions 64x64, 96x96, 128x128 with following percentage overlap 85, 80, 70. These sliding window have the following y coordinate ranges: [400, 528], [510, 610], [500, 756]. Further x coordinate is also restricted to increase the speed to process frame. Total we get around 478 search windows to see for car in each frame of the video.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images of 
##### pipeline stages image:

![pipeline][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./project_video_output.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video. Saved the past frames detections data for averaging over past frames.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected, as show previously in pipeline stages images.  

![Video frame][image5]
---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I had trained and save various SVM classifier, when features where extracted using various combination of spatial bining, color histogram, HOG-transform of Y color channel of YCrCb. And observer when the parameters for number of past frame to be averaged `no_frame_avg` , `heat_threshold` can played around with to reduce the false positive or noise in detection. Giving much cleaner output, even when classifier test accuracy was lesser(~95%). And when classifier accuracy goes higher(~98+%) then above parameter can be reduced.

This pipeline could be further improved with using different, more dynamic classifier and feature extraction method such as deep neural networks. However sliding window should still be used to achieve scale and translation in-variance in the detection process.