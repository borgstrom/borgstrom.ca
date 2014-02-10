---
layout: post
title: Why isn't unarchiving built into CIFS & AFP yet...
date: 2012-11-09 14:28
disqus_identifier: 35351518844
---

&lt;rant&gt;

As I sit here and wait for a giant .zip file to expand onto my network file server it reminds me of how totally crazy this situation is!

I mean, I want to expand a zip file and I want its contents to end up on the file server. Presumably, if you can implement something as complicated as CIFS or AFP implementing at least zip &amp; rar decompression and extraction routines should be simple. The client should query the server for its capabilities and then ask the server to expand it. Done, fast, simple.

&lt;/rant&gt;
