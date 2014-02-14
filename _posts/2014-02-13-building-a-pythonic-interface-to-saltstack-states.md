---
layout: post
title: Building a Pythonic interface to SaltStack states
date: 2014-02-13 09:27
disqus_identifier: 20140213nacl
---

I've been a long time user of, and contributor to [SaltStack](http://saltstack.org), aka Salt. If you've never heard of Salt before it is an **amazing** remote execution and state application framework, suited for building highly reactive and very complex computing projects (it also works just fine for smaller projects too). I encourage you to check out the [Introduction to salt](http://docs.saltstack.com/topics/) for more information on the project.

By default you build your "state" and "pillar" data using the [YAML](http://yaml.org) markup language in conjunction with [Jinja2](http://jinja.pocoo.org/), which allows you some programatic flexibility to create macros and use filters for transforming data.

As you get your feet wet with Salt the YAML syntax and power of Jinja seem unlimited. I have built two large repositories of Salt states; one private for client data and [one public](http://github.com/freesurface/stackstrap-salt) with all kinds of logic for common setup tasks used in web apps. YAML and Jinja have worked out quite well, but over the past year I've been reaching a point where I feel like I'm doing *too much* with YAML and Jinja, which means that I started to sacrifice the clean, readable syntax that was originally intended.

At the end of this past January (2014) while attending the inaugural [Salt Conf](http://saltconf.com) I started playing with some possible interfaces & syntax for a new way to build Salt states based on all of my learnings of the past couple years. At first this started as just some scratch notes and possibilities, but very quickly evolved into what is (IMHO) a very clean and Pythonic way to build Salt states using **pure python** (see note below about the existing Python DSL).

Here's a simple little YAML snippet that will cause Salt to ensure the `/tmp` directory is in the correct state.

    /tmp:
      file.managed:
        - user: root
        - group: root
        - mode: '1777'

And here's the exact same specification using the new pure Python syntax.

    File.managed("/tmp", user='root', group='root', mode='1777')

As you can see you are calling a function named `managed` on the object named `File`, providing the state ID as the first positional argument and then any other attributes as keyword arguments.

Essentially what is happening here is that all of the available [state modules](http://docs.saltstack.com/ref/states/all/index.html) are made available as global objects as the capitalized name of the state. Then each of the functions are exposed as attributes of these global objects. These attributes are functions that take the "name" of the state as the first parameter and every other keyword argument passed becomes the data for the state.

Behind the scenes what is happening is that each call to one of these global objects registers the data provided into a "registry". When Salt parses the state file the registry is filled, then it formats this data as Salt expects it and finally the registry is drained, ready for the next file.

The above example may seem a bit contrived but when you start to look at some of the insanely complex state files that the community is building ([hadoop example](https://github.com/saltstack-formulas/hadoop-formula/blob/master/hadoop/settings.sls), [tomcat example](https://github.com/saltstack-formulas/tomcat-formula/blob/master/tomcat/package.sls) & [apache example](https://github.com/saltstack-formulas/apache-formula/blob/master/apache/register_site.sls) -- to name a few) you can start to see how that simple readability of YAML starts to get lost in the noise of the Jinja syntax. Moving to a clean, pure Python implementation means that you get to drop all of the syntax markup and also benefit from the added flexibility of not being restricted in what Python you have at your disposal under Jinja.

What about the `pydsl` renderer that is already included in Salt? Personally, I find the choice of building a Domain Specific Language for Salt state creation unnecessary, more specifically I don't even know if I consider the `pydsl` renderer a true domain specific language as it is really more of an Python specific interface (very similar in concept to my implementation here). More so, I find that the `pydsl` renderer is not a very Pythonic interface as it requires you to chain method calls, more akin to Javascript and jQuery. Why make power users who want to use the power of Python in building their states use an interface that is unfamiliar to them?

## Introducing NaCl

