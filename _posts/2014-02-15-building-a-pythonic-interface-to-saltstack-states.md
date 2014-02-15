---
layout: post
title: Building a Pythonic interface to SaltStack states
date: 2014-02-15 14:27
disqus_identifier: 20140213nacl
---

I've been a long time user of, and contributor to [SaltStack](http://saltstack.org), aka Salt. If you've never heard of Salt before it is an **amazing** remote execution and state application framework, suited for building highly reactive and very complex computing projects (it also works just fine for smaller projects too). I encourage you to check out the [Introduction to salt](http://docs.saltstack.com/topics/) for more information on the project.

By default you build your "state" and "pillar" data using the [YAML](http://yaml.org) markup language in conjunction with [Jinja2](http://jinja.pocoo.org/), which allows you some programatic flexibility to create macros and use filters for transforming data.

As you get your feet wet with Salt the YAML syntax and power of Jinja seem unlimited. I have built two large repositories of Salt states. YAML and Jinja have worked out quite well, but over the past year I've been reaching a point where I feel like I'm doing *too much* with YAML and Jinja, which means that I started to sacrifice the clean, readable syntax that was originally intended.

At the end of this past January (2014) while attending the inaugural [Salt Conf](http://saltconf.com) I started playing with some possible interfaces & syntax for a new way to build Salt states based on all of my learnings of the past couple years. At first this started as just some scratch notes and possibilities, but very quickly evolved into what is (IMHO) a very clean and Pythonic way to build Salt states using **pure python** (see note below about the existing "pydsl" renderer).

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

The above example may seem a bit contrived but when you start to look at some of the insanely complex state files that the community is building ([hadoop example](https://github.com/saltstack-formulas/hadoop-formula/blob/master/hadoop/settings.sls), [tomcat example](https://github.com/saltstack-formulas/tomcat-formula/blob/master/tomcat/package.sls) & [apache example](https://github.com/saltstack-formulas/apache-formula/blob/master/apache/register_site.sls) -- to name a few) you can start to see how that simple readability of YAML starts to get lost in the noise of the Jinja syntax. Moving to a clean, pure Python implementation means that you get to drop all of the syntax markup and also benefit from the added flexibility of not being restricted by some of the limitations of Jinja (I'm not criticizing Jinja at all here, it's a template engine you're not supposed to have full access to Python).

What about the `pydsl` renderer that is already included in Salt? I actually didn't even know it existed until after I had the initial brainstorm session for NaCl and only learned about it during SaltConf when I looked into the existing py renderer to see how it formatted data. Personally, I dislike the choice of building a Domain Specific Language for Salt state creation, I think that DSL's are way over hyped these days and I switched to Salt in the first place because I came to be very frustrated by the Puppet DSL. I find the `pydsl` renderer is awkward to use and not very Pythonic since it requires you to chain method calls, more akin to Javascript and jQuery.


## Introducing NaCl

With the idea now fleshed out I set out to build a proper implementation. Over the course of a week I built up the first working version that I decided to call NaCl (the formula Sodium Chloride).

You can find the sources up on GitHub: https://github.com/borgstrom/nacl

Before I even posted this announcement @mgwilliams [found it and contributed](https://gist.github.com/mgwilliams/8986670) to the first fully working version, so big greets to him for that!

I would love any feedback or suggestions for this project. Please feel free to use the [issue tracker on GitHub](https://github.com/borgstrom/nacl/issues).
