---
title: Aerial Navigation in GPS Denied Environments
date: 2021-10-18 19:13:20 +/-0530
categories: [Projects, DIY]
tags: [atmega328p,game,imu,touch]     # TAG names should always be lowercase
image_sliders:
  - gamingConsole
---
![Image1](/assets/img/OrthomosaicSLAM/campus_data.png){: .shadow}

A project on achieving realtime-orthomosaic SLAM for aerial navigation using a single downwards facing camera in outdoor GPS denied environments.

# Introduction
To carry out drone-based aerial surveying for generating orthomosaic maps on the fly, this project explores the image processing stack required to achieve the same using the most economical hardware and software footprint. The project explores corner and blob-based feature extraction techniques followed by brute force and KNN based feature matching methods which are later used to generate a homography matrix for stitching images that are passed through a cascaded image mixer to generate orthomosaic maps of a given dataset.


# Feature Detection & Extraction

While there is no universally accepted definition of “Feature” for a given image, it is often regarded as the information that is unique for the given image and thus helps us mathematically associate the image with its unique data properties. 

![Image1](/assets/img/OrthomosaicSLAM/features.png){: .shadow}

Image Features are small patches of unique raw data that can be potentially used to differentiate the given image from any other image, therefore, helping in tracking the similarity between given images. Image Features can be broken down into two major components:

1) Keypoints
2) Descriptor

They are explained in the subsequent sections.

## Keypoints
Keypoints contain 2D patch data like position, scale, convergence area, and other properties of the local patch. Which we define as a distinctive point in an input image that is invariant to rotation, scale, and distortion. In practice, the key points are not perfectly invariant but they are a good approximation. [1]

![Image1](/assets/img/OrthomosaicSLAM/hand.png){: .shadow}
_Example: Keypoints of a human hand._

## Descriptors & Detectors
A feature detector is an algorithm that takes an image and produces the positions (pixel coordinates) of important regions in the picture. A corner detector is an illustration of this, since it outputs the positions of corners in your picture but does not provide any other information about the features discovered.

A feature descriptor is an algorithm that takes an image and generates feature descriptors/feature vectors from it.[2] Feature descriptors encapsulate important information into a sequence of numbers and serve as a numerical "fingerprint" that may be used to distinguish one feature from another. They highlight various keypoint properties in vector format (constant length). It might be their intensity in the direction of their most pronounced orientation, for example. It is giving a numerical description of the area of the image to which the keypoint is referenced.

Important properties of descriptors 
 
- they should be independent of keypoint position
If the same keypoint is extracted at different positions (e.g. because of translation) the descriptor should be the same. [3]

- they should be robust against image transformations
Variations in contrast (for example, a photograph of the same location on a bright and overcast day) and changes in perspective are two examples (image of a building from center-right and center-left, we would still like to recognize it as the same building). Of all, no description is absolutely impervious to all modifications (nor against any single one if it is strong, e.g. big change in perspective). Different descriptors are meant to be resistant against various transformations, which are sometimes incompatible with the speed at which they are calculated. [3]
- they should be scale independent
The scale should be considered in the descriptions. If the "prominent" component of one key point is a vertical line of 10px (within a circular area with a radius of 8px), and the prominent part of another is a vertical line of 5px (inside a circular area with a radius of 4px), the descriptions for both key points should be identical. [3]

## Available Methods
### Shi-Tomasi Corner Detector

![Image1](/assets/img/OrthomosaicSLAM/campus1.png){: .shadow}

<!-- pic here -->
Example: Using Good Features to Track to detect keypoints

The Shi-Tomasi Corner Detector is a modified version of the Harris Corner Detector with the primary change being in the way in which the score “R” is calculated making it more efficient than the standard Harris Corner Detector. 

Corners are considered as areas in photographs where a little alteration in position, results in a significant change in intensity in both the horizontal (X) and vertical (Y) axes.   

The Pseudocode for Shi-Tomasi Corner Detector:
1. Determine windows of small image patches that produce very large variations in intensity when moved in both X and Y directions (i.e. gradients).[4]
2. Compute score R for each such window.
3, Apply a threshold to select and retrieve important corner points.

