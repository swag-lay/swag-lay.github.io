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

将(name)分支合并到当前所在分支，此处通常当分支/测试分支有了独立内容，希望合并的时候，也就是测试正确的时候合并到当前分支。

合并完后一般删除分支