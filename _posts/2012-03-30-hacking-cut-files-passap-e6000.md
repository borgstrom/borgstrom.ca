---
layout: post
title: Hacking .cut files for the Passap E6000
date: 2012-03-30 15:25
disqus_identifier: 20180956227
---

I’ve recently gotten involved in a project that involves building some custom software to do some image manipulation and some hardware to control an amazingly cool machine; [The Passap E6000](http://www.knittingmachinemuseum.com/Passap_E6000.php). Watching this thing in action is awesome to say the least. I’ll be posting more about it as we progress on the build, but for know I’m just going to talk about the first real aspect of the software development part.

A .cut file is a simple image file format that can be used store very basic image data plotted in pixels and using a limited 256 colour palette. These files can be loaded onto the E6000 via a serial cable and there’s a couple different pieces of software out there that can both create .cut files and download them to the E6000 but we’ve been using [Win_Crea](http://www.offthestreet.net/Win_Crea/WiCrDwnl.php) for our testing.

The first step in our experimentation was to learn to decode the `.CUT` files. The files use a run length encoding scheme built upon single bytes in a file. You can find the full details here: [.cut file details](http://www.fileformat.info/format/drhalo/spec/f735fa9d62f348058fe6f6a70aca4f9e/view.htm)

And after a day of tinkering around with it… Success!

![Success](/assets/hacking-cut-files/success.png)

The window on the left is Win Crea showing a simple pattern I designed in the grid editor and the window on the right is Preview on OSX showing the PNG I generated using some python code & PIL.

More to come as we continue to put this project together.
