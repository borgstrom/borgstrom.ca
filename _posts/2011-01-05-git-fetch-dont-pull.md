---
layout: post
title: Git - fetch, don't pull
date: 2011-01-05 11:33
disqus_identifier: 2610217072
---

The more I use git on a day-to-day basis and the more I come to understand about it the better it gets.

Up until recently when I wanted to fetch changes from my origin I was using [git pull](http://www.kernel.org/pub/software/scm/git/docs/git-pull.html) to get the remote changes. I never did like that it forced me merge at the same time. It’s one of those things where when I was initially learning git all the documentation and examples used git pull, so I never questioned it.

While resolving a merge conflict I was in the man page for git pull and the old adage **RTFM** holds true again.

> More precisely, git pull runs git fetch with the given parameters and calls git merge to merge the retrieved branch heads into the current branch.

While I understand that pull may be convenient moving to using fetch and merge separately really allows you to control the whole merge process when you have multiple working branches locally.

### Step 1 - Fetch

<div class="highlight"><code><pre>(git:master)(#502:1u:0s) evan@borgstrom[/h/evan/projects/fatboxplatform]: git fetch origin
Enter passphrase for key '/home/evan/.ssh/id_rsa': 
remote: Counting objects: 299, done.
remote: Compremote: ressing objects: 100% (109/109), done.
remote: Total 216 (delta 157), reused 148 (delta 106)
Receiving objects: 100% (216/216), 24.14 KiB, done.
Resolving deltas: 100% (157/157), completed with 38 local objects.
From git@fatbox.fatboxes.com:platform
   ce6f550..3d3e3df  master     -> origin/master</pre></code></div>

With just the remote (`origin`) specified this step fetches all the changesets to your local repository, but does not apply them. I generally like to fetch all branches/refs and then merge them manually, but if you want just a specific branch you can specify it after the `origin` argument.

### Step 2 &ndash; Inspect

Now with the changesets fetched locally we can inspect and compare them.

<div class="highlight"><code><pre>git:master)(#518:1u:0s) evan@borgstrom[/h/evan/projects/fatboxplatform]: git diff master origin/master
diff --git a/accounts/managers.py b/accounts/managers.py
new file mode 100644
index 0000000..b6f3a48
--- /dev/null
+++ b/accounts/managers.py
@@ -0,0 +1,12 @@
+
+from django.db.models.manager import Manager
+
+
+class EmployeeManager(Manager):
+    '''A custom manager class for accounts.Employee model.'''
...</pre></code></div>

This is really the step I was missing using just pull. Since you can now refer to the changes as origin/master you can operate on it with any local branch.

Case in point: the changesets in my origin/master branch contained the following modifications: 25 files changed, 281 insertions(+), 130 deletions(-). 1 line of those 130 deletions was a fellow developer overwriting a subtle change from a previous changeset that had I not reviewed prior to merging may have gone unchecked for a while (it was a CSS change to a specific page that we do not have unit/functional tests for)

### Step 3 - Merge

Finally here. Let’s apply the change set to our current, local master branch.

<div class="highlight"><code><pre>(git:master)(#527:1u:0s) evan@borgstrom[/h/evan/projects/fatboxplatform]: git merge origin/master
Merge made by recursive.
 accounts/managers.py                            |   12 +++
 ...</pre></code></div>
