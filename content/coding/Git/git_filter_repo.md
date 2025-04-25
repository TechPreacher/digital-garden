---
title: "Git: Filter Repo"
draft: false
date: 2025-04-25
tags:
  - Coding
  - Learning
  - Git
  - GitHub
---
source: [https://github.com/newren/git-filter-repo](https://github.com/newren/git-filter-repo)

## Git Filter-Repo

git filter-repo is a versatile tool for rewriting history, which includes [capabilities I have not found anywhere else](https://github.com/newren/git-filter-repo#design-rationale-behind-filter-repo). It roughly falls into the same space of tool as [git filter-branch](https://git-scm.com/docs/git-filter-branch) but without the capitulation-inducing poor [performance](https://public-inbox.org/git/CABPp-BGOz8nks0+Tdw5GyGqxeYR-3FF6FT5JcgVqZDYVRQ6qog@mail.gmail.com/), with far more capabilities, and with a design that scales usability-wise beyond trivial rewriting cases. [git filter-repo is now recommended by the git project](https://git-scm.com/docs/git-filter-branch#_warning) instead of git filter-branch.

### Installing on Windows

- Copy to a folder that is in the path.
- Move Python 3 to front of Path (so it gets executed when typing "python")
- Deactivate the installation-on-demand handling for _Python.exe_ and _Python3.exe_ under _Start > Manage app execution Aliases_
- Make sure the first line of _git_filter_repo_ reads:

```bash
#!/usr/bin/env python
git filter-repo -h
```

Test by running

```bash
git filter-repo -h
```
