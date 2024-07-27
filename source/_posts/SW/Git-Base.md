---
title: git工具使用指北
date: 2021-03-31 21:40:59
index_img: /img/post_pics/os/git.png
tags:
    - Git
    - Tool
categories: 
    - Software
---

Git常用命令

<!-- more -->  

- [Beginner’s Guide to proper Git Workflow](https://medium.com/@anjulapaulus_84798/beginners-guide-to-proper-git-workflow-35a2d967734e)

```bash
git config --global color.ui auto

git config --global user.name "sjl3110"
git config --global user.email "2823626197@qq.com"

git add [file]
git restore [file]              #discard changes in working directory
git commit -m "[text]"
git commit --amend
git commit -a
git commit -am "[text]"

git log
git log --pretty=short
git log [file]
git log -p                      #see diff

git diff
git diff HEAD

git status

git pull origin master
git push -u origin master

git branch                      #list all branched
git checkout -b feature-A       #create a new branch and switch into it
git checkout [branch]           #switch to another branch


git reset --hard [before feature-A is created]
git checkout -b fix-B           #add fix-B
git reflog
git reset --hard [before merge]
git checkout master
git merge --on-ff fix-B

git checkout -b feature-C
git rebase -i HAED~2            #mix two commits into one(pick -> fixup)

git remote add origin url
git push -u origin [branch]
git branch -a
git checkout -b [branch] origin/[branch]
```
