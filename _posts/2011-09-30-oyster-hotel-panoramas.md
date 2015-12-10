---
title: Oyster Hotel Panoramas
author: Bret
layout: post
permalink: /oyster-hotel-panoramas/
dsq_thread_id:
  - 534335471
categories:
  - Uncategorized
---
Here at Oyster we&#8217;re all about photos. We like photos of [people][1], [pools][2], [property][3] and [power outlets][4]. The more photos the better as our goal is to give you a real and ideally complete picture of the hotels we cover.  A single photo &#8211; while worth a thousand words &#8211; will generally show you a small window of about 40° horizontally and 27° vertically. What if we could give you a 360° view of the hotel? A panorama adds a lot of perspective and helps create a better sense of the space. We weren&#8217;t sure what the results would be but we decided to purchase a couple [Gigapan][5] pano heads and send them out to our photographers and see what we could make of it.

[<img class="alignnone size-large wp-image-231" title="bell_rock_inn_gigastitch" src="http://tech.oyster.com/wp-content/uploads/2011/09/bell_rock_inn_gigastitch1-1024x203.jpg" alt=""   />][6]

And we made this. Well that&#8217;s kind of cool, not very interactive but cool. When looking for some panoramic software we didn&#8217;t really have a big list of requirements. We wanted something that would work well and look good. We didn&#8217;t want any QuickTime or Java implementations, sticking to Flash and hopefully some in-progress HTML5/js solution. The first program we investigated was [Pano2VR][7]. Pano2VR definitely did the job, but when investigating how we might handle UI changes down the line it looked as though we&#8217;d have to generate each panorama over again. As an engineer this seemed like a less than ideal solution.



While not writing it off I continued to search for panorama software. I came across this website dedicated to panoramas, [Panoramas.dk][8], and the panoramas I was looking at felt smoother. Digging a bit I found out they use [krPano][9]. After giving it a guick run-through it became obvious that this was going to be our solution. It ran off a suite of command-line tools and configurable template files, allowing you to separate your UI into an XML file, while packaging up the rest of your panorama into a swf. Now if we decide to change anything in the UI we just have to worry about modifying that one file &#8211; perfect!



Taking a look at the pano above you&#8217;ll see that we&#8217;re almost there. You might&#8217;ve noticed a rather visible seam where the left and right ends of the photo meet. Up until now we&#8217;d just been working out what panorama software we would use and it was time to get that stitching quality up. The software package that came with Gigapan wasn&#8217;t cutting it. Sure we could pass it off to our photo editors to try and do something about it, but the more streamlined this process was the better. [GigaPan Stitch][10] was out and [AcroPano][11] I found barely usable having to manually set points of similarity between each image as it was added. I did not get far enough to see what a stitch would look like. I finally stumbled upon [Microsoft Research Image Composite Editor][12] or MR ICE for short. Not only did ICE get rid of the seam but it also did a far superior job of selecting photos when blending moving objects. If you look closely at the photo below you can see a minivan on the road near the middle of the image that appears to be vanishing, while the ICE stitched photo correctly composites two images without the moving object.

[<img class="alignnone size-large wp-image-246" title="bell_rock_compare" src="http://tech.oyster.com/wp-content/uploads/2011/09/bell_rock_compare1-1024x440.jpg" alt=""   />][13]

So far we&#8217;re just working with individual panoramas for each hotel, but one great feature of krPano is the ability to create [virtual tours][14]. Though not something we&#8217;re currently working on we do have the capability with krPano to create a sort of walkthrough of a hotel. There are also some neat features with hotspots and javascript callbacks that could create some pretty interesting experiences. Take a look at the first batch of panoramas:

[Bell Rock Inn][15]

[L&#8217;Auberge de Sedona][16]

[Best Western Plus Arroyo Roble Hotel & Creekside Villas][17]

 [1]: http://www.oyster.com/las-vegas/hotels/wynn-las-vegas/photos/restaurants-bars-wynn-las-vegas-v143836/ "People"
 [2]: http://www.oyster.com/shots/?qa=pool#q=Pool "Pools"
 [3]: http://www.oyster.com/shots/?qa=hotel#q=Hotel "Hotels"
 [4]: http://www.oyster.com/dominican-republic/hotels/catalonia-royal-bavaro/photos/junior-suite-catalonia-royal-bavaro-v18337/ "Power Outlet"
 [5]: http://www.gigapansystems.com/
 [6]: http://tech.oyster.com/wp-content/uploads/2011/09/bell_rock_inn_gigastitch1.jpg
 [7]: http://gardengnomesoftware.com/pano2vr.php
 [8]: http://panoramas.dk/
 [9]: http://krpano.com/
 [10]: http://gigapansystems.com/gigapan-products/gigapan-software/gigapan-stitcher-software-information.html
 [11]: http://www.acropano.com/
 [12]: http://research.microsoft.com/en-us/um/redmond/groups/ivm/ICE/
 [13]: http://tech.oyster.com/wp-content/uploads/2011/09/bell_rock_compare1.jpg
 [14]: http://krpano.com/tours/weingut/
 [15]: http://www.oyster.com/sedona/hotels/bell-rock-inn/panoramas/entrance-1-4/
 [16]: http://www.oyster.com/sedona/hotels/lauberge-de-sedona/panoramas/entrance-3-3/
 [17]: http://www.oyster.com/sedona/hotels/best-western-plus-arroyo-roble-hotel-and-creekside-villas/panoramas/pool-2-4/