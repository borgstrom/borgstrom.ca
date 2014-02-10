---
layout: post
title: Forcing a Linux kernel panic
date: 2012-10-17 01:02
disqus_identifier: 33758728773
---

During my testing of a project involving [Linux KVM](http://www.linux-kvm.org/) I needed a way to force a real kernel panic so that I could test some of our systems. After some research I came across [this blog comment](http://archive.geekworld.co.za/node/277#comment-858) that is a simple, straight forward way to cause a *real* kernel panic through the use of a custom kernel module.

To do this you're going to need the basic build tools (make, etc) as well as your kernel headers. On Debian you can get these by running:

    apt-get install build-essential linux-headers-`uname -r` 

Once you have your build tools setup make a new directory to do our work in, I called mine: `force_panic`

In this directory we're going to need to setup two files; a `Makefile` to build the module and the actual `.c` file that contains the code for the module. Their contents are below:

**Makefile**
{% highlight makefile %}
obj-m := force_panic.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

default:
	$(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules
{% endhighlight %}

**force_panic.c**
{% highlight c %}
#ifdef __KERNEL__

#include <linux/module.h>
#include <linux/kernel.h>

static int __init panic_init(void)
{
panic("force-panic");
return 0;
}

static void __exit panic_exit(void)
{
}

module_init(panic_init);
module_exit(panic_exit);

#endif
{% endhighlight %}

Now to build the module just issue the command:

    make

Once the build process is completed you should be left with a new file named force_panic.ko and you can panic your system by running:

    insmod ./force_panic.ko

**BIG FAT WARNING** -- The instant you hit enter on the above insmod command your system will halt. You made it panic after all...

Greets to the author of the original post & comment.
