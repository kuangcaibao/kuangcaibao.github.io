---
title: 创建自己的 github.io 站点页面
date: 2017-02-28 20:05:43
tags:
  - 站点
categories:
  工程文档
---

# 1. 为什么使用 github.io 提供的静态页面服务写 blog

  三方网站（如 oschina.net, segmentfault.com, csdn.net 等）提供的 `blog` 系统，虽然可以记录 `blog`。因为托管在第三方，经常容易忘记，然后就没有然后了。`github.com` 是一个免费的代码托管平台，可以托管你的代码和平常的记录，支持`markdown` 语法记录，非常方便。 但是在 `github.com` 上查看自己的 `markdown` 文件内容，虽然渲染成了页面，显示也很友好，但是没有归档功能，查找其他不方便。

`github pages` 提供的是一个静态页面服务，`github.com` 上的每个开发者都有一个域名来使用它的静态网页服务，地址类似 `username.github.io`。免费域名，免费提供站点服务，非常方便，同时自己管理自己的内容。

# 2. 使用 `hexo` 来创建一个 `github pages`

`github pages` 官方推荐的主题是 `ruby` 实现的，使用起来看不懂。这里推荐一个 `hexo`，一个基于 `nodejs` 的工具。 使用起来还是比较方便的。

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

执行完这些命令后，你在本地启动了 `hexo server` 服务，可以在浏览器中输入地址查看内容。通过修改配置来设置你的站点的一些配置[参考官方文档](https://hexo.io/zh-cn/docs/)。这个配置最好和后面的主题一起看。

在浏览器中输入本机测试地址，看到有个 `hello world` 的例子，到这里。你的 `hexo` 站点在本地可以正常运行了。下一步我们需要做什么？

`github pages` 是一个静态页面服务器，理解起来就是它提供的内容就是 `html` + `css` + `js` 这样的形式组织的页面。所以下一步，我们需要生成静态页面，然后发布到我们 `github.com` 对应的仓库中，执行下面的命令：

```bash
> hexo generate
> hexo deploy
```

这里在 `deploy` 的过程中，我们需要配置我们 `git` 仓库的地址。配置文件 `_config.yml`

```

deploy:
  type: git 
  repo: git@github.com:kuangcaibao/kuangcaibao.github.io.git

```

官方文档，推荐的是 `https` 这种协议的，使用这种协议 `git push` 的时候，会提醒用户名不存在，报错。

我们换成 `git` 协议，也就是地址类似上方那种协议。但是有时也会报错，说没有权限操作远程仓库，这种情形也是昨天遇到的。查询下资料，试了换协议，重新生成 `ssh key` 等。最后发现是我的 `git for windows` 版本低了，有这个 `bug`，新的 `git for windows` 版本没有这个问题，果断升级后成功 `deploy` 了。

到这里，可以在浏览器输入 `username.github.io`，来访问你的页面。如果有内容，到此配置都没有问题了。

# 3. 将 `hexo` 的主题换为 `next`

`google` 一下 `hexo` 流行的主题，第一条推荐是 `hexo-theme-next`，就是用这个主题了。

`hexo` 主题存放路径 `/themes/[theme type]`，配置为文件 `_config.yml` 中的 `theme: [type]`。

在这里，我们 `git clone` 主题 `hexo-theme-next` 的内容到对应的文件夹下。

```bash
> git clone https://github.com/iissnan/hexo-theme-next.git themes/next
```

修改配置文件 `_config.yml` 中内容 `theme: next`

这个时候，重启服务器 `hexo server`，来查看效果。

`hexo-theme-next` 的配置，参考其[官方文档](http://theme-next.iissnan.com/getting-started.html)

# 4. 多端同步我们的内容

前面介绍了几个概念：

1. `github pages` 是一个静态网页服务器，所以对应我们仓库中的文件内容都是一些静态 `html` `css` 文件。

2. `hexo` 是使用模板来生成 `html` 页面，就是说，我们的内容其实是 `md` 格式的文件。

这样，使用命令 `hexo deploy` 只是将生成的静态页面文件 `push` 到了我们的仓库中。实际的源文件，没有处理。

所以这里，我们需要把我们的源文件，就是工程文件也上传到 `github` 中。理解了这个过程，那么我们就好管理了，不管是再新建一个仓库，来存放这些文件。还是在 `github pages` 的那个仓库下建一个分支来存放文件，都行。

# 5. hexo-theme-next 继承第三方服务

## 5.1 评论系统

这里选择 [官方文档](http://theme-next.iissnan.com/third-party-services.html#comment-system) 中推荐的 [多说](http://duoshuo.com/)。然后按照文档说明，注册信息和在站点配置文件中配置对应的内容就可以了。

---

最近多说评论系统要被关闭了，现在把评论系统切换到 `Disqus` 上。切换步骤：

1. 在 [Disqus 官网](https://disqus.com/) 上注册一个账号

2. 然后进入个人设置中心，选择 `add disqus to site` 进入 `install disqus` 页面

3. 选择选项 `i want to install disqus on my site`

4. 按照提示创建

5. 创建成功后，然后根据 [hexo-theme-next](http://theme-next.iissnan.com/third-party-services.html#comment-system) 介绍配置，在 `主题配置文件` 中把这个打开。

到此，评论系统配置成功。不过不能微信，QQ 登录，这个有点伤。

## 5.2 图片存储服务

如果把 `blog` 中图片资源也放到 `github` 中，很容易把 `github` 的免费空间耗完。所以这里把文件上传到第三方云存储上去，直接引用链接。

在 [七牛](http://www.qiniu.com/) 上注册账号，开始操作，操作方式自行 `google`。

这种方式的弊端，就是使用图片前，要把图片上传到七牛空间，然后获取到资源链接，在 `markdown` 文本中使用 `![]()` 这种方式引用。这种过程都是手动完成，繁琐不方便，有没有好的方法???

[点这里!](http://www.jianshu.com/p/c2ba9533088a)

[再点击这里!!!](https://www.npmjs.com/package/hexo-qiniu-sync)

使用第2种方式，虽然配置有些复杂，但是要比方式1方便很多，好处自己体会！！！

使用方式2有个坑，试试吧。解释[点击这里](https://github.com/gyk001/hexo-qiniu-sync/issues/41)


# 6. Q & A

## 6.1 next 主题修改配置的同步问题

在使用 `hexo` 的时候引入 `next` 主题，这个是从另外一个项目中 `clone` 过来的。我们修改主题配置文件的时候，在根目录下执行 `git status` 会检查到 `themes/next` 下有变化，但是一套 `git` 操作下来，会发现我们的远程仓库中 `themes/next` 下的文件夹是空的，没有同步过去。

我们手动来添加我们修改的文件。

```bash
> git add themes/next/_config.yml
fatal: Pathspec 'themes/next/_config.yml' is in submodule 'themes/next'
```

看看 `git` 中 `submodule` 是个概念，[解释](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)

具体的 `git submodule` 解释，[点击这里](https://kuangcaibao.github.io/2017/03/04/git-submodule-%E6%B5%8B%E8%AF%95/#more)

{% qnimg cl.jpg title:cl alt:waiting  %}