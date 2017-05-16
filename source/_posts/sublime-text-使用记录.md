---
title: sublime-text 使用记录
date: 2017-04-11 00:08:56
tags:
	tools
categories:
	tools
---

> 介绍使用 sublime text 的过程中个人认为不错的技巧

**`sublime-text3` 中，程序默认装了一些 packages，放在安装目录下的 `packages` 下。然后我们后续安装的包，会放在 `Preferences -> Browse Packages...` 打开的目录下。**

**这里的包一般都是 `[包名].sublime-package`，这里可以用压缩文件打开，会发现里面有配置文件。我们可以修改那里的配置文件，来达到我们配置包的目的。**

# 1. 安装 packages

下载的 `sublime text3` 默认是没有 `install package` 操作，安装这个功能。`` Ctrl + ` `` 打开控制台，然后输入：

```
import urllib.request,os,hashlib; h = 'df21e130d211cfc94d9b0905775a7c0f' + '1e3d39e33b79698005270310898eea76'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
```

## 1.1 tab 占位大小设置

`tab` 大小默认为 4 个空格，这里我习惯设置 `tab` 为 2 个空格大小。`preferences -> settings` 打开文件 `Preferences.sublime-settings User` 并添加内容

```
{
  "tab_size": 2,
  "translate_tabs_to_spaces": true,
}
```

[具体说明](https://packagecontrol.io/installation)

# 2. 快捷键

``Ctrl + ` `` 呼出控制台

`Ctrl + shift + P` 调出 `package control` 命令

`Ctrl + P` 输入文件名，打开改文件

# 3. 不错的 packages

这里介绍经常用到的一些 sublime 插件，安装步骤和使用说明  [点这里](https://packagecontrol.io/packages)

1. `ctrl + shift + p` 进入 package control 控制输入框

2. 输入你要安装的 `package` 名称，等待安装完成

3. 配置 `package`

## 3.1 OmniMarkupPreviewer

markdown 预览插件。 

``ctrl + alt + o`` 在浏览器中预览 md 文件，

``ctrl + alt + x`` 生成 html 文件。

## 3.2 Nodejs

这里我们执行 `ctrl + b` 命令，发现命令窗口中显示的是乱码。怎么修改？

网上的做法是修改 `Nodejs.sublime-build` 文件，这个文件在哪？ 找了一圈都没有看到，并且从 `sublime` 的 `Preferences -> Package Settings -> Nodejs -> Settings User` 来打开配置文件，发现打开的是 `Nodejs.sublime-settings` 文件，这里没有 `encoding` 配置。

`Preferences -> Browse Packages ...` 打开我们安装包的路径，目录截图如下：

{% qnimg sublime-packages-dir.png title:"sublime 安装包路径截图" alt:sublime-package-dir  %}

这里看到没有我们安装的 Nodejs，找不到。上一级目录下有个 `Installed Packages` 目录中会发现 `Nodejs.sublime-package` 文件，我们解压这个文件，会发现有文件 `Nodejs.sublime-build`，我们修改其中的内容，把 `encoding` 值改为 `utf8`。重启 `sublime text`，会发现我们的编码问题解决。

网上还有需要改 `cmd` 这些配置，个人猜想可能是 `path` 环境路径中没有加入 `nodejs` 的安装目录，导致 shell 执行 `node $file` 会报错，但是这里我们安装 `nodejs` 的时候，这些环境都安装了，所以可以执行了。

## 3.3 运行 Java

我们在 `sublime-text3` 中看到了 `build` 项中有 `java`。我们写一个 `Test.java` 文件，然后执行 `ctrl + b` 命令，可以看到在 `java` 文件对应的目录下生成了一个 `class` 文件，但是这里我们缺少输出。

修改 `C:\Program Files\Sublime Text 3\Packages\Java.sublime-package` 中的 `JavaC.sublime-build` 文件。内容如下

```
{
  - "shell_cmd": "javac \"$file\"",
  + "shell_cmd": "runJava.bat \"$file\"",
  "file_regex": "^(...*?):([0-9]*):?([0-9]*)",
  "selector": "source.java"
}
```

> 代码中前面的 `-` 表示删除那一行，`+` 表示添加那一行

```
// runJava.bat

@ECHO OFF
cd %~dp1
IF EXIST %~n1.class (
DEL %~n1.class
)
javac -encoding UTF-8 %~nx1
IF EXIST %~n1.class (
java %~n1
)

```


[具体做法参考](http://www.jianshu.com/p/58bf9e4d5b32#)

# 4. 附录：一些问题

## 4.1 package install 报错 

报错信息为

```
Package Control: Error downloading channel. URL error unknown error (_ssl.c:2228) downloading https://packagecontrol.io/channel_v3.json
```

fixed: 

`preferences -> Browse Packages... ` 进入 `User` 目录，删除目录下的 `Package Control.merged-ca-bundle` 和 `Package Control.user-ca-bundle` 文件。[具体说明](https://github.com/wbond/package_control/issues/957)