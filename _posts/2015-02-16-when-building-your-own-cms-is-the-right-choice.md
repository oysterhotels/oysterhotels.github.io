---
title: When Building Your Own CMS is the Right Choice
author: Alex
layout: post
permalink: /when-building-your-own-cms-is-the-right-choice/
wordpress_post_id: 837
dsq_thread_id:
  - 3521864604
categories:
  - Uncategorized
---
In the latter half of last year, we decided to replace the CMS that powers the content on Oyster.com. Actually we replaced three CMSs with a single one. Oyster is primarily in the business of creating content such as our in-depth hotel [reviews][1], [roundups][2], [slideshows][3], and various other articles that help travelers spend their hard-earned vacation days and dollars wisely. So we knew it was an important task to build the best tool we could to enable our writing and editing staff to put out high quality content easily.

[<img class="aligncenter size-full wp-image-851" src="http://tech.oyster.com/wp-content/uploads/2015/02/content.jpg" alt="content"   />][4]

Obviously we’ve been doing this for a while, so we had tools in place, but we had reached a point where we needed to make a change. As I mentioned above we had three different CMSs that we used: one for hotel reviews, one for articles (both of these were custom), and a WordPress blog for blog posts.

## Documents and Structure

[<img class=" size-full wp-image-842 alignright" src="http://tech.oyster.com/wp-content/uploads/2015/02/pros-cons.png" alt="Pros, Cons & Bottom Line"  />][5]The custom editors were used for creating structured documents which consisted of a number of sections (such as the Pros, Cons, and Bottom Line sections of our hotel reviews) which in turn consisted of a number of fields. These were stored in a custom text format and any text formatting was stored as wiki markdown. This made it harder than it needed to be to update documents’ structure or create new document types since all the code that parsed and rendered the documents was custom. Also the UIs for the custom editors were due for a good refresh.

The WordPress editor presented different problems. WordPress is quite good for producing a nicely formatted bit of text, but what you get when you write a post is a big blob of HTML. Formatting and styles are all mixed in with your content. Also everything is totally static, so if a hotel changes names or closes, or some other piece of information in our hotel database changes, it doesn’t get updated in the blog post. We knew we wanted our blog content to be integrated with the same database used by our hotel reviews so we could more easily surface rich information about hotels and pricing.

We realized we wanted to keep the concept of structured documents; our hotel reviews have a well-defined format, and we need to be able to write those in a structured way. Similarly our [roundups][6] always consist of an intro section and a list of hotels each with a short blurb relevant to the roundup topic. At the same time we want our writers to have the flexibility to produce more freeform blog-oriented content with a degree of flexibility for formatting. We decided we could do this by defining a set of formatting blocks that reflected style conventions they were already using, with an eye to extending these fairly easily as needed. This frees up the writers from having to focus on layout and focus on what they want to say. Having well-defined formatting blocks, or “widgets,” also means we can create responsive templates for how the articles display &#8211; we can make it look good on a desktop, phone, or tablet since the documents contain **content** information and not **layout** information.

## JSON Documents

When deciding how the documents should be stored, it was really a no-brainer that they should be stored as JSON. JSON plays well with pretty much anything these days. Any server side language you use (we use Python) ought to be able to parse JSON into a useful data structure with a few lines of code. We use Postgres for our content database, and Postgres has a built-in JSON type that you can query against, put indexes on, and use with various [functions and operators][7]. Storing documents as JSON in the database means we don’t have to change the database schema every time we want to add a new widget type or document field, but we don’t really have to compromise on queryability either.

**Part of a Slideshow Document**

