---
title: Who Can I Trust To Send My Email?
author: Mike
layout: post
permalink: /who-can-i-trust-to-send-my-email/
wordpress_post_id: 187
dsq_thread_id:
  - 527842048
categories:
  - Uncategorized
---
Oyster.com has a large average transaction size.  Customers visit our site and commonly hand over thousands of dollars.  Eventually they will expect their hotel room to be available, but what they want immediately after paying us is an email telling them that we got their order, their room has been confirmed, and everything is OK.  This is a very compelling reason to have a stable email provider.

There is also the ever-growing number of people on our email lists.  They have told us they love our website and would like to purchase something from us if we would just send them a nicely personalized email that maybe has a coupon in it.  That is another great reason to be able to get our emails out reliably and track their performance.

So who can I trust to send my email?  There are more companies around than you would believe who are fighting for the business.  I looked at about a dozen of them.  The criteria used were:

  1. **Cost**.  Oyster is a startup and doesn’t like to pay BigCompany prices.
  2. **Deliverability**.  Spammers have made email deliverability a huge pain in the rear.  Getting your message into your customer’s inbox is not trivial.
  3. **API**.  I am not interested in a solution that I can’t extend.  We push all our services and tools very hard and that’s impossible without a good API.
  4. **Features**.  Of course, the fewer things I have to write means more time actually working on our website instead of our email campaigns.

## The Competitors

The cheapest is [Amazon][1].  $0.10 CPM (cost-per-thousand) will stretch a startup’s budget a long way.  Their deliverability is a question mark though, and they are very low on features.  I am trying to hire an email vendor, not build one.

A step up are companies like [MailChimp][2].  $0.50 CPM, but you get a lot of feature for your money.  Deliverability was a concern, but they will give you a dedicated IP address for $1,000 and that will at least keep the spammers off the exact same address.  Things started breaking down though when I dug into their API.  It is very campaign-centric.  How do I pull up a list of the last 10 mails I sent a specific customer?  They do get some points for having the best designed website in the industry.

Another bump up the pricing ladder takes you to a fellow NYC startup, [Sailthru][3].  They probably have the worst designed website in the industry, but I heard good things about them so I got a demo account and took it for a spin.  They had a simple and powerful REST API that would play nicely with our codebase.  They definitely understood that deliverability is paramount.    There were some rough edges around the product (like the documentation), but I got quick support whenever I had a question.

<div id="attachment_195" class="wp-caption alignnone">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/08/comparison.jpg"><img class="size-large wp-image-195" title="comparison" src="http://tech.oyster.com/wp-content/uploads/2011/08/comparison-1024x404.jpg" alt=""   /></a> 
  <p class="wp-caption-text">
    Which of these companies hired a web designer?
  </p>
</div>

The top tier includes some of the big names in the industry.  StrongMail, ExactTarget, CheetahMail, BlueHornet, Silverpop, and on and on.  This is the world of long web demos where they spend half an hour showing you their amazing WSIWYG editor (which a) sucks and b) we’re not going to use).  Then comes a breakdown of every possible report you could generate.  The conversation gets awkward when you ask for access to a sandbox where you can try it for yourself.

There were some diamonds in the rough.  [StrongMail ][4]and [ExactTarget ][5]both had strong offerings.  StrongMail  can pull data straight from your database.  They had a convoluted API, but everything you need is in there.  There were also some nice goodies like multi-variate A/B testing support built in.  Prices start in the $4 -$5 CPM range, but it comes down as your volume grows.

## The Winner Is..

The competition was fierce.  Salespeople were sending me Yankees tickets and taking me out to fancy restaurants.  At the end of the day we chose Sailthru.  They had some features that were very useful for us.  Every mail we sent was archived on the web and we easily tied links to them into our customer service portal.  They have a developer-friendly API and were happy to add the hooks and functions that we asked for.  The templating system is simple and flexible.  There are a couple of warts, but it is a great system for the money.

## So how is it going?

We’ve been using Sailthru for a while now.  Overall I’ve been very happy with them.  [Senderscore][6] reports our emails as highly deliverable.

[<img class="size-full wp-image-188 aligncenter" title="senderscore" src="http://tech.oyster.com/wp-content/uploads/2011/08/senderscore.jpg" alt="98% Acceptance Rate"   />][7]

&nbsp;

Besides deliverability, there are a couple key features that make Sailthru work for us.  The first is that their templating system does not get in our way.  Our transactional mails go out in different languages and currencies, so it is not going to work if we have to build emails in your system.  Most of our Sailthru templates look like this:

<pre>{body}</pre>

We process everything on our side with Cheetah templates and our internationalization code and then send the entire email up to them.  We get all the benefits of the reports, link tracking, and other goodies in Sailthru, but we can leverage our localization tools to build the mail.

<div id="attachment_201" class="wp-caption alignnone">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/09/i18n_email.jpg"><img class="size-full wp-image-201 " title="English and Portuguese emails" src="http://tech.oyster.com/wp-content/uploads/2011/09/i18n_email.jpg" alt=""  /></a> 
  <p class="wp-caption-text">
    International emails mean basically all the content changes.
  </p>
</div>

The second is that their API is simple and complete.  It is a REST api and we wrote our own interface in Python, but they have a bunch of clients available.  We use it for everything.  Sending transactional mails is as easy as POSTing the body of the mail.  Campaign mails are a two step process because they are heavily personalized.

The issue is how do you get different content for 100,000 people into your email provider?  A solution like StrongMail can connect right to your database, but I haven’t seen anyone else who does that.  Sailthru has data feeds that you can setup, but it is only good for specifying the same content for everyone.  Sailthru’s solution is through their API &#8216;job&#8217; call.  This is used for a lot of bulk tasks like massive updates to your lists.  In this case it allows you to set data fields on all your users.

Every night we build up a file with customized article selections for each user and post it to Sailthru.  A callback lets us know it has been processed and we are ready to start a campaign mail whenever it is convenient for us.  My data fields look like:

<div id="attachment_203" class="wp-caption alignnone">
  <a href="http://tech.oyster.com/wp-content/uploads/2011/09/email_vars.jpg"><img class="size-full wp-image-203" title="Email data fields" src="http://tech.oyster.com/wp-content/uploads/2011/09/email_vars.jpg" alt=""   /></a> 
  <p class="wp-caption-text">
    Data fields get populated in Sailthru
  </p>
</div>

And they result in an email that looks like this:

[<img class="size-full wp-image-204 alignnone" title="Customized email" src="http://tech.oyster.com/wp-content/uploads/2011/09/customized_email.jpg" alt=""   />][8]

Email is a critical component of our business, and we evaluate our provider regularly.  So far Sailthru’s combination of deliverability, features, API and price point have been tough to beat.

 [1]: http://aws.amazon.com/ses/ "Amazon SES"
 [2]: http://mailchimp.com/ "MailChimp"
 [3]: https://www.sailthru.com/ "Sailthru"
 [4]: http://www.strongmail.com/ "StrongMail"
 [5]: http://www.exacttarget.com/ "ExactTarget"
 [6]: https://senderscore.org/ "SenderScore"
 [7]: http://tech.oyster.com/wp-content/uploads/2011/08/senderscore.jpg
 [8]: http://tech.oyster.com/wp-content/uploads/2011/09/customized_email.jpg
