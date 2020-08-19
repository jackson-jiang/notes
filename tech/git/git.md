# Git手册

### 概念

​	Git是目前最流行的分布式VCS （版本控制系统，version control system），git管理的是`修改`而不是文件

​	git命令中“ -- ”的作用是区分，分割参数和值 

​	git的结构：

​	仓库：工作区，index，repository，refs（heads，remotes，tags...），objects（修改）,logs[logs, reflogs], hooks

​	分支：一系列的修改链形成的树结构，每个分支的HEAD指向链上的某个节点（通常是末端定点）， HEAD指针指向当前分支的HEAD。创建分支只是加了一个Head指针。分叉时 分支上有 start_point 指针



### 常用命令

​	参考文件git_cheetsheet.pdf

```bash
# []：可选， |：或， <>:变量
# 初始化
git init
git clone <仓库地址> # 推荐ssh方式，需1：生成sshkey 2：加入到server的可信列表

# 查看
git status
git log [--pretty=oneline] [--abbrev-commit] [--graph] [-p <file>]
git log --pretty=oneline --abbrev-commit --graph
git reflogs
# diff默认比较工作区和版本库， --cached比较index和版本库
git diff [<branch> [<branch>]|<commit> [<commit>] [--cached]
# 一天写了多少代码
git diff --shortstat "@{0 day ago}"
# 查看文件的修改历史
git blame [file]

# 提交
git add
git commit [-m] [-a] [-amend] # amend 修改上次提交

# 退回
## 清除缓存
git reset --hard HEAD
## 放弃工作区修改
git checkout -- <file>
## 退回版本
git reset --hard <hash> | <tag> | <HEAD>

# 分支管理
git branch <branch>
git checkout [-b] <branch>
git checkout - #切换到上一分支
git branch [-d|-dr|-D]  <branch> # -dr删除本地的远端分支缓存，远端的分支不删除
git branch [-a]
git branch --track <localbranch> <origin>/<branch>
## 从远端建立分支
git fetch <orign> <branch>
git checkout <branch> 
### 或 
git checkout -b <branch> --track <orign> <branch>
## 合并
git merge branch
git rebase [-i] <orign> <branch> # 分支的start point移到最新，之前的commit会作为补丁commit上去
# rebase可以修改之前所分支的提交和信息--用于整理tree
## 处理冲突后 git .add
git rebase --continue
git rebase --abort #退回到rebase前
git cherry-pick <commit>

# 分支 feature-cr-470 中包含了多次提交，中间还拉取了其他分支得代码。如何合并成一个提交
## 修改分支名称
git branch -m feature-cr-470bak
## 从目标分支新建功能分支
git checkout -b feature-cr-470
## 将代码合并到功能分支
## 代码会合并到feature-cr-470, 但是不会提交到仓库相当于add而不是commit。
[feature-cr-470] git merge --squash feature-cr-470
## 提交代码
git commit -m xxxx
## 推送到远端合并到目标分支（mr）

# 远端管理
## 查看远端仓库
git remote [-v]
git remote add
git fetch [<orign> <branch>]
git pull [--rebase] [<orign> <branch>]
## 更新/创建远端分支
git push [-u] [--force] [<orign> <branch>]
## 删除远端分支
git push <orign> :<branch>
## 创建远端tags
git push --tags
## 删除远端tags
git push origin :refs/tags/<tag>

# 本地修改暂存
git stash list
git stash [<message>]
git stash drop <message>
git stash apply <message>
git stash pop <message>

# Tag
git tag
git tag <tag> [hash] #tag打到之前的提交上
git tag -d <tag>
```





#### 项目管理策略

##### 流程 (FDD)

- Master分支：用于对外发布版本
- dev分支：用于内部开发代码管理


- <feature>分支【临时】：功能性分支从dev checkout，开发完成后合并到dev，稳定后可删除。
- <release>分支【临时】：一个版本的所有功能开发完并合并到dev分支后，从dev checkout，用于测试和修复bug不再添加新功能。稳定后合并到master分支和dev分支，之后可删除。
- <bug>分支【临时】：用于维护master分支的bug，稳定后合并到master分支和dev分支，之后可删除。



##### 版本号

- 使用semver管理版本号：x.y.z[-p+b], p: alpha, beta, RC, GA
- 正式版(master)：v1.0.0， v1.0.0-alpha12, v1.0.0-RC24
- 快速开发版(dev)：v0.1.0+build20171011xxxxx
- 每日构建版(dev)：v0.1.6+build20171011xxxxx



##### 提交管理

- commit前要测试
- 每次提交应按照要求格式编写提交信息
- 相关功能的变化要在一次commit中提交
- 一次commit只包含一个独立的功能
- 不要提交没有做完的功能
- 一个issue一个分支（一个分支一次merge，一次merge一个commit）



##### 提交说明格式

```bash
<type>(<scope>): <subject>
// 空一行
<body>
// 空一行
<footer>
```



### 实战（how to contribute）

1. gitlab创建issue
2. 从issue新建分支
3. 本地checkout分支
4. 开发issue（完成n次本地提交）
5. 功能本地测试
6. 拉取最新代码（fetch + rebase）
7. 合并多个commit（rebase -i）
8. 推送到远程
9. 创建merge request
10. 讨论和修改（pair programming）--> 3
11. 合并到主干，解决冲突
12. merge完成，关闭issue
13. 删除issue分支




### 技巧

```bash
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

git config --global core.safecrlf=true
git config --global core.autocrlf=false

git config --global user.name
git config --global user.email
```

git bash中连按两次TAB会出现提示

鼓励频繁创建删除分支

切换工作内容时可以使用stash保存





```bash
$ npm install -g commitizen
# 项目文件夹下（npm）
$ commitizen init cz-conventional-changelog --save --save-exact
$ git cz # 提交

# 生成change log
$ npm install -g conventional-changelog-cli
$ conventional-changelog -p angular -i CHANGELOG.md -w
```



### 修改文件名

windows系统是不区分文件以及文件夹的大小写，例如：`Hello` 和`hello`是同一个名字，linux是区分的。一般git的服务器都是linux系统，写代码的机器一般都是windows系统。

如果在windows系统下工作，git项目的一个文件名大小写起错了`Hello.js`，通过文件系统的方式修改了F2或者删除后添加`hello.js`添加到仓库并push到远端时。gitlab上会看到（Hello.js和hello.js）两份文件。



解决办法

```bash
# 1. 根本办法试用git命令换名字
git mv 
# 2. linux系统可以看到多余的文件，删除后上传即可

# 3. 将仓库中所有提交移除，然后重新上传（所有提交历史被清空）
git rm -r --cached
```



### 参考

[【入门】Git教程-廖雪峰](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

[【参考】git官网](https://git-scm.com/doc)

[Git分支管理策略](http://www.ruanyifeng.com/blog/2012/07/git.html)

[Commit message 和 Change log 编写指南](http://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)

[Git 工作流程](http://www.ruanyifeng.com/blog/2015/12/git-workflow.html)

[Git 使用规范流程](http://www.ruanyifeng.com/blog/2015/08/git-use-process.html)

[常用 Git 命令清单](http://www.ruanyifeng.com/blog/2015/12/git-cheat-sheet.html)

[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)

https://learngitbranching.js.org/?locale=zh_CN

