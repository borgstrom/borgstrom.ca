---
layout: post
title: What to do when you lock yourself out of gitolite?
date: 2011-11-08 12:16
disqus_identifier: 12516723826
---

Today, during some reorganization of my ‘keydir’, I accidentally locked myself out of RW access on the ‘gitolite-admin’ repository.

*Doh!*

Here was my general steps for recovery, with `gl-admin-push` being the secret sauce. These need to be run as the user that gitolite is running as.

1. mkdir /tmp/fix-gitolite
1. cd /tmp/fix-gitolite
1. git clone ~/repositories/gitolite-admin.git
1. cd gitolite-admin
1. vi conf/gitolite.conf
1. … fix stupid oversight …
1. gl-admin-push
1. cd ~
1. rm -fr /tmp/fix-gitolite

And now I can read & write to the gitolite-admin checked out on my workstation.
