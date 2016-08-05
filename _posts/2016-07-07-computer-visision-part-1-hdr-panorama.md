---
title: "Generating HDR panoramas at scale"
author: Tuan
layout: post
permalink: /computer-vision-part-1-hdr-panorama/
disqus_page_identifier: computer-vision-part-1-hdr-panorama
published: true
---

![HDR Panorama](/public/images/cover.png)

Here at [Oyster](https://www.oyster.com), we are the leading website for comprehensive photographic reviews of hotels. One key component of our imagery database is panoramas, produced at high quality and large scale (over 150,000 to date). In this three-part series, we will be looking at the Computer Vision work that has been part of our panorama pipeline. In this first part of the series, we will introduce our automated pipeline for generating High Dynamic Range (HDR) panoramas.

## HDR Panorama
[Panorama images](https://www.oyster.com/jamaica/hotels/moon-palace-jamaica-grande/all-panoramas/beach--v115415/) at [Oyster](https://www.oyster.com) have full angle range with 180 degrees vertically and 360 degrees horizontally. They provide an immersive experience for viewers to explore a ubiquitous view of the venue - whether it is outside at the pool, on the rooftop or inside a hotel room. Panorama images have now become a trending and must-have media type for most image-oriented websites. Meanwhile, HDR imaging is a common technique to produce a greater dynamic range of luminosity than standard digital imaging. It is  especially useful for panorama imaging where an evenly distributed look will greatly improve the quality. HDR imaging is normally achieved by merging multiple low-dynamic-range photographs. We use [PTGui](http://www.ptgui.com), a stitching software, to carry out batch stitching of 12 fisheye images (180 degree x 180 degree) of the four different views (left, right, front, back) with three images of different exposure for each view. The stitching process will return one equirectangular panorama.

#### Raw fisheye images
![Fisheyes images](/public/images/fisheyes.png)

While PTGui is a good choice for image stitching, its HDR quality is not the best available. People often use alternative tools for HDR merging. [SNS-HDR](http://www.sns-hdr.com) is the package used by [Oyster](https://www.oyster.com). It has support for batch processing, excellent HDR quality, and adequate deghosting support (compared to PTGui).

#### PTGui HDR
![PTGui HDR](/public/images/ptgui1.png)

#### SNS-HDR
![SNS-HDR](/public/images/snshdr1.png)

The tricky part for using SNS-HDR as a batch tool is its limited support on file format input and the auto-grouping of images of the same view, and that is where Computer Vision comes into play.

The first problem is on accepted formats for image input. SNS-HDR works with RAW files, but it works especially well with converted raw DNG format (using DNGConverter), compared to two common formats CR2 or NEF.

#### One possible artifact with SNS-HDR on original raw files
![SNS-HDR RAW](/public/images/raw.png)

#### Stable SNS-HDR merge on dng files
![SNS-HDR DNG](/public/images/dng.png)

The second problem of using SNS-HDR batch processing is to figure out the three images of the same view.  This is done using OpenCV  (Python binding with Numpy), one of the most comprehensive Computer Vision libraries to date. We use DCRAW to convert the DNG files to TIFF since OpenCV does not work directly with raw files (CR2, NEF, DNG). Then OpenCV can be used to detect the four sets of near-duplicate three images of the same view.

## Near-duplicate image detection
In this context, we have 12 fisheye images (180 deg vertical by 180 deg horziontal) of four adjacent views (left, front, right, back). Each view has three images taken at different exposure level (shutter speed varied), and we need to robustly divide the 12 images into those groups of three.  Since they are different in exposure, we cannot apply direct comparison methods like checksum.

There exists several methods to carry out near-duplicate image detection which all involve using a pre-processing step (e.g. histogram equalization), a pair-wise similarity metric of choice (e.g. pixel-wise or block-wise distance, edge or contour difference, norm-1 or norm-2 distances), and an association method.  These methods are combined and tuned based on the practical constraints of the problem.

In our approach, we apply histogram equalization to balance out multiple exposure levels, then a pixel-wise absolute difference. Pixel-wise difference is chosen because spatial difference is more important in our case of unaltered adjacent views. For cases like detecting transformed images people normally opt to local edge or contour difference.

This is followed by a postprocessing step of lower bound trimming to remove illumination difference noise, and image erosion to remove camera movement noise. This step will return a binary difference image of any two input images which could also be used for detecting ghosting problems in HDR merging.

#### Image Comparison using pixel-wise difference, with lower bound trimming and erosion
```python
def image_comparison(img1, img2, lower_bound=120):
    e1 = cv2.equalizeHist(img1)
    e2 = cv2.equalizeHist(img2)

    diff = cv2.absdiff(e1, e2)
    _, diff = cv2.threshold(diff, lower_bound, 255, cv2.THRESH_BINARY)
    kernel = np.ones((2, 2), np.uint8)
    diff = cv2.erode(diff, kernel, iterations=1)
    nonZero = cv2.countNonZero(diff)

    return nonZero
```

The last step in this approach is association, where pair-wise image difference is used to associate similar images into sets. This is the step that is normally specific and fine-tuned for different systems, and there are two common ways this can be implemented. It is similar to a common clustering problem, and you can either use distance-based (hierarchical clustering) or iterative centroid or group-based (k-means clustering). 

In a distance-based method, a distance threshold value is chosen (or learned empirically from data) to decide if two items belong to the same group. Once two images are matched, subsequent association steps will only need to be carried on one sample element of the group. This approach has the advantage of being fast (linear processing time), but its performance depends on how well the distance threshold is chosen, therefore this method is mostly used when data is well separated and speed is a requirement. 

In our case the number of images is small and the accuracy requirement is 100% of correct match (imagine a HDR image merged from three different images - it does not look pretty). Therefore we go for the second approach where we match all pair-wise combination of source images (66 matches for 12 images). The constraint on four sets of three images is used as the termination condition for our association step. Our iterative association consists of two steps, collecting tuples of top three matches (representing one group) and filtering out good matches (any match tuples that are collected exactly three times is a correct association). This is repeated until all elements are filtered.

#### Similar image association
```python
def find_groups(unselected, grouped, matches):
    freqs = {}
    m = {}
    for i in range(12):
        n = {}
        for j in range(12):
            if i == j: continue
            if j in unselected:
                n[j] = matches[(i, j)]
        m[i] = sorted(n.items(), key=operator.itemgetter(1))
        new_set = sorted([v[0] for v in m[i][:2]] + [i])
        freqs[tuple(new_set)] = 1 if tuple(new_set) not in freqs else freqs[tuple(new_set)] + 1

    for k, v in freqs.iteritems():
        if v == 3:
            for i in k:
                unselected.remove(i)
            grouped.append(k)
```

Once similar images are grouped into correct views, SNS-HDR is used to merge LDR images into HDR images (with tonemapping). PTGui is then called to stitch the four merged HDR into one equirectangular panorama.


#### Normal Panorama
![PTGui HDR](/public/images/ptgui2.png)

#### HDR Panorama done right
![SNS-HDR](/public/images/snshdr2.png)

Here is list of randomly selected panoramas from Oyster compared to Google
[Trump Soho from Google.com](https://www.google.com/maps/@40.7257012,-74.0055927,3a,75y,266.18h,79.62t/data=!3m8!1e1!3m6!1s-H0_jyN7iLq8%2FVyv1THyMdGI%2FAAAAAAABYto%2FsG9LtMP-Z8sO0bOxSsn8PxlxjU9h35N_gCLIB!2e4!3e11!6s%2F%2Flh5.googleusercontent.com%2F-H0_jyN7iLq8%2FVyv1THyMdGI%2FAAAAAAABYto%2FsG9LtMP-Z8sO0bOxSsn8PxlxjU9h35N_gCLIB%2Fw203-h101-n-k-no%2F!7i8704!8i4352!6m1!1e1)
[Trump Soho from Oyster.com](https://www.oyster.com/new-york-city/hotels/trump-soho-new-york/all-tours/lobby--v24780/)

[Refinery Hotel from Google.com](https://www.google.com/maps/@40.752632,-73.9850296,3a,75y,141.78h,65.16t/data=!3m7!1e1!3m5!1sPZRAzKwZYbgAAAQpkog1Jw!2e0!3e2!7i13312!8i6656!6m1!1e1)

[Refinery Hotel from Oyster.com](https://www.oyster.com/new-york-city/hotels/the-refinery-hotel/all-tours/parker-and-quinn-restaurant--v19455/)


[Holiday Inn Resort Montego Bay on Google.com](https://www.google.com/maps/place/Holiday+Inn+Resort+Montego+Bay/@18.5230122,-77.8590851,3a,75y,180.42h,90.31t/data=!3m8!1e1!3m6!1s-ThDGB2m55gk%2FVEaS54n2W5I%2FAAAAAAACsyA%2FzPC7j55CgeYtXzXZ4qSHhqhjIDO617C5wCLIB!2e4!3e11!6s%2F%2Flh6.googleusercontent.com%2F-ThDGB2m55gk%2FVEaS54n2W5I%2FAAAAAAACsyA%2FzPC7j55CgeYtXzXZ4qSHhqhjIDO617C5wCLIB%2Fw203-h101-n-k-no%2F!7i3584!8i1792!4m12!1m6!3m5!1s0x8eda29428da7fc87:0x654d78ef04bd3848!2sHoliday+Inn+Resort+Montego+Bay!8m2!3d18.522744!4d-77.857566!3m4!1s0x8eda29428da7fc87:0x654d78ef04bd3848!8m2!3d18.522744!4d-77.857566!6m1!1e1)

[Holiday Inn Resort Montego Bay on Oyster.com](https://www.oyster.com/jamaica/hotels/holiday-inn-resort-montego-bay/all-panoramas/beach--v115325/)


[Sensatori Jamaica by Karisma on Google.com](https://www.google.com/maps/place/Azul+Sensatori+Negril/@18.3312721,-78.3373032,3a,75y,166.33h,80.87t/data=!3m8!1e1!3m6!1s-nL2Lp-lHYU0%2FV3VY_yUYSWI%2FAAAAAAAADkU%2Fsbt2HfQiQVAb7naC0QBLc7Lzf70ifVa_ACLIB!2e4!3e11!6s%2F%2Flh6.googleusercontent.com%2F-nL2Lp-lHYU0%2FV3VY_yUYSWI%2FAAAAAAAADkU%2Fsbt2HfQiQVAb7naC0QBLc7Lzf70ifVa_ACLIB%2Fw234-h116-n-k-no%2F!7i6325!8i3162!4m12!1m6!3m5!1s0x8ed90d9910af296b:0x2dd5eafcd573a170!2sAzul+Sensatori+Negril!8m2!3d18.2791209!4d-78.346129!3m4!1s0x8ed90d9910af296b:0x2dd5eafcd573a170!8m2!3d18.2791209!4d-78.346129!6m1!1e1)

[Sensatori Jamaica by Karisma on Oyster.com](https://www.oyster.com/jamaica/hotels/sensatori-jamaica-by-karisma/all-panoramas/grounds--v119425/)

In this post, we have presented our approach to generating HDR panorama at large scale using available packages like DNGConverter, DCRAW, SNS-HDR, PTGui, and with the help from Computer Vision techniques with OpenCV. Please feel free to visit our website [Oyster](https://www.oyster.com) to see our rich collection of hotel panoramas all around the world. Also, please stay tuned for part 2 and 3 of this Computer Vision series, where we will show you how virtual tour can be generated (again fully automated at large scale) from a set of panoramas, and how smart features like mini-maps can be added to your tour to improve user experience.

### About the author:
Tuan Thi is a Senior Software Engineer in Computer Vision at [Oyster](https://www.oyster.com).com, part of Smarter Travel Media Group, at TripAdvisor. He finished his PhD in Computer Vision and Machine Learning in 2011.  Before joining TripAdvisor, he was a research engineer and computer vision scientist at Canon Research and Placemeter Ltd. with various international publications and patents in the field of local features, structured learning and deep learning.


