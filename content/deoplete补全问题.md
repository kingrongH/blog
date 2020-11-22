+++
title = "deoplete补全问题:restarting server because it's taking too long"
date = 2018-08-21 17:57:23
[taxonomies]
tags = ["vim", "SpaceVim", "deoplete"]
categories = ["Problems"]

+++

## 问题描述

用vim写python的时候遇到一个问题：restarting server because it's taking too long,如图所示：

![too long](https://i.loli.net/2018/08/21/5b7be3a0c975d.png "Restarting")

并且补全没有加载出来。

这个问题一般只出现在我们需要调用比较大的pyhton包的时候，比如tensorflow、numpy和pandas等。

## 解决方法

deoplete的github上有相似的issue：[#167](https://github.com/zchee/deoplete-jedi/issues/167)
因为deoplete在补全之前会将我们import的包缓存起来

 `~/.cache/deoplete/jedi/3.6/`

deoplete默认的加载时间是10s,对于有些大包的加载来说是不够的，所以我们要做的便是增加加载的时间。

**解决:**

在这里我使用的是SpaceVim ，我只需要在我的 `myspacevim.vim`的配置下增加以下代码：

    let g:deoplete#sources#jedi#server_timeout = 25

你也可以在你自己的`vimrc`中增加，其中默认的加载时间是`10s`，我将之调整到了`25s`，因为经过我的不断测试对于我加载tensorflow来说，这个时间是刚刚好了。

如果你想要多次测试缓存的时间，你可以在每次测试之后将 `~/.cache/deoplete/jedi/3.6`  里的缓存文件删除。
