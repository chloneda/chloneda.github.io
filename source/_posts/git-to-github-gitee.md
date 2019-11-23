---
title: git同步代码至github和gitee(码云)
tags: Git
categories: Git
keywords: Git
comments: false
---
**注：本文出自博主 Chloneda**：[个人博客](https://chloneda.github.io/) | [博客园](https://www.cnblogs.com/chloneda) | [Github](https://github.com/chloneda) | [Gitee](https://gitee.com/chloneda) | [知乎](https://www.zhihu.com/people/chl_vip/)

我们有时候开发代码需要把代码同步到多个远程库中，如何操作才能做到呢？

我们知道，git是分布式版本控制系统，同步到多个远程库时，需要用不同的名称来标识不同的远程库，而git给远程库起的默认名称是origin。所以我们需要修改、配置名称，以关联不同远程库。有两种方式！

为了方便举例，我以GitHub和Gitee(码云)作为示例！

# 同步方式

## 命令方式同步
先删除已关联的名为origin的远程库：
```
git remote rm origin
```

然后，再关联GitHub的远程库：
```
git remote add github git@github.com:chloneda/demo.git
```

接着，再关联码云的远程库：
```
git remote add gitee git@gitee.com:chloneda/demo.git
```

## 配置方式同步

修改.git文件夹内的config文件：
```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "origin"]
	url = git@github.com:chloneda/demo.git
	fetch = +refs/heads/*:refs/remotes/github/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```
将上述文件内容[remote "origin"]内容复制，修改origin名称，内容如下：
```
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
[remote "github"]
	url = git@github.com:chloneda/demo.git
	fetch = +refs/heads/*:refs/remotes/github/*
[remote "gitee"]
	url = git@gitee.com:chloneda/demo.git
	fetch = +refs/heads/*:refs/remotes/gitee/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
```

# 查看远程库

通过以上两种方式的任一种方式配置完成后，我们用git remote -v查看远程库信息：
```
gitee   git@gitee.com:chloneda/demo.git (fetch)
gitee   git@gitee.com:chloneda/demo.git (push)
github  git@github.com:chloneda/demo.git (fetch)
github  git@github.com:chloneda/demo.git (push)
```
可以看到两个远程库，说明配置生效了。

# 上传代码
```
git add .
git commit -m "update"
```

# 提交到github
```
git push github master
```

# 提交到码云
```
git push gitee master
```

# 更新代码
```
# 从github拉取更新
git pull github

# 从gitee拉取更新
git pull gitee
```

# 踩到的坑
上述过程中，更新或提交代码时可能会遇到**fatal:refusing to merge unrelated histories** (拒绝合并无关的历史) 错误，解决办法：

首先将远程仓库和本地仓库关联起来。

```
git branch --set-upstream-to=origin/remote_branch  your_branch
```
其中，origin/remote_branch是你本地分支对应的远程分支，your_branch是你当前的本地分支。


然后使用git pull整合远程仓库和本地仓库。
```
git pull --allow-unrelated-histories    (忽略版本不同造成的影响)
```
重新更新、提交即可。

注： 如遇到 **Git没有共同祖先的两个分支合并** 的情形请自行查询！




---