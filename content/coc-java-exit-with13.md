+++
title = "coc-java：java exit with code 13"
date = 2019-04-03 19:56:55
[taxonomies]
tags = ["vim", "coc", "java"]
categories = ["Problems"]

+++

这只是一篇简单的记录

今天在安装coc-java这个插件之后，遇到一个问题：`...java exit with code 13`

## 原因分析

因为`coc-java`在你打开一个java文件之后发现没有`jdt`会自动下载[jdt.ls](https://github.com/eclipse/eclipse.jdt.ls)，但是可能处于种种原因你在下载途中退出了，导致下载的的不完整，所以你以打开java文件就报错误。

这一点我觉得coc应当多做一些提示，避免我们下载中途退出，或者在网站上提供一下用户自己下载的方式。

## 解决问题

你只需要删了`~/.config/coc/extensions/coc-java-data/`下的所有文件，然后打开一个java文件，它提示下载，静静等待它下载完就好了。

或者在官网[eclipse jdt download](https://download.eclipse.org/jdtls/snapshots/?d)那一串文件中选择最新的那个下载然后解压内容到`~/.config/coc/extensions/coc-java-data/server/`中。
