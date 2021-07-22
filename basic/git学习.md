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
git remote add origin git@github.com:zheyday/git_study.git
git branch -M main
git push -u origin main
```

## 撤销修改

![1614575081289](git%E5%AD%A6%E4%B9%A0/1614575081289.png)

```yml
# 撤销修改两种情况：
1.修改后没有放到暂存区，撤销修改就是和版本库一样
2.放入暂存区后，又修改了，撤销修改就是回到添加到暂存区的状态
git restore readme.txt
# 从master恢复暂存区，工作区不变
git restore --staged readme.txt
# 工作区删除文件后，
# 1 要在版本库删除
git rm t1
git commit -m 'remove t1'
# 2 如果是删错了，需要恢复
git restore t1
```

## 常用命令

### checkout

可以加节点、分支、文件名

![1614575499866](git%E5%AD%A6%E4%B9%A0/1614575499866.png)

### reset

左图处理节点或分支，加上`--hard`参数会恢复到工作区

右图处理文件

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

git add -A  提交所有变化
git add -u  提交被修改(modified)和被删除(deleted)文件，不包括新文件(new)
git add .  提交新文件(new)和被修改(modified)文件，不包括被删除(deleted)文件
```

## 分支管理

git里有个主分支即master，Head指向master，master指向提交

```yml
# 创建分支，并且切换到分支
git switch -c dev
git add --a
git commit -m 'test'
# 推送本地分支到远程
git push --set-upstream origin 分支名
# change branch
git switch master
#合并dev分支到当前分支
git merge dev
# 查看分支
git branch
# 修改
git branch -m oldName newName
# delete branch
git branch -d dev
# 删除远程分支
git push origin --delete 分支名

#拉取特定分支
#方法一、
git clone -b dev 代码仓库地址(dev是分支名称)
#方法二
git init
git remote add origin 仓库地址
git fetch origin dev(dev是分支名称)
git checkout -b localdev origin/dev(localdev是本地分支名)
git pull origin dev

#查看对应的远程分支
git branch -vv

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
# 恢复指定stash
git stash apply stash@{0}
# 在main分支修复了bug，在dev肯定也存在，那么在dev上执行一次同样的操作即可
git cherry-pick 4c805e2
```

## 标签

```yml
git tag v1.0
# 通过commit id添加
git tag v0.9 f52c63
# 指定标签信息
git tag -a <tagname> -m "blablabla..." f52c63
# delete tag
git tag -d v1.0
# 推送某个到远程
git push origin v1.0
# 推送全部
git push origin --tags
# 删除远程标签，要先从本地删除，再删除远程
git tag -d v1.0
git push origin :refs/tags/v1.0
```

## 别名

```yml
git config --global alias.st status
git config --global unset alias.st
status ==> st
commit ==> cm
branch ==> br
restore --staged ==> restaged
```



## 常见错误

1. <font color='red'>git push报错error: failed to push some refs to 'git@github.com:'</font>
   解决：
   
   ```cmd
   git pull --rebase origin master 
   git push origin master
   ```
   
   Push to origin/master was rejected
   解决：git push -u origin master -f 