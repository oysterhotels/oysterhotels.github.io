---
title: Oyster Shots on the Front End
author: Alex
layout: post
permalink: /oyster-shots-on-the-front-end/
wordpress_post_id: 80
dsq_thread_id:
  - 527842039
categories:
  - Uncategorized
---
In [our last post][1] Ben brought you up to speed on some of the inner workings of our latest addition to the site, Oyster Shots. Building the user interface for this new feature presented its own set of challenges. With Oyster Shots we wanted to create as immersive an experience as possible, allowing users to navigate our mountains of photographic content in a new and fun way.

## Photo Sizing

One of the main goals we had with Oyster Shots was to provide the best photo-browsing experience to users with a wide range of screen resolutions: ranging from nerds like us that have huge monitors, to laptop displays, and even to those desktop displays still kicking around with 1024&#215;768 resolution. We do this both on the [photo detail view][2] and the [results view][3] using a couple of different techniques:

### The Client-Side Part

First off, we scale the photos as you resize your browser window using a combination of CSS and JavaScript. For the photo detail view, we have a fluid layout where the sidebar has a fixed width and the photo expands to fill as much space as is available. For the result view we used percentage measurements to define the width of image columns. At most window sizes four columns of photos looked pretty great, but at some smaller screen resolutions, the photos got too small. To address that we used JavaScript to change the number of columns based on window size: if the window size is less than 1410 pixels, you get three columns; if it&#8217;s more you get four. To keep the spacing between photos consistent regardless of size, we used an outer container with its width set by percentage and an inner box with fixed margins in pixels.

<pre>.photo-result-container {
    display: inline-block;
    max-width: 610px;
    min-width: 245px;
    vertical-align: top;
    /* percentage width to fit four photo results per row */
    width: 25%;
}
.photo-result {
    background-color: #fff;
    border: 1px solid #ccc;
    /* margins on the inside keep the spacing consistent,
    and don't mess with the width of the container */
    margin: 0 16px 44px 0;
    padding: 8px 0;
    position: relative;
    vertical-align: top;
}</pre>

We also use CSS and JavaScript to resize the photos themselves. Setting the image&#8217;s width to 100% and leaving the height at &#8220;auto&#8221; means the image will fill its container horizontally and maintain its original aspect ratio. Obviously we don&#8217;t want to use the same image file for every possible screen size: using a huge image and scaling it down would add undue page weight, while using a small image and scaling it up would look pretty terrible.

<div id="attachment_122" class="wp-caption aligncenter">
  <a href="http://www.oyster.com/shots/?qa=location%3Ajamaica+beach#image=103161"><img class="size-full wp-image-122" title="pdp-compare" src="http://tech.oyster.com/wp-content/uploads/2011/07/pdp-compare.jpg" alt="A Photo Detail Page for high-resolution display, and low-resolution" /></a> 
  
  <p class="wp-caption-text">
    A Photo Detail Page for high-resolution display, and low-resolution
  </p>
</div>

We have a defined set of image sizes that are available for all of our photos. So using a custom jQuery plugin, we check the size of the images when the browser window resizes to see if the image file needs to be replaced. So if the new size of the image is larger than its native resolution, we swap the **src** attribute with a larger image so that the scaling is cleaner. By measuring the direction a user is scaling his or her browser, either growing or shrinking the window, we can start to load new image sizes early.

Conventional wisdom seems to be that you&#8217;re not supposed to scale images in the browser. The two most oft-cited reasons are:

  1. You serve larger images than are needed: We dealt with this by having multiple sizes and serving the one that most closely matches the current display size.
  2. Browsers don&#8217;t do a good job of scaling images: We found that not to be the case. Rescaling images in a modern browser to a size close to its native resolution yields results that are quite acceptable.

<div id="attachment_99" class="wp-caption aligncenter">
  <img class="size-full wp-image-99" title="Photoshop/Browser Scaling Comparison" src="http://tech.oyster.com/wp-content/uploads/2011/07/size-compare.jpg" alt="One of these images was rescaled in Photoshop using bicubic resampling, one was rescaled in Internet Explorer.  Can you tell which is which?" usemap="#size-compare" /> 
  
  <p class="wp-caption-text">
    One of these images was rescaled in Photoshop using bicubic resampling, one was rescaled in Internet Explorer. Can you tell which is which?
  </p>
</div>

<map name="size-compare">
  <area shape="rect" coords="0, 0, 360, 253" href="http://tech.oyster.com/wp-content/uploads/2011/07/left-big.jpg" target="_blank" />
  
  <area shape="rect" coords="361, 0, 720, 253" href="http://tech.oyster.com/wp-content/uploads/2011/07/right-big.jpg" target="_blank" />
</map>

### The Server-Side Part

Changing image sizes as a user resizes his or her browser window is one thing, but ideally we&#8217;d like to serve up the optimal image size on page load as well. To do this, we set a cookie with the user&#8217;s display size. This cookie is written on page load and every time the browser window is resized. We use this cookie on a number of pages that have fluid-sized images, including the Oyster Shots results page, to determine how large an image will be displayed when the HTML is loaded. Starting with the dimensions of the browser window, we do a few calculations: subtracting the height of the page header, the width of the sidebar and so on, to determine how much page real estate the photo will have to display. We find the closest available image size, and generate the HTML on the server side to use that image. That way when the resizing plugin on the client side takes over, the image does not need to be loaded again. Of course this can fail if a user is browsing in multiple windows, or clears cookies, for instance, but that is a minority of cases, and the image would just be reloaded once the client side sizing logic kicks in.

