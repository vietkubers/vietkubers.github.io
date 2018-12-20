---
title: How to create and apply a patch with Git
tags: [tutorials]
keywords: tutorials
last_updated: October 10, 2018
sidebar: mydoc_sidebar
permalink: 2018-10-10-how-to-create-and-apply-a-patch.html
folder: mydoc
---

If you are an upstream developer or software engineer who works with Open Source Projects, definitely you have to use [Git](https://git-scm.com/) for collaborating and contributing your codes to upstream source codes. This article will show you the way to create and apply a patch with Git.


## 1. Create a branch for development
If you want to commit your codes to fix a bug or update some features, creating a new branch is the best solution for that.
For example, let clone my [sandbox](https://github.com/truongnh1992/sandbox) repository:
```sh
$ git clone https://github.com/truongnh1992/sandbox.git
$ cd sandbox
```	
Create a new branch: ***mybranch***
```sh
$ git checkout -b mybranch
Switched to a new branch 'mybranch'

$ git branch -a
  master
* mybranch
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/test
```
Update source codes from master branch
```sh
$ git checkout master
$ git pull origin master
$ git checkout mybranch
$ git rebase -i master
```
	
## 2. Create the patch
In the new branch ***mybranch***, let update source codes, readme,... and whatever you want to fix.
I've just created a commit.
```sh
$ git log
commit d0e4fcb56e2eeca01296cb9f9b78b7548280d517
Author: Nguyen Hai Truong <truongnh@vn.fujitsu.com>
Date:   Tue Oct 9 20:01:52 2018 -0700

    Update readme

    Signed-off-by: Nguyen Hai Truong <truongnh@vn.fujitsu.com>
```
Now, it's time to create a patch for above commit.
```sh
$ git format-patch -1 --stdout > my_patch.patch
```
It will create a patch file has name ***my_patch.patch*** in the current directory.
```sh
$ vim my_patch.patch

From d0e4fcb56e2eeca01296cb9f9b78b7548280d517 Mon Sep 17 00:00:00 2001
From: Nguyen Hai Truong <truongnh@vn.fujitsu.com>
Date: Tue, 9 Oct 2018 20:01:52 -0700
Subject: [PATCH] Update readme

Signed-off-by: Nguyen Hai Truong <truongnh@vn.fujitsu.com>
---
 readme.md | 1 +
 1 file changed, 1 insertion(+)
 create mode 100644 readme.md

diff --git a/readme.md b/readme.md
new file mode 100644
index 0000000..5c2920f
--- /dev/null
+++ b/readme.md
@@ -0,0 +1 @@
+Test repository
--
2.7.4
```
By default, **Git** create a separate patch file for each commit.

## 3. Apply the patch

Verify that the patch will change the source codes?
```sh
$ git apply --stat my_patch.patch

 readme.md |    1 +
 1 file changed, 1 insertion(+)

 $ git apply --check my_patch.patch
```
The above command will not apply the patch, it only shows you the stats about what it will do?
In order to apply the patch, you can use the command: `git apply`
```sh
$ git apply my_patch.patch
```
