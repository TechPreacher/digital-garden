---
title: "Git: Clean up Already Checked-In Ignored Files"
draft: false
date: 2025-04-25
tags:
  - Coding
  - Learning
  - Git
  - GitHub
---
source: [https://www.git-tower.com/learn/git/faq/ignore-tracked-files-in-git](https://www.git-tower.com/learn/git/faq/ignore-tracked-files-in-git)

## Git: Cleaning up ignored already checked in files

```bash
git rm -r --cached .  
git add .  
git commit -m "Clean up ignored files"
```

