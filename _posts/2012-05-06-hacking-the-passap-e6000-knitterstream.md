---
layout: post
title: Hacking the Passap E6000 - control panel & serial interface
date: 2012-05-06 09:20
disqus_identifier: 22513138258
---

![Hacked Passap E6000 Control Panel](/assets/knitterstream/control-panel.jpg)

For the past couple weeks I’ve been working on automating and hacking the [Passap E6000](http://www.knittingmachinemuseum.com/Passap_E6000.php) knitting machine with [Ivan](http://www.ivansharko.com/).

I have [previously posted about hacking on .cut files]({% post_url 2012-03-30-hacking-cut-files-passap-e6000 %}), but it turns out that the machine doesn’t even use .cut files. While Ivan & I were learning to knit we heavily used a piece of software called [WinCrea](http://www.offthestreet.net/Win_Crea/WiCrDwnl.php) and it lead us down the path of .cut files as that it is what WinCrea reads and writes by default. It turns out that it uses something even simpler, much simpler.

The E6000 uses a straight forward protocol that is represented by a specifically formatted ASCII string. The string describes the number of colours, columns & rows in the pattern and then just sends “0”, “1”, “2” or “3” to represent which colour should be used for that stitch. (For more details see the code, link to the GitHub repo is at the end of the post).

With the task of being able to directly send patterns to the E6000 over the serial interface behind us I needed to design and attach the arduino circuit that would allow us to interface with it.

The E6000’s main control panel is exposed and easily hackable inside the plastic housing. Minimal dis-assembly was required to get into it and the electronics for the buttons have through-holes (via’s) for each button that made attaching our own leads to them a breeze. *(Ok, maybe “breeze” was the wrong word. I forgot how out of practice I was at soldering so the first 20 - 25 solders were a total PITA until I found my rhythm again)*

The main component used in our circuit is an [Opto-isolator/Opto-coupler](http://en.wikipedia.org/wiki/Opto-isolator), which is essentially a digital switch that the arduino can control and allows us to trigger the buttons for any arbitrary period of time. Pins 5 & 6 of the opto-isolator attach to the leads to the button, pin 2 goes to ground and pin 1 goes to the arduino, via a 1K resistor to ensure that we don’t arbitrarily trigger the button press. ([This thread on the arduino forums](http://www.arduino.cc/cgi-bin/yabb2/YaBB.pl?num=1217077559) was most helpful in coming up with a design).

The final piece to the hardware hack was a simple arduino program that reads single bytes over the serial interface and translates them into button presses on our console, thus allowing us to trigger the buttons from our control machine. The code is small and straight forward, and can be found in the GitHub repository.

We’re ready to rock. Who’s ready to knit?

[KnitterStream code on GitHub](https://github.com/borgstrom/KnitterStream)
