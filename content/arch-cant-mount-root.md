+++
title = "arch 不能挂载root(You are now being dropped into an emergency shell)"
date = 2018-11-04 12:38:23
[taxonomies]
tags = ["arch"]
categories = ["Problems"]

+++

# 问题描述

这两天很高兴的给自己的新t480装上了arch，将以前的数据移植过来之后在里面快乐的玩耍，结果这天忽然之间更新（`pacman -Syu`）之后，一切都不一样了。电脑开始卡顿，很多应用开始闪退。不久之后，只能看到黑屏。

然后重启

然后gg

	mount /new_root: unknown filesystem type 'ext4'
	You are now being dropped into an emergency shell

# 解决问题

和之前安装arch的时候一样，先到Windows刻录一个arch安装硬盘

然后挂载好root盘和boot盘，`arch-chroot`进到自己的arch中，然后执行`pacman -Syu mkinitcpio systemd-tools linux`

命令如下

	# mount /dev/sdxY /mnt         #Your root partition.
	# mount /dev/sdxZ /mnt/boot    #If you use a separate /boot partition.
	# arch-chroot /mnt
	# pacman -Syu mkinitcpio systemd-tools linux

# 感想

这次出现问题的原因大概是滚动更新滚挂了自己的系统内核。以后更新的时候一定要小心一点，不要在系统不稳定的时候进行更新。

arch的官方文档是十分有用处的。

****************************************

## 参考

[arch pacman 文档](https://wiki.archlinux.org/index.php/pacman "Pacman" )

