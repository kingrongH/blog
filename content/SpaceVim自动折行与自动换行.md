+++
title = "SpaceVim自动折行与自动换行"
date = 2018-08-17 20:03:17
[taxonomies]
tags = ["vim", "SpaceVim"]
categories = ["Problems"]

+++

## 关于

使用SpaceVim写Markdown总是一段一行写到头，让人很生气。而作为半吊子vim使用者脑子里完全没有关键词，不知道如何去搜索解决这个问题，于是搜了半天的自动换行，完全不得要领。在经过大量的查找之后终于让我找到了，自动换行与自动折行的区别。原来我想要达到的目的叫做自动折行。于是乎记录一下今天的愚蠢。。。

## 自动换行与自动折行的区别

* 自动换行是当你写满多少字数之后会自动加一个回车，然后从下一行开始。

* 自动折行是当你写满窗口的一行之后自动从下一行开始，没有加回车，并且是同属一行的，如图：

![自动折行](https://i.loli.net/2018/08/17/5b76bfda92156.png "自动折行")

vim里面设置自动换行是在配置文件中设置`set textwidth = 80` 

设置自动折行则是`set wrap` ，取消自动折行`set nowrap` 

## 解决方法

需要SpaceVim在编辑markdown文件时自动折行，步骤如下：

1.在 `~/.SpaceVim.d/init.toml`  [options] 下添加：
    
    bootstrap_before = "myspacevim#before"


2.然后新建:

`~/.SpaceVim.d/autoload/myspacevim.vim`

将下面这行添加到其中即可
    
    au BufNewFile,BufFilePre,BufRead *.md set filetype=markdown wrap


## 不足

设置了自动折行之后，就无法在插入模式愉悦地通过上下方向键跳到下一行，因为是同属一行的，所以只能通过左右方向键跳到下一行。

