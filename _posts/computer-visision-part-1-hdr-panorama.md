---
title: "Near-duplicate image detection and HDR panorama at large scale"
author: Tuan
layout: post
permalink: /computer-visision-part-1-hdr-panorama/
disqus_page_identifier: computer-visision-part-1-hdr-panorama
published: false
---

![HDR Panorama](/public/images/cover.png)

Here at Oyster.com, we are one of the leading sites who provide comprehensive photographic reviews for hotel travel. One key component of our imagery database is panorama imaging, produced at high quality and large scale. In this three-part series, we will be looking at the Computer Vision work that has been part of our panorama pipeline. Specifically, in this first part of the series, we will introduce our automated pipeline for generating High Dynamic Range (HDR) panorama.

## HDR Panorama
Panorama images (examples) at Oyster.com has full angle range with 180 degree vertically and 360 degree horizontally. HDR imaging (example) is a common technique to produce a greater dynamic range of luminosity than standard digital imaging, normally achieved by merging multiple low-dynamic-range photographs. We use PTGui (screenshot) to carry out batch stitching of 12 fisheye images (180 degree x 180 degree) of the 4 different views (left, right, front, back), three images of different exposure for reach view, the stitching process will return one equirectangular panorama (example).

While PTGui is a good choice for image stitching, its HDR quality is not the best available in this area and people normally use an alternative tool for HDR merging. SNS-HDR is the package used by Oyster, thanks to its support on batch processing, its excellent HDR quality and its adequate deghosting support. The tricky part for using SNS-HDR as a batch tool is its limited support on file format input and the auto-grouping of images of the same view, and that is where Computer Vision comes into place.

The first problem on accepted image input, SNS-HDR works with RAW files, but it works especially best with converted raw DNG format  (using dngconverter from Adobe), compared to two common formats CR2 or NEF (examples).

