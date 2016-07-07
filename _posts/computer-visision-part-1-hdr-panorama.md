---
title: "Near-duplicate image detection and HDR panorama at large scale"
author: Tuan
layout: post
permalink: /computer-visision-part-1-hdr-panorama/
disqus_page_identifier: computer-visision-part-1-hdr-panorama
published: false
---

![HDR Panorama](/public/images/cover.png)

Here at [Oyster](https://www.oyster.com), we are one of the leading sites who provide comprehensive photographic reviews for hotel travel. One key component of our imagery database is panorama imaging, produced at high quality and large scale. In this three-part series, we will be looking at the Computer Vision work that has been part of our panorama pipeline. Specifically, in this first part of the series, we will introduce our automated pipeline for generating High Dynamic Range (HDR) panorama.

## HDR Panorama
[Panorama images](https://www.[Oyster](https://www.oyster.com).com/jamaica/hotels/moon-palace-jamaica-grande/all-panoramas/beach--v115415/) at [Oyster](https://www.oyster.com) has full angle range with 180 degree vertically and 360 degree horizontally, they provide imersive experience for viewers to explore the ubiquitous view of the venue, whethere it is outside at the pool, on rooftop or inside a hotel room. Panorama images now become a trending and to-have media type for most image-oriented website. Meanwhile, HDR imaging is a common technique to produce a greater dynamic range of luminosity than standard digital imaging, especially useful for panorama imaging where an evenly distributed look will greatly improve panorama quality. HDR imaging is normally achieved by merging multiple low-dynamic-range photographs. We use [PTGui](http://www.ptgui.com), a stitching software, to carry out batch stitching of 12 fisheye images (180 degree x 180 degree) of the 4 different views (left, right, front, back), 3 images of different exposure for reach view, the stitching process will return one equirectangular panorama.

#### Raw fisheye images
![Fisheyes images](/public/images/fisheyes.png)

While PTGui is a good choice for image stitching, its HDR quality is not the best available, and people normally use alternative tools for HDR merging. [SNS-HDR](http://www.sns-hdr.com) is the package used by [Oyster](https://www.oyster.com), thanks to its support on batch processing, its excellent HDR quality and its adequate deghosting support compared to PTGui.

#### PTGui HDR
![PTGui HDR](/public/images/ptgui1.png)

#### SNS-HDR
![SNS-HDR](/public/images/snshdr1.png)

The tricky part for using SNS-HDR as a batch tool is its limited support on file format input and the auto-grouping of images of the same view, and that is where Computer Vision comes into place.

The first problem on accepted image input, SNS-HDR works with RAW files, but it works especially best with converted raw DNG format  (using DNGConverter), compared to two common formats CR2 or NEF.

#### One possible artifact with SNS-HDR on original raw files
![SNS-HDR RAW](/public/images/raw.png)

#### Stable SNS-HDR merge on dng files
![SNS-HDR DNG](/public/images/dng.png)

The second problem of using SNS-HDR batch processing is to figure out the three images of the same view, which is done using OpenCV  (Python binding with Numpy), one of the most comprehensive Computer Vision libraries to dates. Since OpenCV does not work directly with raw files (CR2, NEF, DNG), we use DCRAW to convert DNG to TIFF, before OpenCV can be used to detect 4 sets of near-duplicate 3 images of the same view.

## Near-duplicate image detection
In this context, we have 12 fisheye images (180 vert by 180 horz) of 4 adjacent views (left, front, right, back), each view has 3 images taken at different exposure level (shutter speed varied). With no assumption on shooting times (all 12 images could be taken at arbitrary time), and no assumption on exposure level (our camera is auto-metered at each view so shutter times might not be the same at each angle), and the 3 images of the same view are different in exposure so we cannot apply direct comparison methods like checksum, we finally resort to Computer Vision to robustly arrange 12 source images into 3 view groups.

There exists several methods to carry out near-duplicate image detection, but they all involve using a pre-processing step (e.g. histogram equalization), a pair-wise similarity metric of choice (e.g. pixel-wise or block-wise distance, edge or contour difference, norm-1 or norm-2 distances), and an association method, designed based on practical constraints of the problem.

In our approach, we apply histogram equalization to balance out multiple exposure levels, pixel-wise absolute difference (pixel-wise difference is chosen because spatial difference is more important in our case of unaltered adjacent views, for cases like detecting transformed images people normally opt to local edge or contour difference), followed by a postprocessing step of lower bound trimming (to remove illumination difference noise) and image erosion (to remove camera movement noise). This step will return a binary difference image of any two input images, which could also be used for detecting ghosting problem in HDR merging.

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

The last step in this approach is association, where pair-wise image difference is used to associate similar images into sets, this is the step that is normally specific and fine-tuned for different system, and there are two common ways this can be implemented, similar to a common clustering problem, either by distance-based (hierarchical clustering) or by iterative centroid or group-based (k-means clustering). In a distance-based method, a distance threshold value is chosen (or learned empirically from data) to decide if two items belong to the same group, once two images are matched, subsequent association steps will only need to be carried on one sample element of the group, this approach has the advantage of being fast (linear processing time), but its performance depends on how well the distance threshold is chosen, therefore this method is mostly used when data is well separated and speed is a requirement. In our case, the number of images is small, 12, and the accuracy requirement is 100% of correct match (imagine a HDR image merged from 3 different images, yep it does not look pretty), therefore we go for the second approach where we match all pair-wise combination of source images (66 matches for 12 images), the constraint on 4 sets of 3 images is used as the termination condition for our association step. Our iterative association consists of two steps, collecting tuples of top 3 matches (representing one group) and filtering out good matches (any match tuples that are collected exactly 3 times is a correct association), and repeat until all elements are filtered.

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

Once similar images are grouped into correct views, SNS-HDR is used to merge LDR images into HDR images (with tonemapping), and PTGui is called to stitch the 4 merged HDR into one equirectangular panorama.


#### Normal Panorama
![PTGui HDR](/public/images/ptgui2.png)

#### HDR Panorama done right
![SNS-HDR](/public/images/snshdr2.png)

In this post, we have presented our approach to generating HDR panorama at large scale, using available packages like DNGConverter, DCRAW, SNS-HDR, PTGui and with the help from Computer Vision techniques with OpenCV. Please feel free to visit our website [Oyster](https://www.oyster.com) to see our rich collection of hotel panoramas all around the world. Also, please stay tuned for part 2 and 3 of this Computer Vision series, where we will show you how virtual tour can be generated (again fully automated at large scale) from a set of panoramas, and how smart features like mini-maps can be added to your tour to improve user experience.

### About the author:
Tuan Thi is a Senior Software Engineer in Computer Vision at [Oyster](https://www.oyster.com).com, part of Smarter Travel Media Group, at TripAdvisor. He finished his PhD in Computer Vision and Machine Learning in 2011, before joining TripAdvisor, he was a research engineer and computer vision scientist at Canon Research and Placemeter Ltd., with various international publications and patents in the field of local features, structured learning and deep learning.


