---
title: "git push 到遠端儲存庫(github)"
date: 2024-12-27 12:07:00 +0800
categories: 
  - git
tags:
  - github
---

## 加入遠端儲存庫

```bash!
# 建立名稱為 origin 的 github 遠端儲存庫(ssh 模式)
# git remote add <remote-name> <url>
git remote add origin git@github:allenat1981/django-tutorial.git

# 移除 remote
# git remote remove <remote-name>
git remote remove origin
```

### 補充：更改目前分支名稱

若有需要更改分支名稱

```bash!
# 更改目前所處分支的名稱
# git branch -M <new-branch-name>
git branch -M main
```

## 將分支 push 到遠端儲存庫

```bash!
# 將本地端分支 push 到遠端儲存庫
# git push -u <remote-name> <local-branch-name>

git push -u origin main

# -u 參數代表 upstream
# 可以想像成是替目前的 git 儲存庫設定一個遠端 push 的預設串流
# 若沒有設定 -u，則每次 git push 均需要指定 <remote-name> 和 <local-branch-name>
git push origin main

# 若有設定 -u，則若只輸入 git push，就會使用之前指定的 <remote-name> 和 <local-branch-name>

git push # 相當於 git push origin main

```

### 補充：指定 remote-branch-name

若要 push 本機儲存庫到遠端儲存庫，並指定不同的分支名稱時

```bash!
# 指定分支名稱
# git push <remote-name> <local-branch-name>:<remote-branch-name>
# 若指定的遠端分支不存在則會建立一個<remote-branch-name> 的分支
# e.g. 把本地端的 main 分支 push 到 remote 的 hello 分支

git push origin main:hello
```
