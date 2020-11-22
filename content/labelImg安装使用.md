+++
title = "labelImg安装使用"
date = 2018-09-24 21:58:09
[taxonomies]
tags = ["python", "tensorflow"]
categories = ["Learn"]

+++

# why

因为我们在学习和使用tensorflow或者其他机器学习的框架的时候，我们最终还是会需要把自己配置的数据使用到我们的程序中，这个时候我们需要一个能够帮助自己为图片贴标签的工具。labelImg正是这样一个简易好用的工具。

[labelImg](https://github.com/tzutalin/labelImg "labelImg")

![labelImg](https://i.loli.net/2018/09/24/5ba8ef101801f.png "labelImg")

# 安装

我们安装的前提是我们的电脑是必须有pyhton、qt4或者qt5、lxml。

    git clone https://github.com/tzutalin/labelImg.git

然后进入到`labelImg`目录下

    make qt5py3 #如果是pyqt4则 `make qt4py2`

最终执行

    python3 labelImg.py

我们就来到了我们可以操作的图形界面了，然后可以自由的贴标签了。

# 使用

该工具本来就是设计了图形化的界面，所以对于它的使用也十分简单明了的。

前提：

* 一个很多图片的文件夹
* 一个和上一个文件夹并行的文件夹，用来存储标签

使用

## 步骤(PascalVOC)

1. 点击`Change Save Dir` 将默认的标签存储文件改变至之前已经建好的标签文件夹，如图

![Screenshot_20180927_175430.png](https://i.loli.net/2018/09/27/5baca9550cf3c.png)

2. `Open Dir`打开你将要贴标签的图片的目录，选择完毕后会自动加载第一张图片。

3. 右键图片选择`CreateRectBox`或者在菜单中选择`CreateRectBox`,选择出你所需要贴标签的部分。`Ctrl+S`或者点击`Save`将会自动将标签信息保存到刚刚设置到的目录下。

![Screenshot_20180927_180617.png](https://i.loli.net/2018/09/27/5bacabb516d5c.png)

## 步骤(YOLO)

1. 通过点击左边菜单里的`PascalVOC`转换为`YOLO`格式

2. 在你的创建的`label`文件夹下编辑`class.text`，定义你将要贴的所有标签。

![Screenshot_20180927_181929.png](https://i.loli.net/2018/09/27/5bacaed592f66.png)

3. 按照上一个步骤，选框和贴贴标签。

因为贴标签过程中需要一张一张执行，所以快速的方法是合理利用快捷键和`default label`,祝你使用愉快！


