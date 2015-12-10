---
title: 'A few little libraries we&#8217;re making open source'
author: Ben
layout: post
permalink: /a-few-little-libraries-were-making-open-source/
wordpress_post_id: 713
dsq_thread_id:
  - 684317081
categories:
  - Uncategorized
---
Here at Oyster.com most of our codebase is written in Python, which is of course open source, and we use many open source libraries, including [web.py][1], [Babel][2], and [lxml][3]. Now it&#8217;s time to give a (tiny bit) back.

Three third-party services we use heavily are [Sailthru][4], [QuickBase][5], and [myGengo][6]. When we started with them, they didn&#8217;t have Python libraries available, or the Python libraries that were available kinda sucked, so we rolled our own.

Note that the idea here is not a fully-fledged API, but an &#8220;at least what *we* needed&#8221; wrapper. It may well be what you need, too, so check out the [source code on GitHub][7]. Below are some quick examples.

## Sailthru

Sailthru is the [email provider we use][8] to send transactional and mass emails.

<pre>&gt;&gt;&gt; import sailthru
&gt;&gt;&gt; sailthru.send_mail('Welcome', 'bob@example.com', name='Bobby')
&gt;&gt;&gt; sailthru.send_blast('Weekly Update', 'Newsletter', 'The CEO', 'ceo@example.com',
                        'Your weekly update', '&lt;p&gt;This is a weekly update!&lt;/p&gt;')</pre>

## QuickBase

We use QuickBase as a nice user interface to enter and manage portions of our hotel data, and use this Python module to sync between our PostgreSQL database and QuickBase.

<pre>&gt;&gt;&gt; import quickbase
&gt;&gt;&gt; client = quickbase.Client(username, password, database='abcd1234')
&gt;&gt;&gt; client.do_query("{'6'.EX.'Foo'}", columns='a')
[{'record_id': 1234, ...}]
&gt;&gt;&gt; client.edit_record(1234, {6: 'Bar'}, named=False)
1</pre>

## myGengo

Our site is (partially) translated into 5 languages, and myGengo provides an easy-to-use automated translation API. We also run the [gettext][9] strings in our web templates through myGengo.

<pre>&gt;&gt;&gt; import mygengo
&gt;&gt;&gt; client = mygengo.Client(api_key, private_key)
&gt;&gt;&gt; client.get_account_balance()
'42.50'
&gt;&gt;&gt; client.submit_job('This is a test', 'fr', auto_approve=True)
{'job_id': '1234', ...}
&gt;&gt;&gt; client.get_job(1234)
{'body_tgt': "Il s'agit d'un test", ...}</pre>

## Pull requests welcome

You&#8217;re welcome to send pull requests on [our GitHub page][7], or comment below to send other feedback! If you like or use any of these APIs, we&#8217;d love to hear from you.

 [1]: http://webpy.org/
 [2]: http://babel.edgewall.org/
 [3]: http://lxml.de/
 [4]: https://www.sailthru.com/
 [5]: http://quickbase.intuit.com/
 [6]: http://mygengo.com/
 [7]: https://github.com/oysterhotels
 [8]: http://tech.oyster.com/who-can-i-trust-to-send-my-email/
 [9]: http://docs.python.org/library/gettext.html