Python Implementation of Shi-Tomasi Corner Detector: 
![Image1](/assets/img/OrthomosaicSLAM/code2.png){: .shadow}



OpenCV has implemented a function cv2.goodFeaturesToTrack() the parameters for this function are:

- **image** - Input 8-bit or floating-point 32-bit, single-channel image
- **maxCorners** - Maximum number of corners to detect. If the number of corners on an image is higher than the number of maxCorners the most pronounced corners are selected.
- **qualityLevel** - Parameter characterizing the minimal accepted quality of image corners. All corners below the quality level are rejected. The parameter value is multiplied by the best corner quality measure, which is the minimal eigenvalue of the Harris function response.[5]
- **minDistance** - minimal Euclidean distance between corners

### Scale Invariant Feature Transform (SIFT)

![Image1](/assets/img/OrthomosaicSLAM/campus2.png){: .shadow}
_Example: Using SIFT to detect keypoints_

<!-- pic here -->
SIFT, which stands for Scale-Invariant Feature Transform, was introduced in 2004 by D.Lowe of the University of British Columbia. This algorithm is robust against image scale variations and rotation invariances.

The Pseudocode for SIFT:
1. Scale-space peak selection: Potential location for finding features.
2. Keypoint Localization: Accurately locating the feature keypoints.
3. Orientation Assignment: Assigning orientation to keypoints.
4. Keypoint descriptor: Describing the keypoints as a high dimensional vector.
5. Keypoint Matching

The Python Implementation for SIFT:
![Image1](/assets/img/OrthomosaicSLAM/code1.png){: .shadow}

Parameters for SIFT:
- **nfeatures** - The number of best features to retain. The features are ranked by their scores (measured in SIFT algorithm as the local contrast)
- **nOctaveLayers** - The number of layers in each octave. 3 is the value used in D. Lowe’s paper. The number of octaves is computed automatically from the image resolution.
- **contrastThreshold** - The contrast threshold is used to filter out weak features in semi-uniform (low-contrast) regions. The larger the threshold, the fewer features are produced by the detector.

### Speeded Up Robust Features (SURF)
The SURF (Speeded Up Robust Features) technique is a quick and robust approach for local, similarity invariant picture representation and comparison. The SURF approach's prominent feature is its quick computing of operators using box filters, which enables real-time applications such as tracking and object identification.

The Pseducode for SURF:
1. Feature Extraction 
    1. Hessian matrix-based interest points
    2. Scale-space representation
2. Feature Description
    1. Orientation Assignment
    2. Descriptor Components

<!-- pic here -->
![Image1](/assets/img/OrthomosaicSLAM/math.png){: .shadow}

The python implementation of SURF:
![Image1](/assets/img/OrthomosaicSLAM/code4.png){: .shadow}



public static SURF create​(double hessianThreshold, int nOctaves, int nOctaveLayers, boolean extended, boolean upright)

Parameters for SURF:

- **hessianThreshold** - Threshold for hessian keypoint detector used in SURF.
- **nOctaves** - Number of pyramid octaves the keypoint detector will use.
- **nOctaveLayers** - Number of octave layers within each octave.
- **extended** - Extended descriptor flag (true - use extended 128-element descriptors; false - use 64-element descriptors).
- **upright** - Up-right or rotated features flag (true - do not compute the orientation of features; false - compute orientation).

### Oriented FAST and Rotated BRIEF (ORB)
<!-- pic here -->
![Image1](/assets/img/OrthomosaicSLAM/code3.png){: .shadow}

Example: Using ORB to detect keypoints

ORB is a combination of the FAST keypoint detector and the BRIEF descriptor, with some additional characteristics to boost performance. FAST stands for Features from Accelerated Segment Test, and it is used to find features in a picture. It also employs a pyramid to generate multiscale features.[2] It no longer computes the orientation and descriptors for the features, which is where BRIEF is used.