## Quick Browsing

Another goal we had was to be able to browse photos quickly. From the Oyster Shots results page, once you click a photo to get to the detail view, you can start browsing through the results by clicking the arrows at the top right, clicking the photo itself, or using the left and right arrow keys on your keyboard. You may notice how quick it is to navigate from one photo to the next. This is because the detail pages are all loaded via Ajax, which allowed us to make a few key performance improvements.

The advantages of using Ajax to update only a portion of the page content, rather than causing a complete refresh, are well-known. What we did to go beyond that was to load multiple photo detail pages in a single request. The HTML for a dozen photo pages is stored in memory. That way, navigating to the next or previous photo in a series usually only requires reading a JavaScript variable to get the HTML content.

Loading multiple photo pages at once also allows us to preload images as well. When you&#8217;re looking at a photo detail page, we can look at the source of the next or previous image in the series and start to load it ahead of time.

The photo pages are stored in memory as objects that have a handful of properties, including a string that is the HTML for the page itself. Storing and manipulating a large number of photo pages in memory called for a couple of unique solutions, the first of which was a basic issue of organization. In the many ways that Oyster Shots works with these objects, we will at times need to access a particular photo page arbitrarily according to the unique image id, the id of the hotel that image belongs to, or the index (the order in which the image appears in the result set).

To allow for this flexibility, we maintain a single container object that stores the page objects and is keyed by the index, and two supplemental indexes where page objects are keyed separately by image id and hotel id. We used an object rather than an array for the primary collection.

<pre>//set up the main collection and the two supplemental indexes:
var images = {};  //main collection
var imageKeys = {}; //indexed by image id
var hotelIdImageKeys = {}; //indexed by hotel id

//update the indexes when we add an image to the collection:
function addImage(index, imageId, imageObj) {
    images[index] = imageObj;
    imageKeys[imageId] = index;
    var hotelId = imageObj.hotelId;
    if(!hotelIdImageKeys[hotelId])
        hotelIdImageKeys[hotelId] = [index];
    else
        hotelIdImageKeys[hotelId].push(index);
}

//now looking up image pages by image id or hotel id is easy:
function getImageById(imageId) {
    if(!imageKeys.hasOwnProperty(imageId))
        return false;
    var index = imageKeys[imageId];
    return images[index];
}
function getImagesByHotelId(hotelId) {
    if(!hotelIdImageKeys.hasOwnProperty(hotelId))
        return false;
    var hotelImages = [];
    var indexes = hotelIdImageKeys[hotelId];
    for(var i = 0, len = indexes.length; i &lt; len; i++)
        hotelImages.push(images[indexes[i]]);
    return hotelImages;
}</pre>

Maintaining these indexes in this way allows us to quickly find a particular page object by its id, its hotel id, or its position in the search results with just a couple of object attribute lookups rather than looping through the entire set of images.

Another issue that arose is how to change the stored HTML, if needed, once it&#8217;s been retrieved from the server. We store the HTML as strings rather than DOM fragments because string manipulation is faster than DOM manipulation (even in a fragment) and strings take up far less memory than DOM fragments. Unfortunately that means that all of jQuery&#8217;s handy DOM methods are off the table for working with this content. To get around this, we marked sections of the page that would need to be changed on the fly with HTML comments. Then it was just a simple find and replace operation with no DOM manipulation required.

<pre>/*
    the markers are HTML comments like:
    &lt;!--begin section--&gt;
    and
    &lt;!--end section--&gt;
*/
function replaceHtmlAtMarkers(html, replace, beginMarker, endMarker) {
    var begin = html.indexOf(beginMarker);
    var end = html.indexOf(endMarker);
    end += endMarker.length;
    if(begin === -1 || end === -1)
        return html;
    var original = html.substring(begin, end);
    html = html.replace(original, replace);
    return html;
}</pre>

There were tons of problems we had to solve while building out the front end for Oyster Shots, and these are merely examples of a few of them. If you&#8217;re finding our new blog useful or interesting, leave a comment and let us know!

<a name="answer"></a>  
*Turn your monitor upside-down or stand on your head to read the answer:  
<span style="color: #ff0000;">ɹǝɹoןdxǝ ʇǝuɹǝʇuı uı pǝzısǝɹ sɐʍ ʇɟǝן ǝɥʇ uo ǝbɐɯı ǝɥʇ :ɹǝʍsuɐ</span>

 [1]: http://tech.oyster.com/how-our-photo-search-engine-really-works/
 [2]: http://www.oyster.com/shots/?qa=infinity-pool&sort=Sd#image=416766 "Oyster Shots Photo Detail View"
 [3]: http://www.oyster.com/shots/?qa=infinity-pool&sort=Sd#q=Infinity+Pool "Oyster Shots Results Page"