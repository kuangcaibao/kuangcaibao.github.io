---
title: 创建自己的 github.io 站点页面
date: 2017-02-28 20:05:43
tags:
  - 站点
categories:
  工程文档
---

# 为什么使用 github.io 提供的静态页面服务写 blog


  三方网站（如 oschina.net, segmentfault.com, csdn.net 等）提供的 `blog` 系统，虽然可以记录 `blog`。因为托管在第
三方，经常容易忘记，然后就没有然后了。`github.com` 是一个免费的代码托管平台，可以托管你的代码和平常的记录，支持
`markdown` 语法记录，非常方便。 但是在 `github.com` 上查看自己的 `markdown` 文件内容，虽然渲染成了页面，显示也很
友好，但是没有归档功能，查找其他不方便。

`github pages` 提供的是一个静态页面服务，`github.com` 上的每个开发者都有一个域名来使用它的静态网页服务，地址类似
`username.github.io`。免费域名，免费提供站点服务，非常方便，同时自己管理自己的内容。

# 使用 `hexo` 来创建一个 `github pages`

`github pages` 官方推荐的主题是 `ruby` 实现的，使用起来看不懂。这里推荐一个 `hexo`，一个基于 `nodejs` 的工具。
使用起来还是比较方便的。

这里假设你完成了一下的准备：

1. 系统安装了 `node`, `git`

2. 在 `github.com` 上创建了一个仓库 `username.github.io.git`

3. 在你的系统和 `github` 间建立了一个 `ssh` 通路

这时按照 `hexo` 的教程，来创建。

``` bash
> npm install -g hexo-cli
> hexo init blog-test
> cd blog-test
> hexo server
```

执行完这些命令后，你在本地启动了 `hexo server` 服务，可以在浏览器中输入地址查看内容。通过修改配置