The second problem of using SNS-HDR batch processing is to figure out the three images of the same view, which is done using OpenCV, one of the most comprehensive Computer Vision libraries to dates (link and numpy's). Since OpenCV does not work directly with raw files (CR2, NEF, DNG), we use DCRAW to convert DNG to TIFF, before OpenCV can be used to detect 4 sets of near-duplicate 3 images of the same view.

## Near-duplicate image detection
In this context, we have 12 fisheye images (180 vert by 180 horz) of 4 adjacent views (left, front, right, back), each view has 3 images taken at different exposure level (shutter speed varied). With no assumption on shooting times (all 12 images could be taken at arbitrary time), and no assumption on exposure level (our camera is auto-metered at each view so shutter times might not be the same at each angle), and the 3 images of the same view are different in exposure so we cannot apply direct comparison methods like checksum, we finally resort to Computer Vision to robustly arrange 12 source images into 3 view groups.

There exists several methods to carry out near-duplicate image detection, but they all involve using a pre-processing step (e.g. histogram equalization), a pair-wise similarity metric of choice (e.g. pixel-wise or block-wise distance, edge or contour difference, norm-1 or norm-2 distances), and an association method, designed based on practical constraints of the problem.

In our approach, we apply histogram equalization to balance out multiple exposure levels, pixel-wise absolute difference (pixel-wise difference is chosen because spatial difference is more important in our case of unaltered adjacent views, for cases like detecting transformed images people normally opt to local edge or contour difference), followed by a postprocessing step of lower bound trimming (to remove illumination difference noise) and image erosion (to remove camera movement noise). This step will return a binary difference image of any two input images, which could also be used for detecting ghosting problem in HDR merging.

[CODE]

The last step in this approach is association, where pair-wise image difference is used to associate similar images into sets, this is the step that is normally specific and fine-tuned for different system, and there are two common ways this can be implemented, similar to a common clustering problem, either by distance-based (hierarchical clustering) or by iterative centroid or group-based (k-means clustering). In a distance-based method, a distance threshold value is chosen (or learned empirically from data) to decide if two items belong to the same group, once two images are matched, subsequent association steps will only need to be carried on one sample element of the group, this approach has the advantage of being fast (linear processing time), but its performance depends on how well the distance threshold is chosen, therefore this method is mostly used when data is well separated and speed is a requirement. In our case, the number of images is small, 12, and the accuracy requirement is 100% of correct match (imagine a HDR image merged from 3 different images, yep it does not look pretty), therefore we go for the second approach where we match all pair-wise combination of source images (66 matches for 12 images), the constraint on 4 sets of 3 images is used as the termination condition for our association step. Our iterative association consists of two steps, collecting tuples of top 3 matches (representing one group) and filtering out good matches (any match tuples that are collected exactly 3 times is a correct association), and repeat until all elements are filtered.

[CODE]

Once similar images are grouped into correct views, SNS-HDR is used to merge LDR images into HDR images (with tonemapping), and PTGui is called to stitch the 4 merged HDR into one equirectangular panorama.

[SHOW RESULTS]

In this post, we have presented our approach to generating HDR panorama at large scale, using available packages like DNGConverter, DCRAW, SNS-HDR, PTGui and with the help from Computer Vision techniques with OpenCV. Please feel free to visit our website https://www.oyster.com/ to see our rich collection of hotel panoramas all around the world. Also, please stay tuned for part 2 and 3 of this Computer Vision series, where we will show you how virtual tour can be generated (again fully automated at large scale) from a set of panoramas, and how smart features like mini-maps can be added to your tour to improve user experience.

About the author:
Tuan Thi is a Senior Software Engineer in Computer Vision at Oyster.com, part of Smarter Travel Media Group, at TripAdvisor. He finished his PhD in Computer Vision and Machine Learning in 2011, before joining TripAdvisor, he was research engineer and computer vision scientist at Canon Research and Placemeter Ltd., with various international publications and patents in the field of local features, structured learning and deep learning.



In this post I'll be covering some tips on how to use React and jQuery together in the same UI.
Okay so first off you might be thinking "why would you want to do such a thing?" - in fact the idea
of trying to make React's declarative style live together with imperative jQuery DOM updates
may have you thinking something like [this](http://tech.oyster.com/public/images/ghostbusters.gif),
and for the most part, you wouldn't be wrong.

So first the "why." If you're starting a brand new, "greenfield" project and you want to use React,
then just do it. There's no good reason I can think of to mix-and-match React with jQuery or 
Mustache or whatever other DOM-helper/template library if you don't have to - just use React and everything will be cool
and you won't have to worry about it. If on the other hand, you have one of those "legacy" applications
that has "customers" and makes "money," and for some reason your boss is not into the idea of you
spending a few weeks rewriting the whole front end in React, but you still want in on that declarative
React goodness, you may have to figure out how to get React to play well with jQuery or something similar.

## Are you *sure* you can't rewrite it?
Let's say you have some piece of UI that gets rendered with jQuery, and you want to stick in some
new component written with React. Take a look at your jQuery rendering function. Are you just building up some
big string of HTML and sticking it in the DOM? Or using some kind of JavaScript template? If your rendering
is already reasonably functional, i.e. some data goes into your function and some HTML comes out (or gets
appended to the DOM or whatever) it will be pretty easy to just rewrite in React. So you should probably 
just do that and save yourself the inevitable hassle you'll have when something breaks and you have to debug it.

## Okay so you can't rewrite it
For whatever reason you've determined it's not practical to rewrite your jQuery code. Here is an important
caveat: I think using jQuery and React to manage updates to the *same* DOM elements is a bad idea.
React is really smart about figuring out how to update the DOM, but that only works if React is the only thing
doing the updates. So unless you can cleanly separate the DOM elements in your UI so that some *only* get
updated by React and others *only* get updated by jQuery, I wouldn't try it. 

So say for instance you're
going to render a product list with a React template, and then jQuery is going to add and remove CSS classes
to the list items, and then your React render function might get called again later. This is a bad idea. React will have no idea about the changes that jQuery has made. Some of React's efficiency comes from reusing DOM nodes on the page when things change, rather than always inserting or deleting nodes. If jQuery is making DOM changes that React doesn't know about, some node that gets reused might be in an unexpected state.

One further caveat is that you should only really consider this mixed approach if you're planning to
*eventually* replace jQuery rendering with React. Using both doesn't make sense long-term, but if you're
looking to gradually transition to React, you may have some parts of your UI using both for a while.

## Let's write some code
As an example we'll start by rendering a simple list of products with jQuery. We'll just show the name
for each product and a button to buy it. Then we'll get into replacing parts of the UI with React.

Our jQuery product list is pretty basic - it takes an array of products and inserts the list into
\#product-list-container. If the product list is updated, you just call ``productListJustJquery()`` again
and replace the whole list with a new list.

```javascript
var products = [
  {
    id: 1,
    name: 'Book',
    price: 15
  },
  {
    id: 2,
    name: 'Burrito',
    price: 8
  },
  {
    id: 3,
    name: 'Spaceship',
    price: 999999999
  },
  {
    id: 4,
    name: 'Dinosaur Bones',
    price: 5000000
  }
];

function buyProduct(productId) {
  // buy the product
}

/* -- Just jQuery -- */
function buyButtonJquery(product) {
  var button = $('<button class="buy-button">$' + product.price
    + '</button>');
  
  // handle click event
  $(button).on('click', function(event) {
    event.preventDefault();
    buyProduct(product.id);
  });

  return button;
}

function productListJustJquery(products, element) {
  var list = $('<ul class="product-list"></ul>');

  products.forEach(function(product) {
    var item = $('<li>' + product.name + '</li>');
    item.append(buyButtonJquery(product));
    list.append(item);
  });

  // replace the existing list if there is one
  var currentList = $(element).find('.product-list');
  if (currentList.length) {
    currentList.replaceWith(list);
  } else {
    $(element).append(list);
  }
}
```

## jQuery inside a React component
We'll start by replacing most of the product list with React but leaving the buy button in jQuery. 
This is simpler than the inverse - sticking React inside a jQuery UI - so we'll do it first.
The ``ProductListComponent`` is pretty straightforward:

```
var ProductListComponent = React.createClass({
  render: function(props) {
    return (
      <ul className="product-list">
        {this.props.products.map(function(product) {
          return <ProductComponent 
              key={product.id} 
              product={product} />
        })}
      </ul>
    );
  }
});
```

but in ``ProductComponent`` we need some extra
code to make the call to jQuery. We add an extra ``button-container`` element,
so that we have somewhere to put the jQuery DOM, and keep a reference to it.

```
render: function(props) {
  /* we need to keep a ref to the 
   * button-container so we can update it with jQuery
   */
  return (
    <li>
      {this.props.product.name}
      <span className="button-container" 
        ref="buttonContainer"></span>
    </li>
  );
}
```

## Life cycle methods
It's important to get familiar with the various React life cycle methods.
The relevant ones here are ``componentDidMount`` - which is called after the first render, and
``componentDidUpdate`` - which is called after subsequent renders. In each of these methods we just call
``renderBuyButton``, which uses our reference to the ``button-container`` DOM node to create a brand new buy
button with jQuery on each render.

```javascript
componentDidMount: function() {
  this.renderBuyButton();
},
componentDidUpdate: function() {
  this.renderBuyButton();
},
renderBuyButton: function() {
  // render the buy button with jQuery
  $(this.refs.buttonContainer).html(
    buyButtonJquery(this.props.product)
  );
}
```

Here's the complete ``ProductComponent``:

```
var ProductComponent = React.createClass({
  componentDidMount: function() {
    this.renderBuyButton();
  },
  componentDidUpdate: function() {
    this.renderBuyButton();
  },
  render: function(props) {
    /* we need to keep a ref to the 
     * button-container so we can update it with jQuery
     */
    return (
      <li>
        {this.props.product.name}
        <span className="button-container" 
          ref="buttonContainer"></span>
      </li>
    );
  },
  renderBuyButton: function() {
    // render the buy button with jQuery
    $(this.refs.buttonContainer).html(
      buyButtonJquery(this.props.product)
    );
  }
});
```

## React components inside jQuery
Now we're going to do it the other way and stick some React DOM inside our jQuery DOM. This is a little
trickier. We'll start with a ``BuyButtonComponent`` in React, there's not much to it:

```
var BuyButtonComponent = React.createClass({
  onClick: function(event) {
    buyProduct(this.props.product.id);
  },
  componentDidMount: function() {
    console.log('component did mount - stuff to clean up later');
  },
  render: function(props) {
    return (
      <button className="buy-button" 
        onClick={this.onClick}>{this.props.product.price}
      </button>
    );
  },
  componentWillUnmount: function() {
    console.log('about to unmount - clean up stuff here');
  }
});

function buyButtonReact(product, element) {
  ReactDOM.render(
    <BuyButtonComponent product={product} />,
    element
  );
}
```

I've added the ``componentDidMount`` and ``componentWillUnmount`` methods with some ``console.logs``. They clearly don't really
do anything in this component, but in a real component you'll often do something in ``componentDidMount`` - 
subscribe to event from a Flux store or something - that needs to be cleaned up when the component unmounts.
We need to make sure these methods still get called at the right times or you risk memory leaks or trying
to update a component's state property when it no longer exists (which will throw an error).

So now we'll alter our jQuery function for rendering the product list to use our new React ``BuyButton`` component.
We'll use the same strategy of adding an extra ``button-container`` component here. We also attach the product data
to the container component so we can use it later.

```javascript
products.forEach(function(product) {
  var item = $('<li>' + product.name + '</li>');

  /* add a container element where 
   * we'll attach our React component
   */
  var buttonContainer = $('<span class="button-container"></span>');
  // add product data to use in our React component
  buttonContainer.data('product', product);

  item.append(buttonContainer);
  list.append(item);
});
```

After we've inserted the main product list with jQuery, we iterate over the container nodes and use the product
data to render the buy buttons with React. You can see in the console that ``componentDidMount`` is called for each component.

We might render this product list multiple times, so we need to make sure our ``productListJqueryReact`` function works
when called repeatedly. jQuery is going to blow away the whole DOM each time which won't give React a chance
to do its clean up (calling ``componentWillUnmount``), so we need to manually unmount the React components *before*
we insert a new list with jQuery.

```javascript
// clean up any mounted React components
$(element).find('.button-container').each(function() {
  ReactDOM.unmountComponentAtNode(this);
});
```

You can verify in the console that ``componentWillUnmount`` is called for each of the buy buttons every time the list is re-rendered. Here's the complete function for rendering the product list:

```javascript
function productListJqueryReact(products, element) {
  var list = $('<ul class="product-list"></ul>');

  // clean up any mounted React components
  $(element).find('.button-container').each(function() {
    ReactDOM.unmountComponentAtNode(this);
  });

  products.forEach(function(product) {
    var item = $('<li>' + product.name + '</li>');

    /* add a container element where 
     * we'll attach our React component
     */
    var buttonContainer = $('<span class="button-container"></span>');
    // add product data to use in our React component
    buttonContainer.data('product', product);

    item.append(buttonContainer);
    list.append(item);
  });

  // replace the existing list if there is one
  var currentList = $(element).find('.product-list');
  if (currentList.length) {
    currentList.replaceWith(list);
  } else {
    $(element).append(list);
  }

  // attach our React components to the containers
  list.find('.button-container').each(function() {
    var container = $(this);
    var product = container.data('product');

    /* React needs a plain, non-jQueryfied DOM 
     * element, so we can use plain "this"
     */
    buyButtonReact(product, this);
  });
}
```

I'm sure there are other ways to handle the same issues, and other edge cases where jQuery and React can conflict and cause problems, but the examples above cover the most common use cases I've encountered. There's not a lot of writing I could find about using both simultaneously, and the conventional wisdom seems to basically be "don't do it," so let us know if you have experience working with both or think we missed something!
