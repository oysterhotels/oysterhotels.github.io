---
title: "Delivering a great i18n experience on a budget"
author: John
layout: post
permalink: /delivering-a-great-i18n-experience-on-a-budget/
disqus_page_identifier: delivering-a-great-i18n-experience-on-a-budget
published: true
---

Here at Oyster we have a ton of written content. We cover over 15,000 hotels with written reviews, but beyond that we have around 3,500 blog articles, 1,500 'best of' lists, and over 1,000 other documents including our fakeouts, photo slideshows, and travel guides. That is a LOT of words - 17.4 million to be more precise.

Our engineering problem has been making all of that content available to a much wider audience than just those who can understand English. The first part of tackling that problem is getting that content translated. However, it's not as simple as just sending those 17.4 million words to a translation service. There are many factors to consider:

1. Our content changes over time. In total, our documents number around 22,000 but our revisions over time amount to around 325,000! The good thing about that is, a lot of that content is duplicated as drafts evolve and only a portion is published. But, as published documents change, we still have to make sure either our translations get updated or they don't break.

2. Translations are expensive. The rate we work with is 8 cents per word which means if we straight up translated everything, our costs would amount to roughly $1.4 million just for 1 language! And currently we support 5 non English languages bringing that bill up to $7 million. Our buget is a mere fraction of that cost so it's clear that we'd rather spend money delivering tons of new amazing photos and reviews rather than translating everything.

3. Luckily, it seems we don't need to translate everything since there is a high variability of traffic/value across all of our content. A small fraction of our content accounts for a large amount of our revenue - big name hotels in destinations like Dominican Republic, New York, or Las Vegas are more in demand. 'Best of' lists also turn out to be very high in demand because they cut out a lot of the time it take to make a very valuable decision.

4. Just having translations for big ticket items is not good enough. Wait a second, I just said it seems we don't need to translate everything. Well, it turns out that we do in order to create a good overall experience. Imagine you were browsing a list of 10 hotels and half the hotel summaries were in English (or your native language) and half were in a foreign language. The experience would be quite jarring and even if you found the perfect hotel, you still might have a dissatisfied feeling. Luckily there are ways of doing what could be called 'compressed' translations, by either translating only the most important parts of a document or filling in less important parts with low quality translations such as machine translations - which is what Google and Facebook let you do for content they detect is in a foreign language.

These factors led us to build a solution that provides all these features - translations are based on revisions so we can update different languages independently, they can be of different quality/price, there can be multiple translations of the same revision/language but of different quality and merged together, and translations can be full or partial.

Translations table (sample rows)
![German visits](/public/images/translations.PNG)

That is a LOT of moving parts on the backend but it has allowed us to be very flexible and picky with what and how we translate our content. We've managed to get by with only translating around 10% of our documents, of which around only half of those are full translations and the rest are partial. The other 90% is mostly machine translated, but we're constantly striving to lower that number or find other solutions. The cost of this is a rather complex data structure and system to handle all the details needed to put these parts together. We've managed this by streamlining our translation process and consolidating the logic as much as possible.

That is why in addition to this system for handling hotel reviews and articles, we have a separate system for translating and displaying text that is part of our UI. For translating, we automate sending requests to [Gengo](https://www.gengo.com) through their API and for display we use standard [gettext](https://en.wikipedia.org/wiki/Gettext). This solution works very well for text that is static and highly visible - just always translate it at high quality. Fortunately, our UI text at 40,000 words is only 0.2% the size of our document text!

Earlier this year, we optimized our i18n experience even further on [de.oyster.com](https://de.oyster.com) as a test bed by choosing not to go the machine translation route but instead, selectively hide content. We decided to hide all machine or untranslated blog articles and travel guides because we felt that those low quality translations might be interfering with the rest of our content from both a user experience and SEO perspective. At the same time, we almost tripled the number of documents we have translated to German, even though that still only brought us up to around 13%. The hard part about that was determining exactly which hotels and documents to translate which was a whole other problem that our data scientist was charged with tackling. Fortunately it seems these changes paid off well as our de.oyster.com SEO traffic growth is far outpacing the US site (see below).

![Organic search YTD growth](/public/images/ytd_de_us.png)


Normal traffic patterns for a travel site are: 

* High in January
* Dip in the spring
* Highest in the summer
* Declining for the rest of the year

The US site is showing that trend this year, but the German site has broken away since we made these changes in the spring.  This is not the end of our i18n work as we have plenty of other existing features and plans to help us bring more of our amazing content to the rest of the world.
