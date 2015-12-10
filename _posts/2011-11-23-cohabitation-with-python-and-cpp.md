---
title: Cohabitation with Python and C++
author: Chris
layout: post
permalink: /cohabitation-with-python-and-cpp/
wordpress_post_id: 304
dsq_thread_id:
  - 527842050
categories:
  - Uncategorized
---
Back in the day, [Oyster.com][1] was a C++ shop. One day we decided to convert to Python. We didn’t convert everything to Python, which left us with the task of bridging the gap between them. Of course there were issues when setting up this communication between the C++ and Python libraries.

We made the decision to convert to Python for several reasons. One main reason was to take advantage of some good but free Python libraries. We converted almost all of our code &#8212; it&#8217;s better and easier to maintain code in one language than in two. But one of our backend engines was complicated and it worked, so we decided to leave it in C++.

In simple cases, bridging the gap from Python to C++ is relatively easy. Python provides a routine that will convert data from Python’s managed memory to C++. [PyArg_ParseTuple][2] is used to convert incoming Python objects into C data types. We had to then iterate over the lists using PyList_GetItem to convert lists to arrays. We wrote the conversion function, everything worked, and we pushed the results into production.

But we experienced periodic crashes which we could not track down. While investigating the crashes we discovered that our multithreaded system was essentially only handling one incoming request at a time! Our C++ code had lots of dependencies and sometimes could take a while to return a result. It turned out that the entire Python server would block waiting for the C++ code to return.

The problem was [Python’s Global Interpreter Lock][3] (GIL). The GIL prevents multiple native threads from executing Python bytecodes at once. Apparently this is done because Python’s memory management isn’t thread safe.

While there are a few things that can be done to allow Python to play nicely with C++ in a multithreaded environment, we were in a time crunch to get the problem solved. The problem wasn’t so much the multithreading, but the amount of time that was spent inside the C++ code. If the C++ code is quick then Python won’t block for very long.

We solved our problem by redesigning the flow &#8212; instead of calling our C++ code directly inside Python, we switched to running our C++ code in a separate process and talking to it via a simple HTTP API. Our multithreading and GIL issues disappeared and became multi-processing issues (where there are much clearer, safer boundaries).

The lesson we learned from this is that multithreading is difficult, even when it looks simple. Python and C++ can play nicely with each other, as long as the C++ call is quick. You can’t let the call into C++ block for too long, as you need to let Python release the GIL occasionally to let other threads run. While it may be possible to solve this problem, we deemed our new multi-process HTTP solution worked great (in fact it probably works better this way), and we didn’t have the time to delve into a solution involving Python’s GIL.

As always, tread carefully when doing any sort of multithreading.

 [1]: http://www.oyster.com/
 [2]: http://docs.python.org/c-api/arg.html#PyArg_ParseTuple
 [3]: http://wiki.python.org/moin/GlobalInterpreterLock