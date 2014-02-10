---
layout: post
title: Monitoring disks with iostat and Cacti
date: 2010-02-18 08:48
disqus_identifier: 396594044
old_url: /post/396594044/monitoring-disks-with-iostat-and-cacti
---

There is two things I require when running any network:

1. Monitoring
2. Graphing

Without these two things you have no idea of the health of your machines and services, even worse forensic efforts become that much harder.

For one reason or another I decided to switch from [Cacti](http://www.cacti.net/) to [Munin](http://munin.projects.linpro.no/) about 8 months ago. Munin is a very capable monitoring solution, but I missed Cacti. The way it captures information and produces graphs may seem daunting to new users but the flexibility it affords is amazing and I really missed being able to change graphing parameters on the fly.

Last week while implementing statistics and monitoring for a new project we’re about to launch the decision was made to go back to cacti and dump our munin setup.

One of the first things I wanted to cover was the storage array I built. It is two linux servers in a master-slave HA pairing that are connected to two Sun J4400 Storage Arrays. Each array has 11 1TB disks and RAID10 is run across the array. Munin provided decent stats for the array, but I wanted more.

Enter the [ioscript scripts and templates from Mark Round](http://www.markround.com/archives/48-Linux-iostat-monitoring-with-Cacti.html).

The setup is pretty quick and painless, assuming you’ve already got Cacti setup and SNMP working on the host, and the result is excellent! (Secretly I’m a huge graphing nerd so things like this really get me excited).

Here’s what you end up with:

![Bytes per sec](/assets/iostat-cacti/bytes-sec.png)

![Request per sec](/assets/iostat-cacti/requests-sec.png)

![Times](/assets/iostat-cacti/times.png)

![Utilization](/assets/iostat-cacti/util.png)
