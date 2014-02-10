---
layout: post
title: "Python: fileinput module"
date: 2011-10-03 18:31
disqus_identifier: 10995017494
---

As a sys-admin & developer I regularly find myself needing to parse data supplied by clients that’s, uhm how to put it nicely, formatted like it’s supposed to be an email rather than structured data. This means I often find myself munging data; you know, read from STDIN or sometimes open a file, read it line by line, do something with the data. That kind of thing.

Today I was confronted with a rather unruly set of data and went looking for a better way to handle input in Python, and happened upon the fileinput library.

    import fileinput
    for line in fileinput.input():
        process(line)

Wow. Process STDIN OR a list of files line by line and only needing a single line of code!? [Batteries included](http://www.python.org/about/#python-is-powerful-and-fast) is right...