<pre>{
    "IsOrdered": true,
    "IsAward": true,
    "ShowAboutOyster": false,
    "FeaturedOnArticles": false,
    "Title": "Best Beach Hotels in Miami",
    "Intro": "&lt;p&gt;A team of Oyster reporters has made multiple trips to Miami to visit nearly 200 hotels. We slept in the beds, lounged by the pools, ate in the restaurants, and even sampled the nightlife, all with an eye toward selecting the most distinguished properties. Here's a list of our favorite beachfront hotels.&lt;/p&gt;",
    "Hotels": [
        {
            "Url": "/miami/hotels/the-setai/",
            "Type": "Hotel",
            "Blurb": "&lt;p&gt;Paradise doesn't come cheap. Striking but sober mood-lit design; impeccable service; huge, immaculate rooms; three pools, each a different temperature; and a prime beachside location make the Setai one of the best hotels in Miami. Its restaurants are more about design than food, but several of Miami's best restaurants are just half a block away.&lt;/p&gt;",
            "PhotoUrl": "/miami/hotels/the-setai/photos/beach-the-setai-v134081"
        },
        {
            "Url": "/miami/hotels/w-south-beach/",
            "Type": "Hotel",
            "Blurb": "&lt;p&gt;The stunning new 312-room W South Beach -- located on the beach, on the northern outskirts of South Beach -- blends cute comforts, intricate design (that spares no expense), and flawless service. Large, modern rooms; terraces angled to overlook the ocean; elegant landscaping around the pool; a freshly-opened spa -- the W tops the Miami greats.&lt;/p&gt;",
            "PhotoUrl": "/miami/hotels/w-south-beach/photos/beach-w-south-beach-opening-may-2009-v289241"
        },
</pre>

And of course the UI for our CMS is web-based, which means the bulk of the functionality is written in JavaScript, so working with JSON on the front end is extremely easy.

I mentioned before that our documents have a well-defined structure, but a JSON object just consists of arrays, strings, numbers, booleans, and more nested objects. What we needed was a way to define how a given JSON document of a certain type is supposed to look &#8211; a JSON schema if you will. So we used, unsurprisingly, [JSON Schema][8]. Much like XML DTD does for XML documents, JSON Schema lets you define how a JSON object should be structured, and the definition itself is a JSON object. It provides the basics such as what types of values are allowed for a given property, which properties are required, max and min ranges, enums, regexes, most of what you’ll need. You can also have nested schemas, so we can define a “Slide” schema, and then say the “Slideshow” schema consists of a title, an intro paragraph, and one or more Slides.

**JSON Schema for Slideshow documents**

<pre>"Slideshow": {
    "allOf": [
        {"$ref": "#/definitions/baseArticle"},
        {
            "properties": {
                "IsAward": {"type": "boolean", "default": false},
                "Slides": {
                    "type": "array",
                    "items": {"$ref": "#/definitions/Slide"}
                }
            },
            "required": ["Slides"]
        }
    ]
}
</pre>

Well it’s nice to have your document structure defined, but you have to do something with that information. Namely you want to be able to validate your documents and get useful error information when the validation fails. For that we used the Python [jsonschema package][9]. When a writer saves a document in our CMS, it sends a JSON object to the server. The CMS back end validates the document against the relevant schema, and we get back a handy error tree that tells us what went wrong. Since the document structure on the server matches the structure on the CMS UI, it’s not too hard to parse that error tree and match error messages to input fields on the writer’s screen to show them some helpful feedback: “This field is required,” “This is an invalid URL,” and so on.

[<img class="wp-image-858 size-full" src="http://tech.oyster.com/wp-content/uploads/2015/02/errors.png" alt="errors"  />][10] Error Handling 

## UI Concerns

That brings us to the front end of the CMS. There are of course various pages that allow you to search through documents by types and tags, and see the editing history, but most of that was ground we had covered before. A large part of the work done was on the document editor itself &#8211; the interface our writers use for creating a single document.

To create the document editor, we wrote a healthy amount of JavaScript. Basically the editor needs to take the concept of structured Documents composed of Widgets and show the user an intuitive UI for writing and editing.

So let’s say you’ve started writing a new Travel Guide article. What happens is an animated paperclip with eyes pops up and says “I see you’re writing a new Travel Guide, need some help?” Wait no, that’s not what happens.

[<img class="aligncenter size-full wp-image-856" src="http://tech.oyster.com/wp-content/uploads/2015/02/editor-2.png" alt="editor-2"  />][11]

What you’ll see is a mostly blank document with inputs for some top-level fields and then spaces to populate with different Widgets. The toolbar on the right hand side has a list of the relevant Widget types which you can drag into place in the document. All the Widgets can be dragged and dropped in the different places they can go in the Document.

Each Widget contains various fields for text, URLs, checkboxes, drop-down menus, etc. Some fields allow WYSIWYG editing with a whitelisted subset of HTML tags via the [WYSIHTML5][12] editor. We store the HTML as HTML in the JSON rather than markdown &#8211; since we allow such a small list of tags, and the editor outputs well-formed markup, it’s perfectly safe, so we figured why go through an extra encoding and decoding step?

The design for how the editor turns a JSON document into a UI and back again is quite involved, and highly structured, but follows the principles of how the documents are organized. Each Document type, Widget type, and Field type corresponds to a JavaScript class in our editor code, and these classes all inherit from a common base class.

When the editor starts up to edit a Document, we just pass in a JSON object from the server. Each object and nested object in the JSON has a “Type” field which is used to call the proper constructor for the main Document object and its constituent Fields and Widgets to turn the plain JSON objects into class instances. These classes provide, at minimum, a fromJSON method to assign JSON properties to instance properties, and a toJSON method to put the instance properties into a plain JavaScript object that’s ready for serialization. Some classes also provide methods for things like sanitizing input, formatting error messages, providing a word count, etc.

**A method for turning a class instance into a simple JSON object**

<pre>models.Model.prototype.toJSON = function () {
    // return a JSON-friendly object

    var jsonObj = {}, property, jsonProperty, value;

    for(property in this) {
        value = this[property];
        if (value.toJSON) {
            // JSON properties are uppercased
            jsonProperty = util.ucFirst(property);

            jsonObj[jsonProperty] = value.toJSON();
        }
    }

    if (this.typeName) {
        jsonObj.Type = this.typeName;
    }

    // delete null properties and empty strings
    for (property in jsonObj) {
        value = jsonObj[property];
        if (value === null || value === '' || Array.isArray(value) && value.length === 0) {
            delete jsonObj[property];
        }
    }

    return jsonObj;
};
</pre>

So you might rightly ask “okay so you’ve got your document, and you turned that into some other objects, but how do you, you know, do stuff with it?” Well the “doing stuff” part of the editor &#8211; putting in text, dragging things around, deleting, and adding things &#8211; is all handled with data binding.

If you’re interested at all in building JavaScript-based UIs, you’re undoubtedly familiar with the concept of data binding. In short you have some object or objects which represent your data (in our case, the Document) and the UI which consists of a bunch of DOM nodes. When someone changes the DOM representation of your data, you want that to be updated in the model object, and similarly changes to the data model should be reflected in the UI. Data binding is the practice of doing this in an explicit and automated fashion so that your UI and your data are always in sync.

A number of popular JavaScript application frameworks like Angular, Backbone, and Ember provide data binding as part of what they do, but they tend to be more useful when building large, single page applications as they provide lots of other features such as URL routing, module loading, and dependency injection. They also tend to be pretty opinionated about how your code is structured, and often learning to use them correctly is quite involved.

We really only wanted something to handle the data binding piece &#8211; a data binding **library** rather than a whole application framework. To that end we chose [Rivets.js][13] for a number of reasons. Rivets.js is small first and foremost &#8211; both in the scope of what it does (it does data binding, and that’s it) and in terms of source code. It also doesn’t care about what your data model looks like &#8211; it binds to regular object properties, so your model object can be a plain object or some custom class you wrote, whatever. It’s also easy to learn, easy to use, and easy to extend.

**Rivets bindings in use:**

<pre>&lt;div id="tag-search"&gt;
    &lt;div&gt;
        &lt;input type="text" rv-edit="tagSearch:searchTerm" rv-on-keydown="tagSearch:keyDown"&gt;
        &lt;i class="fa fa-search search-main-icon" rv-hide="tagSearch:isSearching"&gt;&lt;/i&gt;
        &lt;i class="fa fa-circle-o-notch fa-spin search-main-icon" rv-show="tagSearch:isSearching"&gt;&lt;/i&gt;
        &lt;i class="fa fa-plus search-main-icon" rv-on-click="tagSearch:clickAdd"&gt;&lt;/i&gt;

        &lt;ul id="tag-results" rv-show="tagSearch:results:length"&gt;
            &lt;li rv-each-result="tagSearch:results" rv-class-selected="result:selected" rv-on-click="tagSearch:clickResult"&gt;
                &lt;span class="tag-icon" rv-addclass="result:type"&gt;
                    &lt;i class="fa fa-tag" rv-match-tag="result:type"&gt;&lt;/i&gt;
                    &lt;i class="fa fa-map-marker" rv-match-location="result:type"&gt;&lt;/i&gt;
                    &lt;i class="fa fa-cubes" rv-match-category="result:type"&gt;&lt;/i&gt;
                &lt;/span&gt;
                &lt;span rv-html="result:html"&gt;&lt;/span&gt; ({ result:count })
            &lt;/li&gt;
        &lt;/ul&gt;
    &lt;/div&gt;
&lt;/div&gt;
</pre>

One instance of how we extended Rivets was for templating. Each of our Widget and Field classes have various properties and functionality, and they also each have a particular way they need to display on the screen. We wanted to have a snippet of HTML for each Widget to use as a template, and use Rivets bindings within the template. Frameworks like Angular let you define partial templates, nest them inside eachother, and each template can have its own isolated scope. Rivets doesn’t support this out of the box, but it turned out to be easy enough to add a new template binder that does just that. It’s pretty basic &#8211; it doesn’t do lazy-loading or have sophisticated scoping options, but it works fine for what we needed, and you never have to try to remember what the word “transclude” means.

**Partial template binder for Rivets.js**

<pre>rivets.binders['template-*'] = {
    bind: function(el) {
    },
    unbind: function(el) {
        var children = $(el).children(), boundView;

        //unbind the view from the child element
        if (children.length) {
            boundView = $(children[0]).data('templateBoundView');

            if (boundView) {
                boundView.unbind();
            }
        }

        $(el).html('');
    },
    routine: function(el, value) {
        var modelName, templateName;

        if (!value) {
            console.log('missing value', el);
        }

        templateName = value.template.toLowerCase();
        modelName = this.type.split('-')[1];

        renderTemplate(el, templateName, modelName, value);
    }
};

function renderTemplate(el, templateName, modelName, model) {
    var myEl, html, templateData, child, view;
    myEl = $(el);

    if (myEl.html()) {
        return;
    }

    //insert the html - must have 1 root element
    html = EDITOR_TEMPLATES[templateName]
    if (!html) {
        throw("Can't find template for: " + templateName);
    }
    myEl.html(html);

    //bind the view to the child element
    templateData = {};
    templateData[modelName] = model;
    child = $(myEl.children()[0]);
    view = rivets.bind(child, templateData);
    $(child).data('templateBoundView', view);
}
</pre>

One other way we had to modify our use of Rivets deals with how the library binds to object properties. Out of the box, Rivets detects changes in the data model by wrapping attribute access with getters and setters. That works fine if the properties you want to observe hold primitive values like Strings and Integers and the like &#8211; but for some of our bindings we needed to detect changes on entire objects, and especially arrays.

Thankfully Rivets allows you to write new [adapters][14] to modify how change detection happens. Also thankfully, we only needed to support recent versions of Chrome on the browser side of things, and Chrome now supports a native Object.observe and Array.observe. So when new Widget gets pushed onto an array inside our Document, the iteration binder that renders the Widgets gets updated automatically.

Too often internal tools don&#8217;t get the attention they deserve &#8211; they are often left to languish while development resources are put towards customer-facing features and operational concerns. In our situation we felt the need to get our content management tools right and do the smart thing rather than what would necessarily be the easy thing. It took a lot of hard work from our dev team, but so far it has paid off in terms of productivity and agility. There were some interesting challenges along the way, and we&#8217;ll have to continually adapt our tools as the business grows, but I think we&#8217;ve set ourselves up in a good position from which to move forward.

 [1]: http://www.oyster.com/miami/hotels/fontainebleau-resort-miami-beach/
 [2]: http://www.oyster.com/dominican-republic/hotels/roundups/best-all-inclusive-resorts-in-the-dominican-republic/
 [3]: http://www.oyster.com/hotels/theme/hotel-odds-and-ends/slideshows/best-sunsets/
 [4]: http://tech.oyster.com/wp-content/uploads/2015/02/content.jpg
 [5]: http://tech.oyster.com/wp-content/uploads/2015/02/pros-cons.png
 [6]: http://www.oyster.com/miami/hotels/roundups/best-beachfront-hotels-in-miami/
 [7]: http://www.postgresql.org/docs/9.3/static/functions-json.html
 [8]: http://json-schema.org/
 [9]: https://pypi.python.org/pypi/jsonschema
 [10]: http://tech.oyster.com/wp-content/uploads/2015/02/errors.png
 [11]: http://tech.oyster.com/wp-content/uploads/2015/02/editor-2.png
 [12]: http://xing.github.io/wysihtml5/
 [13]: http://rivetsjs.com/
 [14]: http://rivetsjs.com/docs/guide/#adapters