ORB employs BRIEF descriptors, although the BRIEF performs badly when rotated. So what ORB does is rotate the BRIEF according to the keypoint orientation. The rotation matrix of the patch is calculated using the patch's orientation, and the BRIEF is rotated to obtain the rotated version. 

The Pseudocode for ORB:[7]
1. Take the query image and convert it to grayscale. 
2. Now Initialize the ORB detector and detect the keypoints in query image and scene. 
3. Compute the descriptors belonging to both images. 
4. Match the keypoints using Brute Force Matcher. 
5. Show the matched images.

The Python implementation of ORB:

## Conclusion
- One of the notable drawbacks of the Shi-Tomasi Corner Detector is the fact that it is not scale-invariant. 
- ORB is a more efficient feature extraction approach than SIFT or SURF in terms of computing cost, matching performance, and, most importantly, patents. 
- SIFT and SURF are patented, and you are required to pay for their use. However, ORB is not patented.
# Feature Matching
## Available Methods
### Brute Force Matcher
![Image1](/assets/img/OrthomosaicSLAM/compare1.png){: .shadow}

The Brute Force Matcher is used to compare the characteristics of the first image to those of another image.

It takes one of the first picture's descriptors and matches it to all of the second image's descriptors, then moves on to the second descriptor of the first image and matches it to all of the second image's descriptors, and so on.

# Homography & Transformation
![Image1](/assets/img/OrthomosaicSLAM/homography.png){: .shadow}

A homography connects any two photographs of the same scene. It is a transformation that transfers the points in one picture to the points in the other. The two pictures can be captured by rotating the camera along its optical axis or by laying them on the same surface in space.

The essence of the homography is the simple 3×3 matrix called the homography matrix.

<!-- math matrix shit here -->
This matrix may be applied to any spot in the picture. For example, in the first image, if we choose a position A(x1,y1), we may use a homography matrix to translate this point A to the matching point B(x2,y2) in the second image.
<!-- math matrix shit here -->

The python implementation of finding homography matrix:
![Image1](/assets/img/OrthomosaicSLAM/code5.png){: .shadow}


It is important to realize that feature matching does not always provide perfect matches. As a result, the Random Sample Consensus (RANSAC) process is used as a parameter in the cv2.findHomography() method, making the algorithm robust to outliers. We can get reliable results using this strategy even if we have a large percentage of bad matches.

### Python Implementation of RANSAC:

![Image1](/assets/img/OrthomosaicSLAM/code6.png){: .shadow}

# Image Processing Pipeline
<!-- image here -->
![Image1](/assets/img/OrthomosaicSLAM/code7.png){: .shadow}

## Dataset
### Urban Area
![Image1](/assets/img/OrthomosaicSLAM/campus_data.png){: .shadow}

This dataset of the Thammasat University campus in Bangkok, Thailand was collected by a senseFly eBee X drone carrying a senseFly Aeria X photogrammetry camera.

Technical Details
|Feature|Value|
|--|--|
Ground resolution|6 cm (3.36 in)/px
Coverage|2.1 sq. km (0.77 sq. mi)
Flight height|285 m (935 ft)
Number of images|443


### Forest Area
![Image1](/assets/img/OrthomosaicSLAM/forest_data.png){: .shadow}

DroneMapper flew Greg 1 and 2 reservoirs on September 16th, 2019 using their Phantom 3 Advanced drone to collect imagery for precision digital elevation model (DEM) and orthomosaic generation of the site. GCP was surveyed with a Trimble 5800 and used during the image processing. Approximately 189 images were collected and processed via photogrammetry to yield the DEM, orthomosaic, and capacity report.[8]

|Feature| Value|
|---|--|
|Ground resolution|6 cm (3.36 in)/px|
|Coverage|2.1 sq. km (0.77 sq. mi)|
|Flight height|285 m (935 ft)|
|Number of images|189|

# Future Work 
Improve performance of the keypoints and descriptors using a mask-based cascaded mixer. 
Add support for live video feed-based mosaic generation.
Add location tracking and path planning using frame centroid transformations.
 Investigate FLANN and other alternatives for Feature matching.

