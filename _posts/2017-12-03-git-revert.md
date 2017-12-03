---
layout: post
title:  "About reverting Git commits"
date:   2017-12-03 19:00:00 +0100
categories: git
---

Today I created a new repository on Github and I wanted to upload some local
code. However I messed it up and created the first commit locally too soon,
so I needed to revert it to fix things.

This is when I realized that the recommended way to revert a commit, in the
case where we don't want to also undo the corresponding changes, is:

```
git reset <old commit>
```

This tells git to set HEAD to a different commit than the current one, and
anything that comes after that reverts to a local uncommitted diff.

But what if the commit you want to revert is the first one? You have no previous
commit hash to revert to.

The way I found is to delete the branch you implicitly created with your first
commit:

```
git update-ref -d HEAD
```

And all is good!
