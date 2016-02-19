---
layout: post
title: Making Your Node.js Processes Fault Tolerant with PM2
about: A brief walkthrough of the process manager PM2 and a use case with a deployed Node.js server.
---

TLDR... The 2 most useful commands:

Start a Node process and automatically restart if it crashes:


```
pm2 start someApp.js
```

List information on all active processes:

```
pm2 list
```

Simply put, PM2 is a powerful process manager for Node.js applications.  It has a few basic commands that are extremely useful, as well as a ream of other commands for more advanced tasks.  It even has built in load-balancing for your processes.  I recommend checking out the PM2's [Github repo](https://github.com/Unitech/pm2).

For today, we'll keep it to one simple use case which I found extremely helpful recently: automatically restarting a Node server on my virtual machine every time it crashed.  I had recently created a droplet on Digital Ocean and was running a Node server on the droplet to serve up a new application I had built.  What I noticed was that about once per day, my server was crashing, leaving my application inaccessible.

What was breaking was application was a form of attacks where different ip addresses were forcing my server to make http GET requests to seemingly arbitrary .php files.  It was an interesting issue, but the main problem was these malicious requests were exceeding the RAM allotment I had purchased for my droplet, and Digital Ocean was then killing the process.  

PM2 solved my problems with a few simple commands.  The first thing you will want to do is install it gloabally with npm:

```
npm install pm2 -g
```

Now to start a process using PM2 and have it automatically restart everytime it crashes, just run

```
pm2 start server.js
```

Or to start the process and have it restart every time a file is changed

```
pm2 start server.js --watch
```

Lastly, to view all active pm2 processes, just run

```
pm2 list
```

And you will see the following output:

![pm2-list](/images/pm2_list.png)

You can see the server has had to restart 53 times, but PM2 has handled these restarts gracefully.  Another service called "forever" offers similar basic functionality, but from my experience PM2 is the better choice, both because it is extremely easy to get started with, and it offers load-balancing and advanced tasks right out of the box.  Hopefully you can use PM2 to make your own processes fault taulerant as well.


