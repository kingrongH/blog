+++
title = "nvim:Error while reading ShaDa file there is an item at position 270498 that must not be there Missing itemsare for internal uses only"
date = 2018-09-04 16:17:01
[taxonomies]
tags = ["vim", "SpaceVim", "vimfile"]
categories = ["Problems"]

+++

# 问题描述


今天打开`SpaceVim` 的时候提示错误：

    Error while reading ShaDa file: there is an item at position 270498 that must not be there: Missing itemsare for internal uses only

并且发现只要一旦打开了vimfile这个插件问题就开始提示。继续按下 `Enter` 会开始提示vim配置上具体哪一行的错误。



# 解决问题

`nvim` github上有相似的问题: [#6875](https://github.com/neovim/neovim/issues/6875 "vimfile错误") ，实际上配置上并没有什么错误。

按照git上所说我们在终端运行 `nvim -u NONE -V`得到输出信息：

    $ nvim -u NONE -V
    Reading ShaDa file "/Users/kevin/.local/share/nvim/shada/main.shada" info marks oldfiles
    E576: Error while reading ShaDa file: there is an item at position 270498 that must not be there: Missing items are for internal uses only
    Press ENTER or type command to continue

我们得到的输出信息也许不同不过我们只要移除 `.local/nvim/shada/main.shada` ，完美工作。

