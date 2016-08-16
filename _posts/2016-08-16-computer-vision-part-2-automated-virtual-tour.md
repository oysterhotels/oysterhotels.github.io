---
title: "Automated virtual tour"
author: Tuan
layout: post
permalink: /computer-vision-part-2-automated-virtual-tour/
disqus_page_identifier: computer-vision-part-2-automated-virtual-tour
published: false
---

![HDR Panorama](/public/images/cv2-cover.png)

Welcome back to the 2nd of the 3-part Computer Vision series at [Oyster.com](https://www.oyster.com). If you have not seen our 1st part of the series, we would recommend you [check it out](http://tech.oyster.com/computer-vision-part-1-hdr-panorama/), in that part we show how HDR (High Dynamic Range) panoramas are done at Oyster. That includes some comparisons between [Oyster's](https://www.oyster.com/new-york-city/hotels/trump-soho-new-york/all-tours/lobby--v24780/) panoramas and [Google's](https://www.google.com/maps/@40.7257012,-74.0055927,3a,75y,266.18h,79.62t/data=!3m8!1e1!3m6!1s-H0_jyN7iLq8%2FVyv1THyMdGI%2FAAAAAAABYto%2FsG9LtMP-Z8sO0bOxSsn8PxlxjU9h35N_gCLIB!2e4!3e11!6s%2F%2Flh5.googleusercontent.com%2F-H0_jyN7iLq8%2FVyv1THyMdGI%2FAAAAAAABYto%2FsG9LtMP-Z8sO0bOxSsn8PxlxjU9h35N_gCLIB%2Fw203-h101-n-k-no%2F!7i8704!8i4352!6m1!1e1).

In this part, I will share the core work of our recently released feature [Virtual Tours](https://www.oyster.com/french-polynesia/hotels/intercontinental-bora-bora-le-moana-resort/all-tours/pool--v35274/), also known as walkthroughs.

I will first give an introduction of walkthroughs and why we needed to build our own framework for this purpose. I will then jump right into the details of the computer vision system that generates virtual walkthroughs from sets of panoramas. Lastly I will show some walkthrough results obtained from this approach, including different scenarios for indoor and outdoor.

## Generating walkthroughs from panoramas
A walkthrough is a set of connected panoramas where users can navigate from one panorama to another. This type of feature provides more interactive experience for users looking at remote destinations. Some examples of walkthroughs can be checked out on Oyster.com, for example: [walkthrough for Trump Soho Penthouse](https://www.oyster.com/new-york-city/hotels/trump-soho-new-york/all-tours/penthouse-suite--v24773/) or [walkthrough along a Scrub Island Beach Pool](https://www.oyster.com/virgin-islands/hotels/scrub-island-resort-spa-and-marina-autograph-collection/all-tours/north-beach--v25717/). There are different approaches to generate those walkthroughs. 

One common approach is to use depth information to reconstruct the 3D scene for each panorama spot, like [Matterport](https://matterport.com). This provides seamless transitions from scene to scene. However, it comes with high cost in purchasing their own special device and model hosting, and its quality is not comparable to standard DLSR cameras. The more economic and more popular approach is one that uses 2D panoramas to build walkthroughs.  These can be viewed with popular 360 image viewers such as [Krpano](http://krpano.com/examples/vtour) that allow users to walk between panoramas.  The connections between panoramas can be built manually, but Oyster shoots thousands of walkthroughs a month and needed an automated solution.  We developed a fully automated framework that uses Computer Vision to find and connect all related panoramas into one complete walkthrough.

## Automating walkthrough process
Our process starts with a set of HDR panoramas as input (Figure 1 shows a list of test panoramas taken at TripAdvisor Office@NYC).  It finds the panoramas that are connected, estimates the links between those panoramas, and integrates those links into a Krpano virtual tour project. 

**Figure 1: Set of equirectangular panoramas as input**
![Set of equirectangular panoramas as input](/public/images/cv2-pano-set.png)

Given two panoramas, which we can call 1 and 2, the problem of creating a virtual tour from these two panoramas becomes finding the location of camera 1 in panorama 2's spherical coordinate, and location of camera 2 in panorama 1's spherical coordinate. By projecting our spherical coordinates (defined by horizontal angular ath and vertical angular atv) into planar coordinates (horizontal value x and horizontal value y) (where the reprojection from planar to spherical is given by: ath = (x/width - 0.5) * hfov and atv = (y/height - 0.5) * vfov, and hfov = 360 and vfov = 180 for our panorama), resulting in a set of local planar slices, we can look for the location of camera 1 in all local slices of camera 2's model and vice versa, the location of camera 2 in all local slices of camera 1's. Therefore, the original problem of finding camera location for each pair of panoramas becomes a search for camera location in all slices of the other panorama, given a set of n panoramas and m local slices for each panoramas we will need to carry out n * (n - 1) * m * (m - 1) / 4 slice-slice matchings.

Figure 2 illustrates the top-down view between panorama 1 (in blue, having camera center O1) and panorama 2 (in yellow, having camera center O2). Each panorama sphere is warped onto a cube, which appears as a square looking top-down with 4 sides FRONT-RIGHT-BACK-LEFT.

**Figure 2: Automated process for generating virtual tour**
![Camera models for two panoramas ](/public/images/cv2-pano-epipolar-topdown.png) 

For each image-image matching of 2 slices, the camera location is the center of the pin-hole camera model, and the geometry between these two image models is an epipolar geometry. If we look at image 1-FRONT and 2-RIGHT, camera centers O1 and O2 of the two images form a line from image 1 to image 2 and intersects each image at E1 and E2, or the Epipoles in epipolar geometry. The Epipole E1 on image 1 is the pixel coordinate (x1, y1) of camera center O2 on image 1, and the Epipole E2 on image 2 is the pixel coordinate (x2, y2) of camera center O1 on image 2. 

Without loss of generality, let us assume sides Front and Back of camera 1 share overlapping views with sides Right and Left of camera 2 in this example. Our process will need to locate E1 (image of camera center O2 in panorama 1) and E2 (image of camera center O1 in panorama 2). The coming sections will describe how these two values can be calculated based on the overlapping plane and corresponding points detected in both images. The whole process for this image-image matching has 4 main steps: construct local views, find corresponding points, find hotspot location in local coordinate, and transform local to spherical coordinate. Those steps are illustrated in Figure 3.

**Figure 3: Automated process for generating virtual tour**
![Automated process for generating virtual tour](/public/images/cv2-virtual-tour-process.png)

### Construct local views
A 360 panorama is a representation of a sphere which center is at the camera location. The original equirectangular format of input panorama can be divided into 6 non-overlapping rectangular local views representing the Up, Down, Left, Right, Front, and Back side of the cube covering the 360 sphere of the panorama scene. This division enables us to use epipolar geometry constraints of two image planes sharing overlapping views to find epipoles, which are images of camera positions in our case. We leave out Up and Down views since they do not contain hotspots for virtual tour.

An equirectangular panorama can be split into 6 rectangular images using krpano tool, the result is shown in Figure 4

```python
krpanotools64.exe makepano image.tif normal.config
```

**Figure 4: Slicing panorama into local views**
![Slicing panorama into local views](/public/images/cv2-pano-disintegrating.png)

 This slicing warps all pixels (having spherical coordinates ath, atv) on the sphere into the pixels (having planar coordinates x, y) on the sides of the bounding cube. Each side dimension is twice the radius of the cube. The spherical-planar projection is defined by x = (ath / hfov + 0.5) * width and y = (atv / vfov + 0.5) * height, which we will use to project our estimated camera planar coordinates back to spherical coordinates.

### Find corresponding points
In pin-hole camera model, a pixel coordinate on an image represents a set of points lying on the ray light from camera center towards that point in 3D (and goes on to infinity). With another camera viewing the same scene, we can see that line, or in other words, a point in one camera is transferable into a line in another camera in epipolar geometry, this line is corresponding line, as illustrated in Figure 5. All the corresponding lines have a common property, they all go through the Epipole, that is, given all points on image 1-FRONT in Figure 2, we can project all corresponding lines on image 2, and all these corresponding lines intersect at Epipole E2 on image O2, and similarly for Epipole E1 on image 1. In order to find corresponding lines from image points in pixel coordinate, we need first to find the fundamental matrix of the epipolar geometry. This fundamental matrix is a rank-2 3x3 matrix that represents the relative pose (translation + rotation) of image Left and right Right (or vice versa) as well as the intrinsic parameters of two camera. It has 7 parameters, 2 for each Epipole, and 3 for the homography that relates the two image planes. The convenient property of fundamental matrix is that it can be calculated from sufficient corresponding points. Corresponding points here are pixel points appear on two images that are pointing to the same 3D real-world point.

**Figure 5: Epipolar geometry of the overlapping view of 2 camera models**
![Epipolar geometry](/public/images/cv2-pano-epipolar.png)

With all those theories established, our problem of connecting panoramas into a virtual walkthrough now comes down to finding corresponding points on each slice image pair of the two panoramas. For this task, we resort to feature matching, which is a robust approach for dynamic views. The matching consists of three main steps, feature detection and feature matching and feature pruning. 


#### Feature detection
Feature detection is the process of running pre-defined feature filters on an image to discover features that are discriminative and view invariant (for example point at corners or edges where ). OpenCV has implementation for a collection of robust local features such as FAST, STAR, SIFT, or SURF (please check out [OpenCV's documentation](http://docs.opencv.org/3.1.0/db/d27/tutorial_py_table_of_contents_feature2d.html#gsc.tab=0) for more available feature detectors)

An example of how SIFT can be used for detecting local features is (Note: Since SIFT and SURF are patented feature, you should use other free features provided by OpenCV to avoid license fee)

```python
detector = cv2.xfeatures2d.SIFT_create(nfeatures=2000, nOctaveLayers=3, 
    contrastThreshold=0.03, edgeThreshold=10, sigma=1.6)
kp, des = detector.detectAndCompute(self.image, self.mask)
```
In the code above, there are 2 paramas that are quite important, nfeatures and contrastThreshold, reducing contrast threshold or increasing number of maximum features will give us more features, and vice versa. Those values should be chosen based on the nature of image data that we are dealing with, and the focus of our detection process. In our case, the distance between camera locations in real-world coordinate is unknown, which could be too far so a more appropriate design is to extract as many features as we can at detection phase, then in feature matching and pruning phase we will filter out irrelevant features. More practical decisions like this will be discussed later in our last section, along with our coarse-to-fine approach to efficiently extract features at constrained processing time.

The following figure shows the result of our feature detection process on two images, one from each panorama, local features are drawn in different colors, and as we can see they are mostly detected on corners and edges, and some features seem to be detected on both paranomas, those are the corresponding points that we are looking for. 

**Figure 6: Local features detected from the two images**
![Local features detected from the two images](/public/images/cv2-pano-keypoint-detection.png) 

#### Feature matching 
At each location where the feature is detected, a set of attributes are extracted to define that feature, they are called feature descriptors, some of the most common feature descriptors implemented in OpenCV are SIFT, SURF, HOG, BRIEF, BRISK, again please refer to [OpenCV's documentation](http://docs.opencv.org/3.1.0/db/d27/tutorial_py_table_of_contents_feature2d.html#gsc.tab=0) for more available feature descriptors. Those descriptors can be seen as a normalized (orientation-wise) vectorized aggregation (spatial-wise) of primitive filter response (e.g. SIFT descriptor gives a 128-dimensional vector aggregated from 4 x 4 location bins in left-right top-down spatial order, each bin is represented by accumulated gradients grouped in 8 orientation bins).

Featue matching is the process of finding the same set of features that appear in both images given feature descriptors (in forms for multidimensional vectors). Given ten of thousands of features being detected in each image, an efficient matching approach is to use kd-tree (k dimensional binary tree) to first index all features of one image and matching with features from the other image can then be done by traversing the indexed trees. In order to minimized false negatives in matching, knn (finding the nearest k matches for each feature) is also used. All those theories can be done with OpenCV API in 2 lines of code

```python
matcher = cv2.FlannBasedMatcher(dict(algorithm=FLANN_INDEX_KDTREE, trees=5), dict(checks=50))
matches = matcher.knnMatch(self.pano_image_from.des, self.pano_image_to.des, k=2)
```

In this context, number of trees, traversal checks, and number of k nearest neighbors are chosen based on accuracy-time tradeoffs in our particular applications. The following figure shows the result of this matching step, as we can see in this design we aim to reduce false negative matches so more matches are returned than needed, those matches will be cleaned up in the following feature pruning section.


**Figure 7: Feature matching results**
![Feature matching results](/public/images/cv2-pano-keypoint-matching.png) 

#### Feature pruning
Feature pruning is a series of filters being applied onto matched features, to filter out features that have passed through our previous matching process (which was purely based on similarity in appearance) but are not the exact corresponding features. These wrongly matched features are actually very common, most of the time they are features on similar or the same objects (e.g. features on the brick wall do share the same appearance across the whole wall, features on 2 different corners of the board do have similar appearance, just a rotated version of each other), or features that are noise (those are normally random dots on clean backgrounds that appear so frequently that there are many matches of these, with similar matching distance).

A successful feature matching system is mostly determined by how well feature pruning is implemented, if it is too strict we might end up with insufficent matches, but if it is too loose that will not only increase our processing time but will return in incorrect matches for later phases.

Pruning features should be designed based on the nature of the data. For our problem, there are 4 main pruning filters that can be used, which are ratio filter, cross-match filter, orientation consistency filter and spatial consistency filter.

* Ratio filter is designed to remove feature noise as mentioned above, the idea is, given all matches of a feature along with their similarity scores, a feature is determined as a feature noise when its best and second best match has too similar similarity scores. Again, if a feature can be matched with similar confidence to 2 different features, that feature is considered as a feature noise (e.g. a random dot on a wall could be matched with similar confidence with other random dots). This filter checks for the ratio of similarity score between the second best match and the best match, that ratio has to fall under a certain value for that feature and that match to be valid. This pruning technique was first proposed by David Lowe (the author of SIFT feature) using 0.7 as ratio threshold, but this type of filter could be used effectively with any features that have explicit similarity scores.

* Cross-match filter is checking for mutual matching result of a pair of features, in other words, 2 features are considered to be correctly matched when each feature appears in the match list of the other feature. 

* Orientation consistency filter is specific for our particular problem where there are no rotation in the transformation of one image into another, that is, given the cameras are placed and locked on tripod when taking the photo, that results in images having same upright orientation wherever the tripod is placed. This filter checks for the dominant orientation of the feature and its match to see if they are close. This filter can be applied on any feature detector that calculates dominant orientation.

* Spatial consistency filter is similar to orientation consistency, because image orientation is preserved between different shots, relative spatial relationship between filters are preserved, and this filter checks if a feature and its match keeps this relationship.

The following figure shows the result of feature pruning process after 4 filters have been applied, those features indicate a more clean and accurate match than the first matching set we obtained from pure appearance comparison.

**Figure 8: Feature pruning results**
![Feature pruning results](/public/images/cv2-pano-match-inliers.png) 

### Find hotspot location in local coordinate

The corresponding features detected from two images are then used to find hotspot location in local coordinate, this is done in three steps, first to find the fundamental matrix of the epipolar geometry constructed from these two sets of corresponding points, then use the fundamental matrix and the two sets of features to find a set of corresponding lines on each image, and lastly interesections of corresponding lines on each image are derived as the epipoles or camera locations in local planar coordinate.

As discussed previously, fundamental matrix in epipolar geometry can be estimated based solely of a set of coordinates (with at least 8 corresponding pairs). A robust approach towards finding fundamental matrix is by using RANSAC (Random sample consensus) technique, which is an iterative process where at each iteration, a random subset of the corresponding pairs are chosen as the "inliers" to construct the sample fundamental matrix, the rest of the corresponding pairs are then treated as test samples, where the sample fundamental matrix is used to find estimated corresponding points of the test set, which is then compared with the actual corresponding points of the test set to find inliers in the test set, which is then used as the measure for this sample pick. The best sample fundamental matrix is then chosen and returned. The following code shows how it is done in OpenCV (here fundamental_mask is used to trace back on original matches for inlier (or valid) matches).

```python
fundamental_mat, fundamental_mask = cv2.findFundamentalMat(valid_matches_left, valid_matches_right, cv2.FM_RANSAC)
```

Given the fundamental matrix, we can then calculate the epilines on the other image for every inlier point from one image.

```python
valid_matches_left = valid_matches_left[fundamental_mask.ravel() == 1]
valid_matches_right = valid_matches_right[fundamental_mask.ravel() == 1]
epilines_left = cv2.computeCorrespondEpilines(valid_matches_right.reshape(-1, 1, 2), 2, fundamental_mat)
epilines_right = cv2.computeCorrespondEpilines(valid_matches_left.reshape(-1, 1, 2), 2, fundamental_mat)
```

Once epilines are detected, epipole is then derived as the intersection of those epilines, and epipoles that fall within the image boundaries are valid. The estimated epipoles indicate the ray that two cameras are connected, so it could go from one image to another or vice versa. As we need to identify whether we can navigate from one image to another, we use the concept of far-near relationship to describe the 2 images. Given 2 images are showing the same scene, there is one image is closer to the scene than the other image, that is the requirement for valid epipole to be found within image boundary, and the navigation will only happen from the far image to the close image, in order words we are looking for the far image as its epipole is the hotspot we are finding. In order to find out which image is further from the scene, we use the average distance all all inlier features to its mean location (in both horizontal and vertical dimension). Image with smaller average distance is the image further away from the scene.


**Figure 9: Hotspot estimation - forward**
![Correct hotspon estimation](/public/images/cv2-pano-hotspot-estimation.png) 

Out of all possible local view matches (16 matches for 4 view consideration - LEFT, RIGHT, FRONT, BACK - or 36 matches for 6 local view consideration - including TOP, DOWN) we should ideally end up with 2 epipole locations, the first one to go from one panorama to the other, and the second one to go back from the other panorama. However, in practice we normally end up with more than 2 valid epipoles, we use a metric called average vertical distance to rank pairs of epipoles in terms of correctness. Average vertical distance is the average of the distance from the two estimated epipoles to the middle lines. In theory, given the camera tripod is at a fixed height level, the epipoles should always reside on the middle of the image, so we can use this property to find the best epipole pair that has the minimum distance to the middle lines of the images. Using this metric, we are then be able to locate the best hotspot in all matches from one pano to the other pano (E1 in Figure Epipolar Top-Down), in turns help us decide the opposite hotspot on the other side of the spherical cube from the other pano back (E2 in Figure Epipolar Top-Down)

**Figure 10: Hotspot estimation - backward**
![Correct hotspon back](/public/images/cv2-pano-hotspot-estimation-backward.png) 

### Transform local to spherical coordinate

Once the location of hotspots in local planar coordinate are found, we can derive global planar coordinate based on the index of the local view plane, then spherical coordinate can be calculated based on panorama's size and field of view

**Figure 11: Projecting local coordinates to spherical coordinates**
![Correct hotspon back](/public/images/cv2-pano-hotspot-integrating.png) 

## Practical implementation tips and tricks
So far we have presented a complete workflow to generate virtual tours automatically using OpenCV and Krpano. There are few additional points that we might take into consideration when trying to implement this workflow for production scale.
 
* Local view matching: an alternative to running all 4 x 4 local views matching, we can use the result one view against other 4 views to decide we need to carry out the rest of the matches
* Coarse-to-fine framework for feature detection and feature matching: the number of features detected and matched is proportional to processing time and accuracy, so we derived a coarse-to-fine framework with 3 layers going from small number of features (coarse layer) to large number of features (fine layer). Only when some good matches are identified at a coarser level, we can move on to the finer level. This type of framework reduces greatly processing time while still maintains high accuracy in matching.
* Metrics to find best hotspot: in this context we use the average distance to the horizon line of the image as the metric to find best epipole location, but that metric could also be other estimators that work best for the scenario, for example good feature ratio (ratio of final inlier feature counts on all detected features), or the amount of inliners found...
* Free vs. non-free: One important note about feature choice is the extra cost involved in using non-free features (SIFT, SURF), for production code it is best to consider the tradeoffs between accuracy, processing time and cost to make sure you pick the most suitable feature type for your application.

## Results and summary

In this post, we have presented the complete workflow to create virtual tour from a Computer Vision approach, using local feature matching and epipolar geometry. We also discussed some practical notes about implementing such system for production scale. Together with [our presented work in HDR panoramas](http://tech.oyster.com)/computer-vision-part-1-hdr-panorama), we can create high quality virtual tours to boost up user experience and engagement on our sites. Again, if you have not checked out our previous part where we talked about how High Dynamic Range panoramas are created at large scale, please [check it out](http://tech.oyster.com)/computer-vision-part-1-hdr-panorama), in the next and last part of this Computer Vision series, we will show you how to integrate smart features like moving mini-maps or dynamic mouse arrows into virtual tours. 

Here are some of the virtual tours generated from the approach we just described, the links above each screenshot will lead you to our live walkthroughs on Oyster.com, with support for both desktop and mobile.

[Dream Downtown Lobby](https://www.oyster.com/new-york-city/hotels/dream-downtown/all-tours/lobby--v22596/)
![cv2-wt-dream-downtown-lobby](/public/images/cv2-wt-dream-downtown-lobby.png)

[Gansevoort The Chester](https://www.oyster.com/new-york-city/hotels/gansevoort-meatpacking-nyc/all-tours/the-chester--v18296/)
![cv2-wt-gansevoort-the-chester](/public/images/cv2-wt-gansevoort-the-chester.png)

[Holiday Inn Montego Main Pool](https://www.oyster.com/jamaica/hotels/holiday-inn-resort-montego-bay/all-tours/main-pool--v24936/)
![cv2-wt-holiday-inn-montego-main-pool](/public/images/cv2-wt-holiday-inn-montego-main-pool.png)

[Refinery Hotel Rooftop](https://www.oyster.com/new-york-city/hotels/the-refinery-hotel/all-tours/refinery-rooftop--v19460/)
![cv2-wt-refinery-hotel-rooftop](/public/images/cv2-wt-refinery-hotel-rooftop.png)

[Scrub Island North Beach](https://www.oyster.com/virgin-islands/hotels/scrub-island-resort-spa-and-marina-autograph-collection/all-tours/north-beach--v25717/)
![cv2-wt-scrub-island-north-beach](/public/images/cv2-wt-scrub-island-north-beach.png)

[Trump Soho Penthouse](https://www.oyster.com/new-york-city/hotels/trump-soho-new-york/all-tours/penthouse-suite--v24773/)
![cv2-wt-trump-soho-penthouse](/public/images/cv2-wt-trump-soho-penthouse.png)


### About the author:
Tuan Thi is a Senior Software Engineer in Computer Vision at [Oyster](https://www.[Oyster.com](https://www.oyster.com)).com, part of Smarter Travel Media Group, at TripAdvisor. He finished his PhD in Computer Vision and Machine Learning in 2011.  Before joining TripAdvisor, he was a research engineer and computer vision scientist at Canon Research and Placemeter Ltd. with various international publications and patents in the field of local features, structured learning and deep learning.


