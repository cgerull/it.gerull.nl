---
layout: post
title: Git rebase
description: "A brief description how to simplyfy Git merge operations."
tag: Git
category: Git
date: 2016-03-23 22:23:17
---

The git rebase command is one of the more powerfull and advanced commands. You can use it to avoid overly complex merge operations. The common practice in Git is to do lot's of small commit's. But when your code builds fine and you don't need that detailed piece of history anymore, you can compact these small commits into one or two big commits.

The ultimate source and authority on all Git commands is [*Pro Git*](https://git-scm.com/book/en/v2) the Git book.

## Rebase before merge

A good practice is to always rebase before a merge. Git will produce a more comprehensive timeline. Especiallyt with with larger branches it will make it easier to fast forward.
What you do with rebase is to move your point in time where you branched. The less changes are on the timeline you want to merge (back) to, the chance of conflicts you have.

### Rebase to simplify merging

We have a master and one branch b1. After adding a third file and some content you needed to go back to master for some other work.

```shell
$ git lg
* 6350dfa (HEAD, master) Add fourish content
* 217709e Add fourth file.
| * f20dfb3 (b1) Add more content
| * 2e5d2df Add third file.
|/  
* df37868 Add second file
* fd37cc9 Initial commit.
```

If you merge b1 now into the master you end up with a hump in your history line. That is, of course, the historical correct representation. But in practice lot's these little bumps make your timeline harder to understand. As we have made changes to different files, the order doesn't really matter. (As long as you no major change to dependencies between those files.)
To rewrite history and straighten your timeline, checkout branch b1 first.

```shell
$ git checkout b1
$ git lg
* 6350dfa (master) Add fourish content
* 217709e Add fourth file.
| * f20dfb3 (HEAD, b1) Add more content
| * 2e5d2df Add third file.
|/  
* df37868 Add second file
* fd37cc9 Initial commit.
```

Now execute the rebase command. What this command actually does goes as follow. 
First the changes are save to a temporare place. The commit df37868 is checked out. Now Git applies the commits 217709e and 6350dfa. At last the saved changes 2e5d2df and f20dfb3 are applied.
The repository contains the same changes, but looks a lot clearer.

```shell
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Add third file.
Applying: Add more content
$ git lg
* de27624 (HEAD, b1) Add more content
* 39dde4e Add third file.
* 6350dfa (master) Add fourish content
* 217709e Add fourth file.
* df37868 Add second file
* fd37cc9 Initial
```

This is a very  simple example. More complex scenarios are in Pro Git. Just imagine working on new features but get interupted by a couple of changes / fixes to the latest release.

## Rebase on pull

To simplify the incorporation of changes from your team use git pull --rebase for every pull you made. Git will automatically check if can rebase the changes and keep your history simple.

Make your life easier and put this command in your config file.
 git config --global pull.rebase true.

## Don't mess with time

As convenient as rebase is, you are changing local history. When the commits your are moving on the timeline only exist in your local repository nothing bad happens.
But if you try to rebase commits that are already pushed to upstream repositories you probably get in trouble. The Pro Git book is quite elaborate on these cases and the documentation to read if you want to dive deeper into rebasing.
