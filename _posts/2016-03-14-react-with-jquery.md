---
title: "Using React and jQuery Together"
author: Alex
layout: post
permalink: /using-react-and-jquery-together/
disqus_page_identifier: react-jquery
draft: true
---

In this post I'll be covering some tips on how to use React and jQuery together in the same UI.
Okay so first off you might be thinking "why would you want to do such a thing?" - in fact the idea
of trying to make React's declarative style live together with imperative jQuery DOM updates
may have you thinking something like [this](http://tech.oyster.com/public/images/ghostbusters.gif),
and for the most part, you wouldn't be wrong.

So first the "why." If you're starting a brand new, "greenfield" project and you want to use React,
then just do it. There's no good reason I can think of to mix-and-match React with jQuery or 
Mustache or whatever other DOM-helper/template library if you don't have to - just use React and everything will be cool
and you won't have to worry about it. If on the other hand, you have one of those "legacy" applications
that have "customers" and makes "money," and for some reason your boss is not into the idea of you
spending a few weeks rewriting the whole front end in React, but you still want in on that declarative
React goodness, you may have to figure out how to get React to play well with jQuery or something similar.

## Are you *sure* you can't rewrite it?
Let's say you have some piece of UI that gets rendered with jQuery, and you want to stick in some
new component written with React. Take a look at your rendering function. Are you just building up some
big string of HTML and sticking it in the DOM? Or using some kind of JavaScript template? If your rendering
is already reasonable functional, i.e. some data goes into your function and some HTML comes out (or gets
appened to the DOM or whatever) it will be pretty easy to just rewrite in React. So you should probably 
just do that and save yourself the inevitable hassle when something breaks and you have to debug it.

## Okay so you can't rewrite it.
For whatever reason you've determined it's not practical to rewrite your jQuery code. Here is an important
caveat: I think using jQuery and React to manage updates to the **same** DOM elements is a bad idea.
React is really smart about figuring out how to update the DOM, but that only works if React is the only thing
doing the updates. So unless you can cleanly separate the DOM elements in your UI so that some **only** get
updated by React and others **only** get updated by jQuery, I wouldn't try it. So say for instance you're
going to Render a product list with a React template and then jQuery is going to add and remove CSS classes
to the list items - this is a bad idea.

One further caveat is that you should only really consider this mixed approach if you're planning to
*eventually* replace jQuery rendering with React. Using both doesn't make sense long-term, but if you're
looking to gradually transition to React, you may have some parts of your UI using both for a while.

## Let's write some code
As an example we'll start by rendering a simple list of products with jQuery. We'll just show the name
for each product and a button to buy it. Then we'll get into replacing parts of the UI with React.

Our jQuery product list is pretty basic - it takes an array of products and inserts the list into
\#product-list-container. If the product list is updated, you just call productListJustJquery() again
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
  // add price button to DOM
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

# jQuery inside a React component
We'll start by replacing most of the product list with React but leaving the buy button in jQuery. 
This is simpler than the inverse - sticking React inside a jQuery UI - so we'll do it first.
The ProductListComponent is pretty straightforward:

```
var ProductListComponent = React.createClass({
  render: function(props) {
    return (
      <ul className="product-list">
        {this.props.products.map(function(product) {
          return <ProductComponent key={product.id} product={product} />
        })}
      </ul>
    );
  }
});
```

but in ProductComponent we need some extra
code to make the call to jQuery. We add an extra button-container element,
so that we have somewhere to put the jQuery DOM, and keep a reference to it.

```
render: function(props) {
  // we need to keep a ref to the button-container so we can update it with jQuery
  return (
    <li>
      {this.props.product.name}
      <span className="button-container" ref="buttonContainer"></span>
    </li>
  );
}
```
# Lifecycle methods
It's important to get familiar with the various React lifecycle methods.
The relevant ones here are componentDidMount - which is called after the first render, and
componentDidUpdate - which is called after subsequent renders. In each of these methods we just call
renderBuyButton, which uses our reference to the button-container DOM node to create a brand new buy
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
  $(this.refs.buttonContainer).html(buyButtonJquery(this.props.product));
}
```

Here's the complete ProductComponent:

```
var ProductComponent = React.createClass({
  componentDidMount: function() {
    this.renderBuyButton();
  },
  componentDidUpdate: function() {
    this.renderBuyButton();
  },
  render: function(props) {
    // we need to keep a ref to the button-container so we can update it with jQuery
    return (
      <li>
        {this.props.product.name}
        <span className="button-container" ref="buttonContainer"></span>
      </li>
    );
  },
  renderBuyButton: function() {
    // render the buy button with jQuery
    $(this.refs.buttonContainer).html(buyButtonJquery(this.props.product));
  }
});
```

# React components inside jQuery
Now we're going to do it the other way and stick some React DOM inside our jQuery DOM. This is a little
trickier. We'll start with a BuyButtonComponent in React, there's not much to it:

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
      <button className="buy-button" onClick={this.onClick}>{this.props.product.price}</button>
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

I've added the componentDidMount and componentWillUnmount with some console.logs. They clearly don't really
do anything in this component, but in a real component you'll often do something in componentDidMount - 
subscribe to event from a Flux store or something - that needs to be cleaned up when the component unmounts.
We need to make sure these methods still get called at the right times or you risk memory leaks or trying
to update a component's state property when it no longer exists (which will throw an error).

So now we'll alter our jQuery function for rendering the product list to use our new React BuyButton component.
We'll use the same strategy of adding an extra button-container component here. We also attach the product data
to the container component so we can use it later.

```javascript
products.forEach(function(product) {
  var item = $('<li>' + product.name + '</li>');

  // add a container element where we'll attach our React component
  var buttonContainer = $('<span class="button-container"></span>');
  // add product data to use in our React component
  buttonContainer.data('product', product);

  item.append(buttonContainer);
  list.append(item);
});
```

After we've inserted the main product list with jQuery, we iterate over the container nodes and use the product
data to render the buy buttons with React. You can see in the console that componentDidMount is called for each component.
We might render this product list multiple times, so we need to make sure our productListJqueryReact function works
when called repeatedly. jQuery is going to blow away the whole DOM each time which won't give React a chance
to do it's clean up (calling componentWillUnmount), so we need to manually unmount the React components **before**
we insert a new list with jQuery.

```javascript
// clean up any mounted React components
$(element).find('.button-container').each(function() {
  ReactDOM.unmountComponentAtNode(this);
});
```

You can verify in the console that componentWillUnmount is called for each of the buy buttons every time the list is re-rendered. Here's the complete function for rendering the product list:

```javascript
function productListJqueryReact(products, element) {
  var list = $('<ul class="product-list"></ul>');

  // clean up any mounted React components
  $(element).find('.button-container').each(function() {
    ReactDOM.unmountComponentAtNode(this);
  });

  products.forEach(function(product) {
    var item = $('<li>' + product.name + '</li>');

    // add a container element where we'll attach our React component
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

    // React needs a plain, non-jQueryfied DOM element, so we can use plain "this"
    buyButtonReact(product, this);
  });
}
```

--- TODO: Conclusion, code cleanup, spell check ---
