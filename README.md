[![Udacity - Robotics NanoDegree Program](https://s3-us-west-1.amazonaws.com/udacity-robotics/Extra+Images/RoboND_flag.png)](https://www.udacity.com/course/sensor-fusion-engineer-nanodegree--nd313)

# Udacity Nanodegree: Sensor Fusion

## Project 03: 3D Object Tracking

<p align="center">
    <img src="./Project/docs/Results.gif" width="700" height="400" title="3D object tracking" >
</p>

This project aims to develop a software stack that will enable us to achieve the following objectives.

```
1. Keypoint detectors and descriptors
2. Object detection using YOLO V3 framework
3. Image filtering based on the bounding boxes and spatial readings(for Lidar data)
4. Bounding box tracking based on the keypoint matching
5. Projecting the Lidar data on the image data
```
The complete Project pipeline is as follows. As you can see pipeline corresponding to the orange box is already implemented in the [Tracking project](https://github.com/Suraj0712/SFND_2_Camera_Based_2D_Tracking). The blue box corresponds to Lidar point processing and object detection. Finally, we combine the data from the orange and blue box to track the object and calculate thE Time to collision.

<img src="./Project/images/course_code_structure.png" width="779" height="414" />

To acheive our goal we need to complete following four major tasks: 

1. First, you will develop a way to match 3D objects over time by using keypoint correspondences. 
2. Second, you will compute the TTC based on Lidar measurements. 
3. You will then proceed to do the same using the camera, which requires to first associate keypoint matches to regions of interest and then to compute the TTC based on those matches. 
4. And lastly, you will conduct various tests with the framework. Your goal is to identify the most suitable detector/descriptor combination for TTC estimation and also to search for problems that can lead to faulty measurements by the camera or Lidar sensor. In the last course of this Nanodegree, you will learn about the Kalman filter, which is a great way to combine the two independent TTC measurements into an improved version which is much more reliable than a single sensor alone can be. But before we think about such things, let us focus on your final project in the camera course. 

### Dependencies for Running Locally
* cmake >= 2.8
  * All OSes: [click here for installation instructions](https://cmake.org/install/)
* make >= 4.1 (Linux, Mac), 3.81 (Windows)
  * Linux: make is installed by default on most Linux distros
  * Mac: [install Xcode command line tools to get make](https://developer.apple.com/xcode/features/)
  * Windows: [Click here for installation instructions](http://gnuwin32.sourceforge.net/packages/make.htm)
* Git LFS
  * Weight files are handled using [LFS](https://git-lfs.github.com/)
* OpenCV >= 4.1
  * This must be compiled from source using the `-D OPENCV_ENABLE_NONFREE=ON` cmake flag for testing the SIFT and SURF detectors.
  * The OpenCV 4.1.0 source code can be found [here](https://github.com/opencv/opencv/tree/4.1.0)
* gcc/g++ >= 5.4
  * Linux: gcc / g++ is installed by default on most Linux distros
  * Mac: same deal as make - [install Xcode command line tools](https://developer.apple.com/xcode/features/)
  * Windows: recommend using [MinGW](http://www.mingw.org/)

### Basic Build Instructions

#### 1. First of all, clone this repo:
```
$ git clone git@github.com:Suraj0712/SFND_3_3D_Object_Tracking.git
```

#### 2. Run Quiz
```
$ cd <Path_to_quiz_directory>
$ makdir build && cd build
$ cmake ..
$ make
$ ./<executable_name>
```
#### 3. Run Project
```
$ cd <directory_where_you_have_cloned_the_repo>/SFND_3_3D_Object_Tracking/Project/
$ makdir build && cd build
$ cmake ..
$ make
$ ./<executable_name>
```

### Project Rubric

#### FP.1 Match 3D Objects

Implement the method "matchBoundingBoxes", which takes as input both the previous and the current data frames and provides as output the ids of the matched regions of interest (i.e. the boxID property). Matches must be the ones with the highest number of keypoint correspondences.
   > For this task I have created a 2D array of size ```#bounding boxes in the previous frame * #bounding boxes in the current frame```. Then I iterate over matches and based on the location of keypoint in the current and previous frame to find the correspondence between bounding boxes. After iterating over all the matches I get a 2D array structure which is similar to the adjacency map between bounding boxes from two successive frames. To find the best match I iterate over the rows and search for max value. Finally, I updated the map containing the bounding box matches.

* Lid
<img src="./Project/images/10.png" width="779" height="414" title="Lidar Data on image plane before filterig"/>

<img src="./Project/images/11.png" width="779" height="414" title="Lidar Data on image plane after filterig"/>

#### FP.2 Compute Lidar-based TTC
Compute the time-to-collision in second for all matched 3D objects using only Lidar measurements from the matched bounding boxes between current and previous frame.

<img src="./Project/images/3.png" width="779" height="414" />

<img src="./Project/images/4.png" width="779" height="414" />

   > I used Newton's motion equation with a constant velocity model to find the time to collision. From the Lidar, we get several distance values and we can consider ```Average, Min, Max``` value for the distance. However, these quantities are highly affected by outliers. To mitigate this to some extent, I used the ```Median``` value of distance reading.

#### FP.3 Associate Keypoint Correspondences with Bounding Boxes
Prepare the TTC computation based on camera measurements by associating keypoint correspondences to the bounding boxes which enclose them. All matches which satisfy this condition must be added to a vector in the respective bounding box.
   > So here we have a vector of keypoint matches between the current and previous frame. There will be an error in the keypoint matches and to account that error and mitigate it to some extent I did the following processing on the data. I iterated over the keypoint matches and calculated the mean distance value. Once i get the mean distance I calculated the standard deviation. To remove the outliers I filtered the readings which are within ```one standard deviation``` from the mean distance value.

#### FP.4 Compute Camera-based TTC
Compute the time-to-collision in second for all matched 3D objects using only keypoint correspondences from the matched bounding boxes between current and previous frame.

<img src="./Project/images/6.png" width="779" height="414" />

<img src="./Project/images/7.png" width="779" height="414" />

   > As you can see in the above images based on Pinhole camera assumption, symmetrical triangles, and size of the object in the image plane we can calculate the TTC with the camera data. So based on the Keypoint matches I calculated the distance ratio for all key-points in the current and previous frame. Then to get rid of outlier values, I considered the ```Median``` value of distance ration to calculate the Time to collision.

#### FP.5 Performance Evaluation 1
Find examples where the TTC estimate of the Lidar sensor does not seem plausible. Describe your observations and provide a sound argumentation of why you think this happened.

   > Following are the results I got with the ```FAST + BRIEF``` detector descriptors. Most of the time camera gave the higher readings than the Lidar however this I think its because the way i am handling the outliers. 

|Frame     |0      |1      |2      |3      |4      |5      |6      |7      |8      |9      |
|----------|-------|-------|-------|-------|-------|-------|-------|-------|-------|-------|
|Camera TTC|16.2688|16.7673|17.7801|15.3782|16.118 |16.3351|16.4664|17.6808|17.3522|18.6324|
|Lidar TTC |12.5156|12.6142|14.091 |16.6894|15.7465|12.7835|11.9844|13.1241|13.0241|11.1746|

   > For some frames I got the INF, Negative or High variation``` in the TTC values for both the sensors. After careful examination, I observed that most of the time the root cause is improper detection and selection of the bounding box. 

1. I also observed significant ```noise in the Lidar data and calibration Noise for camera data```. which resulted in a significant difference in the TTC.
2. I think significant improvement is possible in the given estimate if we consider the ```multiple points for TTC calculation, fuse Camera and Lidar sensor on hardware, and Kalman filter```. if we consider the multiple points the estimate will be robust. Fusing the sensors will reduce the calibration error significantly. Finally, Kalman filter because of its recursive nature will yield better results.

[Click here for detailed Analysis](./Project/docs/FP5_Results/)


#### FP.6 Performance Evaluation 2
Run several detector/descriptor combinations and look at the differences in TTC estimation. Find out which methods perform best and also include several examples where camera-based TTC estimation is way off. As with Lidar, describe your observations again and also look into potential reasons.

1. To get the performance analysis I ran the loop over different descriptors -->```BRISK, BRIEF, ORB, FREAK```, and detector -->```SHITOMASI, FAST, BRISK, ORB, AKAZE``` combinations. Upon observation, I found that the ```ORB``` detector performed worst. The same goes for ```SIFT```, However, SIFT gave reliable results most of the time. The camera TTC seems to be affected with the Histogram based detectors and matching performance is severely affected which can be due to changes in ```scale and intensity```.

2. I think considering more than two frames ~5 would help to get a better estimate. Also as mentioned in the FP5 addition of Kalman filter will result in significant improvements in reliability and performance.

Based on the and TTC data distribution and combine with our previous benchmark. Here are the top 3 detector/descriptor combinations:

FAST + BRIEF

FAST + ORB

FAST + BRISK

[Click here for detailed Analysis](./Project/docs/FP6.csv)

|Detector_type|Descriptor_type|ImgNumber|TTC_Lidar|TTC_Camera|
|-------------|---------------|---------|---------|----------|
|FAST         | BRISK         |0        |         |          |
|FAST         | BRISK         |1        |12.2891  |12.551    |
|FAST         | BRISK         |2        |13.3547  |12.8653   |
|FAST         | BRISK         |3        |16.3845  |12.6318   |
|             |               |         |         |          |
|FAST         | BRIEF         |0        |         |          |
|FAST         | BRIEF         |1        |12.2891  |11.6697   |
|FAST         | BRIEF         |2        |13.3547  |12.0207   |
|FAST         | BRIEF         |3        |16.3845  |13.7125   |
|             |               |         |         |          |
|FAST         | ORB           |0        |         |          |
|FAST         | ORB           |1        |12.2891  |12.5425   |
|FAST         | ORB           |2        |13.3547  |12.5265   |
|FAST         | ORB           |3        |16.3845  |13.7667   |
|             |               |         |         |          |
|FAST         | FREAK         |0        |         |          |
|FAST         | FREAK         |1        |12.2891  |12.9991   |
|FAST         | FREAK         |2        |13.3547  |12.093    |
|FAST         | FREAK         |3        |16.3845  |14.6114   |
|             |               |         |         |          |
|ORB          | BRISK         |0        |         |          |
|ORB          | BRISK         |1        |12.2891  |10.676    |
|ORB          | BRISK         |2        |13.3547  |13.5068   |
|ORB          | BRISK         |3        |16.3845  |11.2793   |
|             |               |         |         |          |
|ORB          | BRIEF         |0        |         |          |
|ORB          | BRIEF         |1        |12.2891  |12.2865   |
|ORB          | BRIEF         |2        |13.3547  |14.042    |
|ORB          | BRIEF         |3        |16.3845  |19.9017   |
|             |               |         |         |          |
|ORB          | ORB           |0        |         |          |
|ORB          | ORB           |1        |12.2891  |19.0873   |
|ORB          | ORB           |2        |13.3547  | -inf     |
|ORB          | ORB           |3        |16.3845  |21.8644   |
|             |               |         |         |          |
|ORB          | FREAK         |0        |         |          |
|ORB          | FREAK         |1        |12.2891  |12.2074   |
|ORB          | FREAK         |2        |13.3547  |11.1048   |
|ORB          | FREAK         |3        |16.3845  |11.3288   |
|             |               |         |         |          |
|AKAZE        | BRISK         |0        |         |          |
|AKAZE        | BRISK         |1        |12.2891  |11.8455   |
|AKAZE        | BRISK         |2        |13.3547  |13.1113   |
|AKAZE        | BRISK         |3        |16.3845  |13.0918   |
|             |               |         |         |          |
|AKAZE        | BRIEF         |0        |         |          |
|AKAZE        | BRIEF         |1        |12.2891  |12.101    |
|AKAZE        | BRIEF         |2        |13.3547  |13.519    |
|AKAZE        | BRIEF         |3        |16.3845  |12.0061   |
|             |               |         |         |          |
|AKAZE        | ORB           |0        |         |          |
|AKAZE        | ORB           |1        |12.2891  |11.6102   |
|AKAZE        | ORB           |2        |13.3547  |13.5195   |
|AKAZE        | ORB           |3        |16.3845  |12.745    |
|             |               |         |         |          |
|AKAZE        | FREAK         |0        |         |          |
|AKAZE        | FREAK         |1        |12.2891  |11.5026   |
|AKAZE        | FREAK         |2        |13.3547  |12.7074   |
|AKAZE        | FREAK         |3        |16.3845  |12.827    |
|             |               |         |         |          |
|SHITOMASI    | BRISK         |0        |         |          |
|SHITOMASI    | BRISK         |1        |12.2891  |13.3689   |
|SHITOMASI    | BRISK         |2        |13.3547  |13.0305   |
|SHITOMASI    | BRISK         |3        |16.3845  |12.3202   |
|             |               |         |         |          |
|SHITOMASI    | BRIEF         |0        |         |          |
|SHITOMASI    | BRIEF         |1        |12.2891  |13.7746   |
|SHITOMASI    | BRIEF         |2        |13.3547  |13.308    |
|SHITOMASI    | BRIEF         |3        |16.3845  |11.3769   |
|             |               |         |         |          |
|SHITOMASI    | ORB           |0        |         |          |
|SHITOMASI    | ORB           |1        |12.2891  |13.603    |
|SHITOMASI    | ORB           |2        |13.3547  |13.2121   |
|SHITOMASI    | ORB           |3        |16.3845  |12.884    |
|             |               |         |         |          |
|SHITOMASI    | FREAK         |0        |         |          |
|SHITOMASI    | FREAK         |1        |12.2891  |13.7616   |
|SHITOMASI    | FREAK         |2        |13.3547  |11.8519   |
|SHITOMASI    | FREAK         |3        |16.3845  |11.4126   |
|             |               |         |         |          |
|BRISK        | BRISK         |0        |         |          |
|BRISK        | BRISK         |1        |12.2891  |12.8659   |
|BRISK        | BRISK         |2        |13.3547  |16.5957   |
|BRISK        | BRISK         |3        |16.3845  |12.7803   |
|             |               |         |         |          |
|BRISK        | BRIEF         |0        |         |          |
|BRISK        | BRIEF         |1        |12.2891  |10.1889   |
|BRISK        | BRIEF         |2        |13.3547  |16.1327   |
|BRISK        | BRIEF         |3        |16.3845  |11.5662   |
|             |               |         |         |          |
|BRISK        | ORB           |0        |         |          |
|BRISK        | ORB           |1        |12.2891  |11.7327   |
|BRISK        | ORB           |2        |13.3547  |15.829    |
|BRISK        | ORB           |3        |16.3845  |14.2885   |
|             |               |         |         |          |
|BRISK        | FREAK         |0        |         |          |
|BRISK        | FREAK         |1        |12.2891  |11.2939   |
|BRISK        | FREAK         |2        |13.3547  |19.6872   |
|BRISK        | FREAK         |3        |16.3845  |12.2048   |


## Thank you!!!


