---
title: git submodule 测试
date: 2017-03-04 10:17:29
tags:
  - git
  - 工具
categories:
  - 工程文档
---

# 1. 前言

在做 `github pages` 工程多端同步的时候，我们在我们的主工程下面 `clone` 了一个子工程，如 `themes/next`。当使用 `git push origin master` 提交我们的 `blog` 内容修改的时候，可以在远程仓库中 `themes/next` 下的内容为空，没有同步。

网上解释这个现象的关键词 `git submodule`, 官方的解释说明，[点击这里](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

# 2. 测试 git submodule 工作现象

## 2.1 裸仓库和一般仓库的区别

裸仓库和一般的 `git` 仓库有什么区别？主要有下面2点：

1. 裸仓库没有工作区，一般 `git` 仓库有工作区

2. 执行 `git push` 命令，可以推送修改到裸仓库，而一般的仓库不能被推送

说到推送，向一般的 `git` 仓库执行 `git push`，会提示：

```
! [remote rejected] master -> master (branch is currently checked out)
error: failed to push some refs to 'E:/git-submodule-test/main/'
```

看到这里就理解了2者区别。

## 2.2 创建2个裸仓库，并向仓库推送修改

执行命令：

```bash
$ mkdir main
$ cd main
$ git init
$ cd ../
$ git clone --bare main main.git

$ mkdir sub
$ cd sub
$ git init
$ cd ../
$ git clone --bare sub sub.git
```

至此，我们成功建立了2个裸仓库：`main.git` 和 `sub.git`。

```bash
$ rm -rf main
$ rm -rf sub
$ git clone main.git main
$ git clone sub.git sub
```

`clone` 2个一般仓库，并添加内容，提交到对应的裸仓库中。

## 2.3 从主仓库开始

我们现在有2个仓库，下面开始我们一般的工作流程：

1. `clone` 一个仓库代码到本地

2. 在这个仓库下面，再 `clone` 一个仓库代码

对应的执行代码：

```bash
$ git clone main.git ztest1
$ cd ztest1
$ git submodule add ../sub.git sub
```

这时我们可以看到 `ztest1` 目录下多了 `.gitmodules` 文件，多了 `sub` 文件夹。执行 `git status`，显示内容：

```bash
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   sub

```

提交修改到 `main.git`。

```
$ git commit -m "first post"
[master 3fb2def] first post
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 sub

$ git push origin master
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 353 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To E:/git-submodule-test/main.git/
   607e63d..3fb2def  master -> master

```

我们看看效果。

```bash
$ git clone main.git ztest2
```

这里可以看到 `main.git` 中的内容回来了，但是 `sub` 下面没有内容。这里我们要取回这个内容。

```
$ git submodule init
Submodule 'sub' (E:/git-submodule-test/sub.git) registered for path 'sub'

$ git submodule update
Cloning into 'E:/git-submodule-test/ztest2/sub'...
done.
Submodule path 'sub': checked out '9f2f93760733ede5a1e30ccd4b6e0450fd60e221'

```

到这里，我们的 `sub` 内容返回了。

## 2.4 操作子仓库

上面的步骤中，我们知道怎么获取2个仓库的内容。假设下下面的场景:

1. 我自己在本地修改了 `sub` 中的内容，如何推送到服务端，这样别人可以更新

2. 别人已经推送到服务端远程仓库的修改，我在这边如何同步

对于场景1，我们可以这样模拟，在 `ztest1` 中的 `sub` 下修改内容。我们在 `ztest1` 路径下执行下面操作：

```bash
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   sub (modified content)

$ git add .
$ git commit -m "change sub"
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
        modified:   sub (modified content)

no changes added to commit

```

可以看到 `ztest1` 这个仓库操作不了 `sub` 下的内容，我们进入 `sub` 路径下执行下面操作：

```bash
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   helloworld.md

no changes added to commit (use "git add" and/or "git commit -a")

$ git add .
$ git commit -m "change sub"
[master bf7521b] change sub
 1 file changed, 3 insertions(+), 1 deletion(-)

$ git push origin master
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 293 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To E:/git-submodule-test/sub.git
   9f2f937..bf7521b  master -> master

```

这里我们把修改，推送到远程仓库了。下面我们在 `ztest2` 中来同步这个修改，到 `ztest2/sub` 目录下，执行操作：

```bash
$ git pull origin master
From E:/git-submodule-test/sub
 * branch            master     -> FETCH_HEAD
Updating 9f2f937..bf7521b
Fast-forward
 helloworld.md | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)

```

这里看到同步了。

# 3. 结果说明

使用 `git submodule` 虽然管理了多个子仓库，但是每个子仓库的操作都是各自操作。唯一和一般的仓库使用区别是，在 `git clone` 主仓库后，我们需要执行：

```bash
$ git submodule init
$ git submodule update
```
操作来更新我们子项目代码内容，其他的没有什么区别。

---
2017-04-07

具有 `submodule` 的仓库，当我们 `clone` 主仓库后，执行

```
> git submodule init
> git submodule update
```

操作后，这里会自动同步子仓库内容，当然不是同步远程的子仓库内容。这个说法是什么意思呢？

我们操作子仓库，然后经过 `git commit` 后会生成一个版本号，这个版本号我们需要在主仓库中也要 `git commit` 一次。这个时候，主仓库会记录子仓库中他需要的版本号。所以执行 `git submodule update` 操作，这个时候其实做的操作是拿着主仓库中记录的子仓库的版本号，去把对应的子仓库版本下载下来。

怎么理解？

# 4. 再总结

## 4.1 git submodule update 报错

我们执行 `git clone` 把主仓库克隆下来，然后按照教程顺序执行 `git submodule init` 和 `git submodule update` 操作，这个时候，可能会提示 `ref` 和子版本的引用不一致，导致下载子仓库失败。为什么？？？

> 我在子仓库中的修改，没有 push 到远程仓库中，导致在另一个机器上克隆时，会发现远程子仓库中的版本和主仓库中记录的版本，不一致，准确的说是不存在。所以那个版本的内容没有，导致下载失败。 

## 4.2 测试示例

```
kcb@kcb-PC MINGW64 /e
$ mkdir git-submodule-test

kcb@kcb-PC MINGW64 /e
$ cd git-submodule-test/

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ mkdir main

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd main

kcb@kcb-PC MINGW64 /e/git-submodule-test/main
$ git init
Initialized empty Git repository in E:/git-submodule-test/main/.git/

kcb@kcb-PC MINGW64 /e/git-submodule-test/main (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ mkdir sub

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd sub

kcb@kcb-PC MINGW64 /e/git-submodule-test/sub
$ git init
Initialized empty Git repository in E:/git-submodule-test/sub/.git/

kcb@kcb-PC MINGW64 /e/git-submodule-test/sub (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ git clone --bare main/ main.git
Cloning into bare repository 'main.git'...
warning: You appear to have cloned an empty repository.
done.

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ git clone --bare sub/ sub.git
Cloning into bare repository 'sub.git'...
warning: You appear to have cloned an empty repository.
done.

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd main

kcb@kcb-PC MINGW64 /e/git-submodule-test/main (master)
$ git remote -v

kcb@kcb-PC MINGW64 /e/git-submodule-test/main (master)
$ git remote add origin ../main.git

kcb@kcb-PC MINGW64 /e/git-submodule-test/main (master)
$ git remote -v
origin  ../main.git (fetch)
origin  ../main.git (push)

```

至此，我们建立了 2 个裸仓库，并且各自有一个仓库来提交修改。我们在 main 和 sub 中分别添加内容，然后提交到本地仓库同时推送到远程仓库中，这里是 main.git 和 sub.git。

建立第一个带 sub 的实例。

```
kcb@kcb-PC MINGW64 /e/git-submodule-test
$ git clone main.git/ ztest1
Cloning into 'ztest1'...
done.

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd ztest1/

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git submodule add ../sub.git sub
Cloning into 'E:/git-submodule-test/ztest1/sub'...
done.
warning: LF will be replaced by CRLF in .gitmodules.
The file will have its original line endings in your working directory.

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   .gitmodules
        new file:   sub


kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git commit -m "ztest1 添加 sub"
[master f416ded] ztest1 添加 sub
 2 files changed, 4 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 sub

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
nothing to commit, working tree clean

```

至此，ztest1 中的文件结构

```
/ztest1
  .gitmodules
  main.txt
  /sub
    sub.txt
```

2 个仓库的内容都克隆下来了。我们在 ztest1 中修改 main.txt 和 sub.txt 内容。

```
kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   main.txt
        modified:   sub (modified content)

no changes added to commit (use "git add" and/or "git commit -a")

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   main.txt

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   sub (modified content)

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git commit -m "ztest1 main.txt change"
[master 7ece350] ztest1 main.txt change
 1 file changed, 3 insertions(+), 1 deletion(-)

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   sub (modified content)

no changes added to commit (use "git add" and/or "git commit -a")

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ cd sub/

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ git add .

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ git status
On branch master
Your branch is up-to-date with 'origin/master'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   sub.txt


kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ git commit -m "ztest1 sub.txt change"
[master 22d4842] ztest1 sub.txt change
 1 file changed, 3 insertions(+), 1 deletion(-)

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   sub (new commits)

no changes added to commit (use "git add" and/or "git commit -a")

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git add .

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   sub


kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git commit -m "ztest1 main submit sub"
[master ca2a14b] ztest1 main submit sub
 1 file changed, 1 insertion(+), 1 deletion(-)

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ git push origin master
Counting objects: 8, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (7/7), done.
Writing objects: 100% (8/8), 917 bytes | 0 bytes/s, done.
Total 8 (delta 1), reused 0 (delta 0)
To E:/git-submodule-test/main.git/
   ef7567f..ca2a14b  master -> master


```

这里我们只是在 ztest1 下把内容推送到 mian.git 中了。我们这里执行下面操作

```
kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ git clone main.git ztest2
Cloning into 'ztest2'...
done.

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd ztest2

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest2 (master)
$ git submodule init
Submodule 'sub' (E:/git-submodule-test/sub.git) registered for path 'sub'

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest2 (master)
$ git submodule update
Cloning into 'E:/git-submodule-test/ztest2/sub'...
done.
Fetched in submodule path 'sub', but it did not contain 22d484273b4dca3fe105251fde9885af436d73ac. Direct fetching of that commit failed.

```

> 这里看到的提示信息是什么，sub 中不包含版本 `22d484273b4dca3fe105251fde9885af436d73ac`。失败了，这个是因为我们没有把我们的 sub 修改推送到 sub 中。

```
kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest2 (master)
$ cd ../ztest1/sub/

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ git push origin master
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 320 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To E:/git-submodule-test/sub.git
   d3386e7..22d4842  master -> master

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1/sub (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest1 (master)
$ cd ../

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ git clone main.git ztest3
Cloning into 'ztest3'...
done.

kcb@kcb-PC MINGW64 /e/git-submodule-test
$ cd ztest3

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest3 (master)
$ git submodule update

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest3 (master)
$ git submodule init
Submodule 'sub' (E:/git-submodule-test/sub.git) registered for path 'sub'

kcb@kcb-PC MINGW64 /e/git-submodule-test/ztest3 (master)
$ git submodule update
Cloning into 'E:/git-submodule-test/ztest3/sub'...
done.
Submodule path 'sub': checked out '22d484273b4dca3fe105251fde9885af436d73ac'

```

这里就可以很明白的看清楚了，主仓库中记录子仓库的版本，我们在执行 git submodule update 时，拉取的对应子仓库版本中的版本号。至此 git submodule 的表现情形测试清楚了。

1. 各个版本各自更新

2. 主仓库会把子仓库的修改也要 commit 一遍，主要是版本号信息

{% qnimg cl.jpg title:cl alt:waiting  %}