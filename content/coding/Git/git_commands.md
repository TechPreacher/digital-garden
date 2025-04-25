---
title: "Git: Commands"
draft: false
date: 2025-04-25
tags:
  - Coding
  - Git
  - GitHub
  - Learning
---
## Set Upstream

```Bash
git branch -a
git checkout main
git branch
git remote add upstream https://github.com/x/y.git
git pull upstream main
```

## Squash all commits

```Bash
git switch yourBranch
git reset --soft $(git merge-base main HEAD)
git commit -m "Squash commits"
```

## Remove files ignored (.gitignore) but still checked in

You can remove them from the repository manually:

```Bash
git rm --cached file1 file2 dir/file3
```

Or, if you have a lot of files:

```Bash
git rm --cached `git ls-files -i -c --exclude-from=.gitignore`
```

But this doesn't seem to work in Git Bash on Windows. It produces an error message. The following works better (`-d '\n'` to split input line by line):

```Bash
git ls-files -i -c --exclude-from=.gitignore | xargs -d '\n' git rm --cached
```

If you are using PowerShell on Windows this works too (handles spaces in path and filenames):

```Bash
git ls-files -i -c --exclude-from=.gitignore | %{git rm --cached $_}
```

Regarding rewriting the whole history without these files, I highly doubt there's an _automatic_way to do it.
