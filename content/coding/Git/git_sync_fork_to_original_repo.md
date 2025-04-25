---
title: "Git: Sync a Fork to the Original Repo"
draft: false
date: 2025-04-25
tags:
  - Git
  - GitHub
  - Learning
  - Coding
---
## TL;DR

```bash
# Add a new remote upstream repository
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git

# Sync your fork
git fetch upstream
git checkout master
git merge upstream/master
```

## Add a new remote upstream repository

You cloned your fork on your laptop and started to work on your issue.

Your fork is now an orphan. If you list the configured remote repository you will only see your fork as origin:

```bash
git remote -v
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin  https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
```


We need to configure this information to restore the family relationship by adding a new remote upstream repository:

```shell
git remote add upstream https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git
```

You can now see both the original repository and the fork:

```bash
git remote -v
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (fetch)
origin    https://github.com/YOUR_USERNAME/YOUR_FORK.git (push)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (fetch)
upstream  https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY.git (push)
```

## Sync your fork

Everything is now set up. You can sync your fork with only 2 commands.

Be sure you're in the root of your project and also in the master branch. Otherwise, you can check out to the master branch:

```bash
git checkout master
Switched to branch 'master'
```

Now, you have to fetch the changes from the original repository:

```bash
git fetch upstream
remote: Enumerating objects: 16, done.
remote: Counting objects: 100% (16/16), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 7 (delta 5), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (7/7), 1.72 Kio | 160.00 Kio/s, done.
From https://github.com/ORIGINAL_OWNER/ORIGINAL_REPOSITORY
   909ef5a..0b228a8  master     -> upstream/master
```

And merge the changes in your master branch:

```bash
git merge upstream/master
Updating 909ef5a..0b228a8
Fast-forward...
```

That's it! The fork is now up to date.