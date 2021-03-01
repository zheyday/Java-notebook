---
title: git学习
tags: git
---

## 配置

```yml
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
# 创建ssh key
ssh-keygen -t rsa -C "youremail@example.com"

# 推送本地git到远程git
git remote add origin https://github.com/zheyday/git_study.git
git branch -M main
git push -u origin main
```

![1614575081289](git%E5%AD%A6%E4%B9%A0/1614575081289.png)

## 常用命令

### checkout

可以加节点、分支、文件名

![1614575499866](git%E5%AD%A6%E4%B9%A0/1614575499866.png)

### reset

左图处理节点或分支，右图处理文件

![1614575124450](git%E5%AD%A6%E4%B9%A0/1614575124450.png)

### amend

![1614568621645](git%E5%AD%A6%E4%B9%A0/1614568621645.png)

```yml
# init
git init
git add readme.md
# check status
git status
# check difference
git diff readme.md
git commit -m "description"
# 查看历史版本记录，时间由近到远 简化显示：--pretty=oneline
git log
# HEAD表示当前版本 HEAD^表示上一个版本 HEAD~100表示上100个版本
git reset --hard HEAD^
# 保存命令记录
git reflog
# 把所有更改添加进暂存区
git add --a
# bug分支
git cherry-pick ***
```



## 撤销修改

```yml
# 撤销修改两种情况：
1.修改后没有放到暂存区，撤销修改就是和版本库一样
2.放入暂存区后，又修改了，撤销修改就是回到添加到暂存区的状态
git restore readme.txt
# 放到暂存区之后撤销
git restore --staged readme.txt

# 工作区删除文件后，
# 1 要在版本库删除
git rm t1
git commit -m 'remove t1'
# 2 如果是删错了，需要恢复
git checkout -- t1
```



## 分支管理

git里有个主分支即master，Head指向master，master指向提交

```yml
# 创建分支，并且切换到分支
git switch -c dev
git add --a
git commit -m'test'
# change branch
git switch master
git merge master
# look branch
git branch
# delete branch
git branch -d dev
git push -u origin master
git push origin dev
# 暂存工作现场
git stash
# 查看
git stash list
# 恢复工作现场 
# 1、恢复后不删除
git stash apply
# 2、恢复后删除
git stash pop

```

## 标签

```yml
git tag v1.0
# 通过commit id添加
git tag v0.9 f52c63
# 指定标签信息
git tag -a <tagname> -m "blablabla..."
# delete tag
git tag -d v1.0
# 推送某个到远程
git push origin v1.0
# 推送全部
git push origin --tags
# 删除远程标签
git push origin :refs/tags/v1.0
```

## 别名

```yml
git config --global alias.st status
git config --global unset alias.st
status ==> st
commit ==> cm
restore --staged ==> unstage
```



## 常见错误

1. <font color='red'>git push报错error: failed to push some refs to 'git@github.com:'</font>
   解决：git pull --rebase origin master git push origin master
2. Push to origin/master was rejected
   解决：git push -u origin master -f 