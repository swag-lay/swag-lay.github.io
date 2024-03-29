---
title: git-learning
date: 2023-05-10 18:14:02
categories:
- 工具
- git
tags:
- git
---

# git命令学习

初始化文件夹

```shell
git init
```



添加新文件

```shell
git add .
git add filename
git add -am
```

`git commit -am` 的作用是将所有已经被 Git 管理的文件添加到暂存区并提交这些变更，并且可以通过 `-m` 参数指定提交消息。需要注意的是，如果有新文件或未被 Git 管理的文件需要添加到仓库中，还需要使用 `git add` 命令将它们添加到暂存区中。



提交版本

```shell
git commit -m "说明"
```



在本地的git仓库添加远程仓库

```shell
git remote add origin [url]
```

发布版本

```shell
git push origin master
git push --set-upstream origin [branch]
```

origin其实就是远程仓库地址，master就是分支。在.git文件夹中的config都可以修改。



拉去到本地仓库

- 默认合并

  ```shell
  git pull
  ```

- 非默认

  ```shell
  git pull [url]
  ```

-  拉取所有分支

  ```shell
  git fetch
  ```

  

创建分支

```shell
git branch (name)
```



切换分支

```shell
git checkout (name)
```



创建分支并切换到该分支

```shell
git checkout -b (name)
```



删除分支

```shell
git branch -d (name)
```



合并分支

```shell
git merge (name)
```

将(name)分支合并到当前所在分支，此处通常当分支/测试分支有了独立内容，希望合并的时候，也就是测试正确的时候合并到当前分支。合并完后一般删除分支



撤回到某个版本（在还没push之前）

```shell
git reset HEAD
```



合并到上一次commit中

```shell
git add .
git commit --amend
```



## git error

- ssh: connect to host github.com port 22: Connection timed out fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.

  解决方法：

  1. 修改.git文件中的config文件将origin的url换为http地址

  2. 修改端口号

     ```shell
     cd ~/.ssh
     vim congfig
     ```

  


## git pr

fork开源项目

git clone 本地fork仓库

与源项目上游建立连接

git remote add upstream ...

同步最新代码

git fetch upstream 分支名

