---
layout: post
title: Making a bootable USB key from an .iso image on Mac OS X
date: 2010-10-14 14:10
disqus_identifier: 1314205955
---

#### UPDATE - July 4th, 2012

After numerous comments on this post I have made some updates that will both make the process simpler and speed up the copy of the image to the USB key. All updates have been marked as such and I have put a strike through any text I have removed.

-----

This was WAY harder than it should've been to find all this info. All the info out there was either wrong, didn't work or incomplete. I ended up having to read the source code comments from Michael Forrest's [USB Creator For OS X](https://launchpad.net/ubuntuusbcreator-osx) project to figure it all out. While his approach of doing it through a GUI is nice, OS X contains all the tools you need to do it from a terminal, which suits me just fine.

Hopefully this document will work its way up Google so other people will find it easier than I did.

### Purpose

Just to make sure we're all on the same page here, the purpose of this exercise is to take an [ISO image](http://en.wikipedia.org/wiki/ISO_image) that you've downloaded and contains a bootable filesystem intended to be burned onto a CDROM. Instead we're going to put it on a USB key so that a machine with the ability to boot from USB keys can boot it the exact same as if it were a CDROM.

*NOTE: Intel Based Apple Machines contain an EFI BIOS that __CANNOT__ boot USB keys. You will not be able to put your OS X .iso file onto a USB key and install it onto an Apple laptop (MacBook Pro/Air/etc).*

We'll be doing most of this from a terminal, with the help of disk utility for some things.

### Prepare the USB key

We're going to wipe the partition structure on the USB key. **WARNING! THIS WILL DESTROY ALL DATA ON THE KEY!**

Open up Disk Utility (it's in /Applications/Utilities/).

![Disk Utility](/assets/osx-usb/diskutil.jpg)

Now do the following:

1. Select the USB key (select the root device, not its partitions)
2. Select the partition section at the top
3. Change the Scheme to 1 Partition
4. Change the Format to Free Space
5. Click Apply

You will get a confirmation dialog appear ensuring you really want to delete all data on the key, choose Partition.

Once it's completed you can quit out of Disk Utility.

*Update: The purpose of doing this is mainly to ensure that the USB key is in a consistent known state and also to ensure that any volumes are not mounted by OS X. It is not required and you can skip it if you'd rather just unmount the volumes yourself.*

### Preparing the ISO image

Now that our USB key is ready, we need to get our .iso image into a format that we can copy to it. Open up a Terminal (it too is in /Application/Utilities, and I'll assume you know how to use the terminal).

Now, convert the image from a ISO to a [Read/Write Universal Disk Image Format](http://en.wikipedia.org/wiki/Universal_Disk_Image_Format) (or UDRW). Here I'm using the Debian 6.0.7 Net Installation ISO, but you can use anything else that's an ISO file.

<div class="highlight"><code><pre>[#542]: hdiutil convert -format UDRW -o debian-6.0.7-amd64-netinst.img debian-6.0.7-amd64-netinst.iso 
Reading Master Boot Record (MBR : 0)…
Reading Debian 6.0.7 amd64 1             (Apple_ISO : 1)…
Reading  (Windows_NTFS_Hidden : 2)…
.....................................................................................................................................................................................................................................................
Elapsed Time:  2.054s
Speed: 81.8Mbytes/sec
Savings: 0.0%
created: /Users/evan/Downloads/debian-6.0.7-amd64-netinst.img.dmg</pre></code></div>

Once completed this will create the .img file. The hdiutil function likes to append a .dmg suffix to the file so it will probably end up .img.dmg after conversion.

### Copy the image to the USB key

We're finally here. The easy part, actually copying the image to the USB key.

First run diskutil list to get a listing of the disks in your machine so you can identify the USB key. It will look like this:

<div class="highlight"><code><pre>[#543]: diskutil list
/dev/disk0
   #: TYPE NAME SIZE IDENTIFIER
   0: GUID_partition_scheme *250.1 GB disk0
   1: EFI 209.7 MB disk0s1
   2: Apple_HFS Macintosh HD 249.7 GB disk0s2

/dev/disk1
   #: TYPE NAME SIZE IDENTIFIER
   0: *1.0 GB disk1</pre></code></div>

Here mine's /dev/disk1.

*Update: We want to use the RAW disk device so that our copy will happen much faster because the RAW disk device provides unbuffered access to the device (See [this Apple mailing list post](http://lists.apple.com/archives/filesystem-dev/2012/Feb/msg00015.html) for more info). This is accomplished by simply prepending 'r' to the device so that /dev/disk1 is going to become /dev/rdisk1*

*Update 2: Specifying a blocksize of 1m will also significantly speed things up.*

Next we use the dd command to copy the image over.

<div class="highlight"><code><pre>[#544]: dd if=./xbmc-9.11-live-repack.img.dmg of=/dev/rdisk1 bs=1m</pre></code></div>

On the command line we specify the Input File using if= and the Output File using of= and dd will copy the data from input to output, block by block.

Once it's completed you can exit Terminal and remove the USB key from your OS X machine, it should now be able to bootup your ISO on another machine.
