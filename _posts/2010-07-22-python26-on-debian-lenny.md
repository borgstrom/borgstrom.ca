---
layout: post
title: Python 2.6 on Debian Lenny (5.0)
date: 2010-07-22 10:51
disqus_identifier: 845359613
---

Python 2.5 is REALLY starting to show it's age now. <a target="_blank" href="http://packages.debian.org/squeeze/python">Python 2.6 is in testing</a>, but anyone who's tried to mix and match Python packages from stable &amp; testing already knows that it never really seems to work out...

I'm working on a project that includes a piece of software that requires Python &gt;=2.6, and since I have no intentions of moving to <a target="_blank" href="http://www.debian.org/releases/testing/">Debian Squeeze</a> on my production machines it has to be compiled by hand.

I've seen <a target="_blank" href="http://serverfault.com/questions/111337/how-do-i-install-python-2-6-on-debian-5-0-lenny-using-apt-get">lots</a> <a target="_blank" href="http://blog.pythonaro.com/2008/10/horrible-hack-to-get-python-26-on.html">of</a> <a target="_blank" href="http://projectdaenney.org/2009/building-python2-6-for-debian-lenny">posts</a> online trying to integrate Python 2.6 into the core system (binaries in /usr/bin, libraries in /usr/lib, etc) by compiling Python with a <code>--prefix=/usr</code>, but that's another sure fire way to cause issues down the road IMHO.

So, we're going to compile Python 2.6.5 into <code>/usr/local</code> and then use virtualenv for all of our project dependancies thus keeping everything neatly away from the core system.

First lets get some dependancies. We can ask apt to install the dependancies required to build Python 2.5, which is actually pretty much the same as what's required for Python 2.6 and ensures you get a Python install that some what matches the Debian one.

    apt-get build-dep python2.5

This may take awhile as there's quite a bit of dependancies. Once it's completed grab a copy of the <a target="_blank" href="http://www.python.org/download/releases/2.6.5/">Python 2.6 sources</a>. We're using 2.6.5 as it's the most current of the 2.6 releases at the time of writing.

    mkdir -p ~/src/
    cd ~/src/
    wget http://www.python.org/ftp/python/2.6.5/Python-2.6.5.tar.bz2
    tar xjpvf Python-2.6.5.tar.bz2
    cd Python-2.6.5

Next we need to configure and build.

    ./configure --with-threads --enable-shared
    make
    sudo make install

Once the software is built and installed you'll likely notice that you can't yet run python.

    (#1:5u:0s) evan@corp-cloud[/home/evan]: python --version
    python: error while loading shared libraries: libpython2.6.so.1.0: cannot open shared object file: No such file or directory

This is because your system is not configured to look in <code>/usr/local/lib</code> for libraries. If we force it on the command line you can see it works.

    (#2:5u:0s) evan@corp-cloud[/home/evan]: LD_LIBRARY_PATH="/usr/local/lib" python --version
    Python 2.6.5

The solution is to configure the shared library loader to look in /usr/local/lib by default.

    (#3:5u:0s) root@corp-cloud[/home/evan]: echo /usr/local/lib > /etc/ld.so.conf && ldconfig

    (#5:5u:0s) evan@corp-cloud[/home/evan]: python --version
    Python 2.6.5

We now need to install setuptools using our new 2.6 Python, so we can further install packages.

    cd ~/src
    wget 'http://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz#md5=7df2a529a074f613b509fb44feefe74e'
    tar zxvf setuptools-0.6c11.tar.gz
    cd setuptools-0.6c11
    sudo python setup.py install

Next we install virtualenv and pip

    sudo easy_install virtualenv
    sudo easy_install pip

At this point you're ready to rock. You have a system with python 2.6, build tools &amp; headers, and a compatible virtualenv &amp; pip. To start work on a new project create a new environment with virtualenv and then install your required packages with pip.
