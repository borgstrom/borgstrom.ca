---
layout: post
title: Open Source Monitoring != Free Puppies
date: 2010-07-12 11:30
disqus_identifier: 802346452
---

I don't often create posts to respond directly to other blog posts, but something about this one just rubbed me the wrong way.

[Free Puppies and Open Source Monitoring Solutions – Fight the Temptation](http://www.systemshepherd.com/aboutus/application-monitoring-blog/2010/06/27/3-free-puppies-and-open-source-monitoring-solutions-fight-the-temptation)

Now, first things first. I understand that the author of that post works for a company that competes with the self hosted, DIY, solutions of the open source world. But, if you're going to make a comparison of open source monitoring/trending/alerting solutions to a Free Puppy then you're lying to end user just to try to sell your service.

See, the thing is that all software ([FOSS](http://en.wikipedia.org/wiki/Free_and_open_source_software) or otherwise & [SaaS](http://en.wikipedia.org/wiki/SaaS) or otherwise) requires an administrator who understands the goal of deploying the software. The post describes a scenario where the FOSS software is downloaded and the expectation is that it should run flawlessly out of the box. In a decade of running networks where downtime is not an option I have NEVER (EVER) had a piece of software just work out of the box. Instead it requires an in-depth understanding of the goals you're trying to achieve, a properly laid out road map of how to get there, management that understands the business value of it and finally an admin (or team of admins) that uphold these values and are given realistic timelines.

Back to the point at hand.

> Two weeks later and you finally finish the setup and install, and now, at last, the definition of the metrics, thresholds and alarms and if there is time, you can play with the GUI and setup the network and systems topology map (your boss will be dazzled).

Ok, so your point here is that it took the admin two weeks to get the software installed. This sounds to me like an admin who was in over his/her head or a management team that doesn't understand the business value monitoring and trending provide and are willing to accomodate it. Either way, I don't see how this is the fault of the open source solution.

> Oops!  With all of the metrics and thresholds you defined the spare server that you are using to run the free software is way slower than expected.  You decide that it will have to do for now, and when you get some extra budget you can upgrade the server; heck while you’re at it let’s do it right and set up a High Availability environment – more budget.

I'm not sure what type of things you're monitoring, but from my experience a properly deployed solution is not that resource intensive. One of my systems monitors hundreds of hosts and thousands of services on a virtual machine with 384Mb of RAM and the load barely ever peaks above 0.2 (yes, zero point two).

![Load Average of 0.2](/assets/open-source-monitoring/load_avg.png)

Then you say high-availability is doing it right, I say getting your monitoring into a data center that is not where you run your production, staging or development instances is the right way to do it - then you can worry about high availability. Monitoring is pretty useless unless you're monitoring it from the point of view of the general internet, or your customers, and with all of the [IaaS](http://en.wikipedia.org/wiki/IaaS#Infrastructure) providers offering instances for less than $30 per month I find the argument about doing it yourself requiring more budget moot. (Especially, because to monitor hundreds of hosts through your service would cost FAR more than the infrastructure and head count if done properly in house) [based on [your pricing page](http://www.systemshepherd.com/whysystemshepherd/packagecomparison)].

Don't get caught up in the FOSS bashing hype. Open Source is an amazing foundation to build upon, but that's just it. IT'S A FOUNDATION. It requires that both the admins and management be on board with the ideals and challenges Open Source provide and are willing to accomodate it in order to reap the benefits. In the end you can allocate your budgets to head count of talented admins that can do more than just monitoring and will be able to provide more value across the entirety of your business objectives instead of buying to into vendor proprietary solutions.
