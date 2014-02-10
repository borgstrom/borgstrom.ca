---
layout: post
title: Installing kernels in Xen domU guests that boot using pygrub (Debian)
date: 2011-06-11 12:28
disqus_identifier: 6421879397
---

Today I was upgrading my development machine from Lenny to Squeeze and ran into a problem where installing the new kernel (linux-image-2.6 / linux-image-2.6-amd64) wouldn’t install complaining:

    /usr/sbin/grub-probe: error: cannot find a GRUB drive for /dev/sda1.  Check your device.map.

This link was the key to me figuring out what was going on: http://lists.bitfolk.com/lurker/message/20080529.142153.954fedf4.el.html

Apparently there’s no `/dev/sda` device setup in my domU, so no wonder grub was complaining about not being able to find the drive for `/dev/sda1`.

A simple mknod and we were on the way.

    borgstrom:/dev# ls -al sda*
    brw-rw---- 1 root root 8, 1 Jun 10 17:29 sda1
    brw-rw---- 1 root root 8, 2 Jun 10 17:29 sda2
    borgstrom:/dev# mknod sda b 8 0
    borgstrom:/dev# chmod 660 sda
    borgstrom:/dev# ls -al sda*
    brw-rw---- 1 root root 8, 0 Jun 10 17:38 sda
    brw-rw---- 1 root root 8, 1 Jun 10 17:29 sda1
    brw-rw---- 1 root root 8, 2 Jun 10 17:29 sda2

A note about the mknod command, specifically why we passed it 8 & 0. They are the device [major and minor numbers](http://www.linux-tutorial.info/modules.php?name=MContent&pageid=94), so in our case we just need to duplicate the 8 (since sda1 & sda2 were both major 8) and then set the minor to 0, which is what it should be for the base drive.
