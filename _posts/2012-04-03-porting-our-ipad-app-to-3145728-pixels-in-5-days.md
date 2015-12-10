---
title: Porting our iPad app to 3,145,728 pixels in 5 days
author: Chris
layout: post
permalink: /porting-our-ipad-app-to-3145728-pixels-in-5-days/
dsq_thread_id:
  - 642275861
categories:
  - Uncategorized
---
In March 2012, Apple announced and released the new iPad with a Retina display. There was only a short time between the announcement and when the device arrived in stores. This left a very short time for developers to get a working app out the door. And we wanted to take advantage of the wave of publicity, so we tried to get our app ready the same day the devices shipped.

There had been speculation for months that the next iPad would have a Retina display. But we didn’t want to start working on a Retina version without official specs. It wasn’t even confirmed that Apple would release a new iPad yet. Finally, on March 7th, Apple announced the new iPad with a Retina display. The released the official specs, and later that evening they released Xcode with a Retina simulator. The device would ship on March 16th. We had just over a week!

## Porting to Retina

The biggest improvement that we cared about was the &#8220;Retina display&#8221; &#8212; Apple&#8217;s name for a screen resolution where the pixels are so small the average human eye can’t see them. This means that angled lines look smoother and more natural. The display has twice the resolution in both the vertical and horizontal directions. But the size of the screen remained the same, so each pixel is one quarter the size.

To minimize issues with the transition to Retina, Apple&#8217;s iOS APIs measure things in what they call &#8220;points&#8221;, rather than straight pixels. In the old iPads one point represents one pixel, in the new iPad one point represents four pixels.

<div style="text-align: center;">
  <div style="display: inline-block; margin-right: 1px;">
    <div id="attachment_686" class="wp-caption alignnone" style="width: 360px">
      <img class="size-full wp-image-686" title="Close-up on a non-Retina device" src="http://tech.oyster.com/wp-content/uploads/2012/04/paris-2c.png" alt="Close-up on a non-Retina device" width="360" height="250" /> 
      
      <p class="wp-caption-text">
        Close-up on a non-Retina device
      </p>
    </div>
  </div>
  
  <div style="display: inline-block;">
    <div id="attachment_685" class="wp-caption alignnone" style="width: 360px">
      <img class="size-full wp-image-685" title="The same image on a Retina display" src="http://tech.oyster.com/wp-content/uploads/2012/04/paris-3a.png" alt="The same image on a Retina display" width="360" height="250" /> 
      
      <p class="wp-caption-text">
        The same image on a Retina display
      </p>
    </div>
  </div>
</div>

This makes it relatively simple to upgrade an app. Most of the work actually falls to the artists to generate larger sized images for UI elements. You bundle the new Retina assets in the app, and you don&#8217;t even have to modify the code &#8212; if the old asset is called “foo.png”, then name the new one “foo@2x.png”, and everything just magically works. But our app is photo heavy and relies on pulling down a lot of images off the web. This took some work to fix.

## Huge photo sizes

The biggest amount of work involved making sure we request the right size photo. We have 31 different image sizes of each of our 750,000 hotel photos, and they are not all double the previous size. We chose to solve this in the client &#8212; if the app detects it&#8217;s running on a Retina device, it requests the Retina-ready photo.

The one challenge I ran into was some button icons that were being served by our web server. The original buttons were 16&#215;16. We already had some 35&#215;35 versions, and so the artist decided to use these, instead of creating new 32&#215;32 ones. Since they were going on a button I figured it would be ok. Until I saw them &#8230; and they were just over twice as big as they should have been. The problem was iOS doubling the image size automatically. I didn’t want that &#8212; I wanted to use the same magic as UIImage does with &#8220;@2x&#8221; in the filename. The magic is in the &#8220;scale&#8221; attribute of the UIImage object. Set scale = 2 and the image is no longer doubled. I added this code to our image downloader, to mimic the &#8220;@2x&#8221; iOS supports.

<pre>UIImage *image = [[UIImage alloc] initWithData:imageData];
if ([url hasSuffix:@"@2x"])
{
    image = [UIImage imageWithCGIImage:[image CGImage] scale:2.0 orientation:UIImageOrientationUp];
}</pre>

## But let&#8217;s not quadruple the file size

We did have to create one new size for our photos &#8212; for the full-screen 2048&#215;1536 images. This was no big deal, although it did take a few days to churn through all 750,000 hotel photos. But 2048&#215;1536 is one big image, and contains a lot of data. With over 3 million pixels the image file sizes were four times as big as before. We didn’t want our app to become slower due to image load time, so we reduced the image quality on the larger sized images. We figured the Retina display would hide some of the JPEG aritifacts, and settled on a 65% quality level. The resulting file size was only about 1.5 times as big as before (instead of 4 times).

We worked fast and furious the last few days, to hit our goal of uploading to the app store on Friday. There were a few minor layout issues that had to be tweaked. While the simulator was nice (although I could only see about half of it on my 21 inch 1600&#215;1200 monitor!) we had to test the program on a real device. But like everyone else we had to wait until they were released on Friday. After receiving the new iPad and verifying everything worked, we packed up the app and shipped it off .

## Reviewed by Apple in six hours

Our app was verified by Apple in just six short hours! Our Retina version was available to the general public late Friday evening. Which means folks could see all of our thousands of awesome photos in glorious Retina display right from the very beginning. Also, being one of the first &#8220;retinized&#8221; apps, we made it on several of Apple’s top lists. This boosted our download rate immensely. It was worth the effort to take advantage of the small window Apple gave us, and makes our app that much better.