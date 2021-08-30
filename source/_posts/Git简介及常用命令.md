title: Git简介及常用命令
date: 2019-03-25 20:31:11
tags: [Git]
---

# 前言
最近想深入了解一下Git，阅读了一下[Pro Git](https://git-scm.com/book/en/v2)的前两章，前两章几乎涵盖了Git的大部分功能，在此做个记录，以便后续查阅

# Git介绍
Git实际上是一个版本控制系统，本质上，版本控制是一种记录一个或若干文件内容变化，以便将来查阅特定版本修订情况的系统。

版本控制经历了本地版本控制系统（RCS，我没听过），集中化版本控制系统（CVS，Subversion等）和分布式版本控制系统（Git等）的演变。
<!-- more -->
本地版本控制系统无法实现协同工作，集中化的版本控制系统解决了协同工作的问题，但中央服务器的单点故障会影响所有人的工作，分布式版本控制系统采用快照的方式，每个开发者电脑上都有一份代码的完整拷贝，故障恢复起来就比较容易。

Git是Linux开源社区的杰作，他们开发Git之初就对其有以下目标：
1. 速度
2. 简单的设计
3. 对非线性开发模式的强力支持（允许成千上万个并行开发的分支）
4. 完全的分布式
5. 有能力高效管理类似Linux内核一样的超大规模项目（速度和数据量）

Git和其他版本控制系统（如Subversion）的主要差别在于对待数据的方法。不像其他版本控制系统保存基本文件和基于基本文件的差异，Git直接保存每个文件的快照。

![存储每个文件和基础版本的差异](/img/存储每个文件和基础版本的差异.png)

![存储项目随时间变化的快照](/img/存储项目随时间变化的快照.png)

正因为如此，Git中的大多数操作只需要访问本地资源，而不需要与服务器进行通信，保证了操作的速度。

# Git基础
## Git的三种状态
在Git中，你的文件处于以下三种状态之一：
1. 已提交（committed），表示数据已经安全的保存在本地数据库中。
2. 已修改（modified），表示已经修改了文件，但还没保存在数据库中。
3. 已暂存（staged），表示对一个已修改文件的当前版本做了标记，使之包含在下次提交的快照中。

由此引入Git项目的三个工作区的概念：Git仓库，工作目录以及暂存区域。

![Git仓库，工作目录以及暂存区域](/img/工作目录，暂存区域以及Git仓库.png)

基本的 Git 工作流程如下： 
1. 在工作目录中修改文件。 
2. 暂存文件，将文件的快照放入暂存区域。 
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录。 

# Git常用命令
## Git帮助
有三种方式获取Git的帮助：
```
git help <verb>
git <verb> --help
man git-<verb> 
```

## Git配置
Git的配置信息存放在三个不同的位置
1. /etc/gitconfig文件，系统上每一个用户及他们仓库的配置，使用--system选项会读写此配置文件
2. ~/.gitconfig 或 ~/.config/git/config 文件：只针对当前用户。使用--global选项会读写此配置文件
3. 当前仓库中使用的配置（.git/config），需要在当前仓库进行修改，不需要加选项。

每一个级别的配置会覆盖上一个级别的配置。

以下是常用的配置
```
// 查看用户配置
git config --global -l

// 设置全局的用户名和邮箱
git config --global user.name "gwq5210"
git config --global user.email "gwq5210@qq.com"

// 设置使用的编辑器
git config --global core.editor "vim"

// 设置常用的别名，以下命令会在稍候介绍
git config --global alias.co checkout
git config --global alias.ci commit
git config --global alias.br branch
git config --global alias.st status
git config --global alias.unstage reset HEAD --
git config --global alias.last log -l HEAD

// 为Git设置代理
git config --global http.http http://127.0.0.1:8080
git config --global http.https http://127.0.0.1:8080

// 为不同的网址设置不同的代理
git config --global http.https://github.com.proxy http://127.0.0.1:8081
git config --global https.https://github.com.proxy http://127.0.0.1:8081

// 避免每次都输入账号密码，将账号密码保存在内存中几分钟
git config --global credential.helper cache 
```

## 获取Git仓库
有两种方式获取Git仓库：从现有目录初始化仓库和从服务器克隆一个现有的Git仓库。
```
// 从现有目录初始化仓库
git init

// 从服务器克隆一个现有的仓库
git clone [url] [name]
```

Git仓库里存在着.git目录，保存着Git仓库的必要信息。

## 查看当前文件状态、跟踪新文件和暂存已修改的文件
工作目录的文件状态不外乎：已跟踪和未跟踪。已跟踪是已纳入版本控制的文件，在上一次快照中有他们的记录，工作过一段时候后，他们的状态可能是未修改，已修改，已暂存状态。除此之外其他文件都属于未跟踪文件
![文件状态变化生命周期](/img/文件状态变化生命周期.png)

```
// 查看当前文件状态
git status
// 之前设置的status别名为st
git st

// 显式简介的状态
git status -s
git status --short
// 第一列表示在暂存区域的状态，第二列表示在工作区域的状态
-------------------
 M README
MM Rakefile
A lib/git.rb
M lib/simplegit.rb
?? LICENSE.txt 
-------------------

// 跟踪新文件或暂存已修改的文件
git add [path]
```

我们可以使用.gitignore文件来忽略我们不想跟踪的文件，这里有.gitignore文件的列表，可供查阅https://github.com/github/gitignore

## 查看已暂存和未暂存的修改
```
// 查看未暂存的修改，即工作区域和已暂存区域的修改
git diff
// 查看已提交和已暂存的修改
git diff --cached
// Git 1.6.1及更高版本
git diff --staged
```

## 提交更新
使用如下命令将暂存区域的内容提交
```
git commit
// 命令行指定提交消息
git commit -m "commit message"
// 跳过暂存，进行提交
git commit -a -m "commit message"
// commit的别名
git ci
```

## 移除或修改文件
要移除文件，需要从已跟踪清单中（确切的说是从暂存区域）移除，然后提交。
```
git rm [path]
-------------
// git rm类似如下命令
rm [path]
git add [path]
-------------

// 从暂存区域移除，仍然保留在工作区域
git rm --cached [path]

// 如果已经修改了文件或者已经暂存了该文件，则需要使用-f进行删除，防止误操作删除了不可恢复的文件
git rm -f [path]

// 重命名文件
git mv file_from file_to
-------------
// git mv相当于下边的操作
mv file_from file_to
git rm file_from
git add file_to
-------------
```

## 查看提交历史
提交若干更新或者克隆一个仓库之后，我们可以查看提交历史
```
// 默认会列出所有的提交历史，最近的更新会排在最上边
git log

// 显示最新的n条记录
git log -n

// 显示每次提交的内容差异
git log -p

// 查看每次提交的简略统计信息
git log --stat

// 单行显示提交历史
git log --oneline

// 使用图形提交历史
git log --graph
```

## 撤销操作
```
// 修改最后一次提交，使用这个选项会覆盖上一次的提交
git ci --amend

// 取消暂存文件
git reset HEAD <file>...
// 别名
git unstage <file>...

// 撤销对文件的修改，此操作会丢失对文件的修改，很危险
git checkout -- <file>...
```

## 使用远程仓库
克隆一个仓库，会自动给远程仓库添加origin的名称
```
// 显示远程仓库
git remote -v
git remote show <remote-name>

// 添加远程仓库
git remote add <shortname> <url>

// 拉取远程仓库的信息，不会自动合并或修改你当前的工作
// 分支设置了跟踪一个远程分支的话git pull会自动抓取然后合并
git fetch <remote-name>

// 推送到远程仓库
git push <remote-name> <branch-name>

// 远程仓库重命名和移除
git remote rename <old-name> <new-name>
git remote rm <remote-name>
```

## 使用标签
Git可以给历史中的某一个提交打上标签，表示重要
Git标签有两种标签，轻量标签和附注标签
轻量标签很像一个不会改变的分支，它只是一个特定提交的引用
附注标签是存储在Git数据库中的完整对象，可以包含打标签者的名字，邮件，日期等信息
通常建议创建附注标签
```
// 列出标签
git tag
git tag -l "v1.8*"

// 创建附注标签，指定存储在标签中的信息
git tag -a v1.0 -m "create tag v1.0"
// 查看标签的信息和对应的提交信息，轻量标签只会显示提交信息
git show v1.0

// 创建轻量标签，只需要提供标签名字
git tag v1.0-lw

// 对过去的提交打标签
git tag -a v0.9 9fceb02

// 共享标签，将标签推送到远程仓库，类似共享远程分支
git push <remote-name> <tag-name>

// 将所有不在远程服务器的标签都推送过去
git push <remote-name> --tags

// 删除标签，此命令不会删除远程服务器的标签
git tag -d v1.0

// 删除远程服务器的标签
git push <remote-name> :refs/tags/<tag-name>
git push <remote-name> --delete <tag-name>

// 检出标签，此命令会使你的仓库处于分离头指针（detached HEAD）状态
// 在分离头指针状态你进行某些更改然后提交他们，标签不会发生变化，你的提交不属于任何分支，只能使用确切的哈希来访问
git checkout <tag-name>
git co <tag-name>

// 因此，需要修复旧版本的错误，通常需要创建一个新的分支，在新分支上进行修改，并提交，该分支就和此标签不同了
git co -b <branch-name> <tag-name>
```

## Git分支
几乎所有的版本控制系统都支持分支，你可以把工作从开发主线上分离出来，以免影响开发主线
Git处理分支的方式十分轻量，创建和切换分支操作都十分便捷，因此Git鼓励使用分支

进行提交操作时，Git会保存一个提交对象，该提交对象包含一个指向暂存内容快照的指针，还包括作者的姓名，邮箱等信息，以及它的父对象信息（一个或多个）

Git的默认分支是master分支，master分支不是一个特殊的分支，与其他分支没有差别，git init命令会默认创建它

这些分支操作全部都保存在本地，没有与服务器发生交互
```
// 创建分支，会在当前所的提交对象上创建一个指针，Git有一个HEAD的特殊指针，指向当前分支，可以理解为当前分支的别名
git branch <branch-name>
git br <branch-name>

// 切换分支，分支切换会改变工作目录的内容，如果Git不能干净利落的完成这个任务，它将禁止切换分支
git checkout <branch-name>
git co <branch-name>

// 新建分支并切换
git checkout -b <branch-name>
git co -b <branch-name>

// 合并分支，将<branch-name>分支合并到当前分支
// 合并两个分支时，如果顺着一个分支能够到达另外一个分支，那么就进行快进（fast-forward）
// 有分叉的合并会产生一个合并提交，该提交使用三方（两个分支末端的快照和它们的公共祖先）合并的结果
// 如果合并时遇到对同一个文件同一处的修改，则会发生冲突，等你解决冲突后，使用git add命令将冲突的文件标记为已解决冲突
git merge <branch-name>

// 删除分支
git branch -d <branch-name>
git br -d <branch-name>

// 查看分支，*号代表当前所在分支
git branch
git br

// 查看分支最后一个提交
git branch -v
git br -v

// 显示列表中已经合并或尚未合并到当前分支的分支
git branch --merged
git br --no-merged
```

![一个和并提交](/img/一个和并提交.png)

## 远程分支
远程引用是对远程仓库的引用，包括分支标签等。
我们经常利用远程跟踪分支，它是远程分支状态的引用，它们是你不能移动的本地引用，当你进行任何网络操作时，它们会自动移动
远程跟踪分支像是你上次连接到远程仓库时，那些分支所处状态的书签，以<remote>/<branch>命名，如origin/master

origin和master没有特殊含义，origin是git clone创建的远程仓库的默认名字，master是git init时默认的起始分支的名字
```
// 查看完整远程引用列表
git ls-remote
git remote show <remote-name>
```

![本地与远程的工作可以分叉](/img/本地与远程的工作可以分叉.png)

### 推送到远程分支
你想分享一个分支，必须要将其推送到有写入权限的远程仓库，本地的分支不会自动与远程仓库同步，必须显式的推送
```
// 将分支推送到远程仓库
git push <remote-name> <branch-name>
// 本地分支与远程分支采用不同的名字
git push <remote-name> <branch-name>:<remote-branch-name>

// 其他人可以获取到远程分支
git fetch <remote-name>
// 合并远程分支到当前分支
git merge <remote-name>/<branch-name>
// 本地创建新的分支，起点为远程分支，该分支跟踪远程分支
git co -b <branch-name> <remote-name>/<branch-name>
```

### 跟踪分支
从一个远程仓库检出一个本地分支会自动创建一个做跟踪分支（上游分支），跟踪分支是与远程分支有直接关系的本地分支
如果在跟踪分支上输入git pull，则Git能自动识别去哪个服务器抓取、合并到哪个分支
克隆仓库时，会自动创建一个跟踪origin/master的master分支
```
// 本地创建新的分支，起点为远程分支，该分支跟踪远程分支
git checkout -b <branch-name> <remote-name>/<branch-name>
git checkout --track <remote-name>/<branch-name>
// 当前分支设置跟踪的远程分支
git branch -u <remote-name>/<branch-name>
git branch --set-upstream-to <remote-name>/<branch-name>

// 设置好跟踪分支后，可以通过@{u}或@{upstream}快捷方式来引用它
git merge origin/master
git merge @{u}

// 查看分支跟踪的远程分支
git branch -vv

// 获取远程分支，并不会修改本地工作目录中的内容，需要自己合并
git fetch origin master
git merge origin/master

// git pull会查找当前分支的跟踪分支，从服务器抓取数据然后尝试合并入那个远程分支
git pull

// 删除远程分支
git push <remote-name> --delete <branch-name>
```

### 变基
Git中整合来自不同分支的修改主要有两种方法：merge和rebase
合并进行一次三方合并，产生一个新的快照并提交
变基首先找到两个分支的共同祖先，对比当前分支的历次提交，提取相应的修改，然后在新的分支上进行重新播放，最后进行一次快速合并
一般进行变基的目的是确保向远程分支推送时，保持提交历史的整洁
这两种方式整合的最终快照是一样的

```
// 将当前分支变基到master分支
git rebase master
// 将在client分支里但不在server分支里的修改，在master分支里重放
git rebase --onto master server client
// 无需切换分支进行变基
git rebase master server
```

![变基然后进行快进合并](/img/变基然后进行快进合并.png)

![通过合并操作来整合分叉的历史](/img/通过合并操作来整合分叉的历史.png)

变基存在风险，使用变基的准则是不要对在你的仓库外有副本的分支执行变基
变基的操作实际上是丢弃一些现有的提交，然后相应的新建一些内容一样的但是实际上不同的提交
如果别人在该分支上进行了工作，你对此分支进行了变基，那么就会出现问题
出现类似的情况可以使用变基来解决变基
```
git pull --rebase
```

# 总结
以上的命令和知识可以应付大部分Git的使用场景了
如果遇到其他的困难问题，可以后续再补充

# 参考

1) [Pro Git][1]

[1]: https://git-scm.com/book/en/v2