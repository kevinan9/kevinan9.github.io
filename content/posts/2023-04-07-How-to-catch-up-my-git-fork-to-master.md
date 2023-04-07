---
title: "How to catch up my git fork to master"
date: 2023-04-07
tags: ["git"]
authors: ["kevinan9"]
---

# How to catch up my git fork to master

![](https://garygregory.files.wordpress.com/2016/11/git_logo.png?w=300&h=300)

## Configuring a git remote for a fork
You only need to do this once: Add a new remote upstream repository to sync with the fork where ORIGINAL_OWNER is the original GitHub account and ORIGINAL_REPOSITORY is the original repository name.

```
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

For example:

```
git remote add upstream https://github.com/apache/thrift.git
```

You can verify that all went well:

```
git remote -v
origin https://github.com/kevinan9/thrift.git (fetch)
origin https://github.com/kevinan9/thrift.git (push)
upstream https://github.com/apache/thrift.git (fetch)
upstream https://github.com/apache/thrift.git (push)
```

## Catching up a git fork to master
Fetch project branches from the upstream repository to get all the commits. Your commits to master will be stored in the local branch upstream/master.

```
git fetch upstream
```

Check out the master branch from your local fork.

```
git checkout master
```

Now merge the changes from upstream/master into your local master branch. Your fork’s master branch will be in sync with the upstream repository. You will not lose your local changes.

```
git merge upstream/master
```

All done!


Refs:
https://garygregory.wordpress.com/2016/11/10/how-to-catch-up-my-git-fork-to-master/

\- The End -