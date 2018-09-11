---
layout: post
title: Merge 2 Git repositories
description: "Merge 2 unrelated repositories into one."
tag: Git
category: Git
date: 2018-09-04 12:23:17
---

Sometimes it is necessary to combine 2 different Git repositories with preservation of both histories. In this example I assume that we only have one active branch (master) around that none of the files are duplicates.

### Follow these steps

- Start with 2 fresh clones
- Switch into the repository you want to keep
- Add a remote that points to the URL of the repository you want to import
```
git remote add <tmp name of 2nd repo> <URL of second repo>
```
- Pull the second repo with 
```
git pull <tmp name of 2nd repo> master --allow-unrelated-histories
```
- The repositories are now merged.
- If necessary move anything around with `git mv`
- Remove the second remote entry 
```
git remote rm <tmp name of 2nd repo>
```
- Push the combined repository to the server.
