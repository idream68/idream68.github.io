---
title: git 在不删除.git目录的情况下删除提交历史
date: 2021-05-07 20:10:51
categories:
  - git
tags:
  - git
---

```
# Check out to a temporary branch:
git checkout --orphan TEMP_BRANCH

# Add all the files:
git add -A

# Commit the changes:
git commit -am "Initial commit"

# Delete the old branch:
git branch -D master

# Rename the temporary branch to master:
git branch -m master

# Finally, force update to our repository:
git push -f origin master
```

若上述方法失败，则删除 .git 文件夹，并重新生成 .git 