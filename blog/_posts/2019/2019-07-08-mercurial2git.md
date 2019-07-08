---
layout: post
title: Convert Mercurial to Git repositories
description: "Notes on migrating Mercurial to Git repositories."
tag: Git Mercurial repositories
category: Git
date: 2019-07-08 13:13:17
---

Please follow the excellent description on the [Git documentation](https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git)

Here are some personal additions.

At the time of writing fast-export has a bug in v190107. An [issue](https://github.com/frej/fast-export/issues/132) is created.

The described workaround works for me as well.

My setup is:

- CentOS Linux release 7.6.1810 (Core)
- git version 2.22.0
- Mercurial Distributed SCM (version 2.6.2)
- Python 2.7.5

 I issued these commands:
 
 ```bash

# Install mercurial if not present and prepare a working directory.
sudo yum install -y mercurial
hg --version
mkdir hg2git-myrep
cd hg2git-myrep

# Get fast-export
git clone https://github.com/frej/fast-export.git
git checkout tags/v180317

# Clone mercurial repository and extract authors
# Use the --insecure switch if you are cloning from an internal 
# server with self-signed certificates
hg clone https://<url-to-my-hg-repository>/ tmp/hg-my-repository
cd tmp/hg-my-repository
hg log | grep user: | sort | uniq | sed 's/user: *//' > ../authors
cd ..

# Now sanitize the author list
vi authors

# Create and init git repository and import mercurial repo
mkdir git-my-repository
cd git-myrepository
git init
../../fast-export/hg-fast-export.sh -r ../hg-oltptest -A ../authors
# Check if have imports
git shortlog -sn
git branch
```

Now we have the repository with the complete history in git.

Next steps can be to (re-)organize the branches to match your Git's policy and to add a remote and upload 
the repository to your project's Git server.
