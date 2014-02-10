---
layout: post
title: Using Django's runserver command under Tornado (say hi to django-fujita)
date: 2013-11-10 22:36
disqus_identifier: 66642331520
---

These past couple weeks there's been lots of conversation in the studio and over lunch around making developers lives easier/better (aka. DevOps). One topic that came up was that when developing a [Django](http://djangoproject.com) based application using [Vagrant](http://vagrantup.com) in a team environment, that has some very terminal-savvy users and then the developers that prefer web based interfaces, a challenge for the less terminal-savvy users is managing Django's built-in development server. It is often easy to loose track of the terminal when you don't use it in most of your other development workflows and compile time exceptions can render the server unusable, prompting frustration.

Over one of these lunches I had the idea to run the Django dev server in [Tornado](http://tornadoweb.org) and provide a web based console with controls to force terminate & restart the dev server, and after about 5 hours today I have a working package that does the job quite nicely (if I do say so myself).

### How it works

Tornado is an amazing framework for async HTTP purposes and luckily for us they [already provide an interface to Python's subprocess module](http://www.tornadoweb.org/en/stable/process.html#tornado.process.Subprocess) that allows you to use a "STREAM" type for STDOUT & STDERR data from the command you launch. This STREAM type turns these two filehandles into special "IOStream" instances, aptly called "PipeIOStream".

From here, it's just a matter of hooking up to the IOStream "read_until" function to capture new lines coming out of the dev server.

This captured output is put into a cache and exposed over a WebSocket handler. When a client connects all of the data in the cache is replayed to them, then on any new data it is pushed to them in realtime over the socket.

There is a second WebSocket handler that provides status update (indicate if the process is running and provide a human friendly status, ie. "Django is running"). Couple this with handlers to terminate and start the development server and we're off to the races!

### What it looks like

![Screenshot](/assets/django-fujita/screenshot.png) 

In the first tab is the Fujita Console showing the log of the request to "/", and in the second tab is the corresponding "Welcome to Django" page that generated it.

### Code

You can find all of the code up on GitHub.

https://github.com/fatbox/django-fujita

Test it out. Feedback is welcomed!
