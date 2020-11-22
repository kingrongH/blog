+++
title = "你应该试试这个科学上网工具clash-node"
date = 2019-05-13 15:06:40
[taxonomies]
tags = ["clash", "ssr", "vmess", "node"]
categories = ["JavaScript"]

+++

我觉得使用linux的你应该尝试一下使用这个工具科学上网，自从ssr发展了科学上网节点的订阅形式之后，各大的机场都支持了订阅，由于日渐增高的墙，许许多多的科学上网的工具也都层出不穷，但不是所有工具都支持订阅链接导入的方式获取节点，许多的工具还都只能够手动更改配置运行，尤其是对linux的用户来说。[clash](https://github.com/Dreamacro/clash)是其中一款，非常棒的工具，我正在使用它，但没有linux下的支持订阅的实现。
所以我写了一个工具[clash-node](https://github.com/kingrongH/clash-node)，支持订阅链接。

<!--more-->

## 介绍

这个工具是使用Node.js + typescript 完成了一个clash的包装, 实现订阅链接的一个功能，核心的代理功能还是由[clash](https://github.com/Dreamacro/clash) 来实现的。当然它不仅仅是只有订阅链接的功能，那样的话我们使用的人会无从下手的，我正是从我日常使用的一个角度来考虑如何设计这个功能的。他还可以直接开启和关闭clash的功能，以及正在测试的切换链接的功能。

## 依赖

1. Node.js

最好是10及以上的版本，推荐使用`nvm`来管理和安装nodejs

[nvm](https://github.com/nvm-sh/nvm)

2. npm or cnpm or yarn

你可以使用你想要用的那一个node.js包管理器。
中国用户推荐使用`cnpm`

3. typescript

你需要有`typescript`的支持。
通过

	npm install -g typescript

来完成一个全局的安装，如果你只想要本地安装的话：

	npm install typescript


## 安装

现在来说只能通过`github`进行安装。

```shell
git clone https://github.com/kingrongH/clash-node
cd clash-node
npm install
npm build
```

## 使用

工具本身使用是符合日常用户的使用习惯的。只需要通过终端运行一些命令即可。

### 订阅链接

#### 添加订阅

订阅链接我们需要这么做，在项目文件的根目录下：
```shell
cd bin
node clash-node.js addSub
```
然后接下来根据提示完成订阅链接的添加即可。它看起来像这样：

![订阅链接](https://camo.githubusercontent.com/ee554091b16ceb6073636ffeb8949ce5274a08b7/68747470733a2f2f692e6c6f6c692e6e65742f323031392f30342f32392f356363373030613234386633362e706e67)

这个时候链接的名称和链接地址以及节点信息，已经分别放到对应配置文件中了。

#### 更新订阅

更新已经在配置文件中的订阅同样简单：

```shell
node clash-node.js updateSub
```


### 运行clash

在我们更新订阅节点信息之后，我们就可以直接运行clash作为代理了。运行以下命令：

```shell
node clash-node.js start
```
这个时候应该就可以看到任务管理中有正在运行的clash了，打开[google](www.google.com)试试。

关闭正在运行的clash进程:
```shell
node clash-node.js stop
```

### 配置

配置和[clash](https://github.com/Dreamacro/clash)的配置一样，你可以在`./config/config.yml`中配置除了`Proxy`、`Proxy Group`和`Rule`以外的配置，clash-node在运行的时候会读取这里的配置。

## 感谢

[Dreamacro/clash](https://github.com/Dreamacro/clash)

## 结束

如果你有任何关于这个工具的想法或者建议，可以联系我，或者提request or issue

[github](https://github.com/kingrongH/clash-node)

