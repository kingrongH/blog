+++
title = "树莓派(raspberry pi) ssh RefuseConnection"
date = 2018-09-29 14:37:36
[taxonomies]
tags = ["树莓派", "ssh"]
categories = ["Problems"]

+++

# 问题描述

通过ssh连接树莓派的时候，出现如下问题：

    ssh: connect to host xxx.xxx.xx.x port 22: connection refused.

这个时候意味着你的ip是正确的，但是树莓派系统中的`ssh`却并没有开启。


# 解决办法

这个时候我们只需要将内存卡拔下了，插入电脑中，在 `boot` 分区创建一个名字为 `ssh` 的新文件夹。重新插回设备，启动，问题解决。


