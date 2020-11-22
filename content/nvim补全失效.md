+++
title = "更换python环境后 neovim自动补全失效"
date = 2018-09-10 21:49:55
[taxonomies]
tags = ["vim", "deoplete", "python"]
categories = ["Problems"]

+++

# 问题描述

更换python环境后，打开neovim自动补全就失效了。

使用的补全插件：`deoplete`

开发环境： `Pyhton`

# 解决办法

这是因为更换了python的环境当前`deoplete` 补全插件失去了依赖，我的解决办法是

      pip insall nvim

重新打开`nvim` 后补全又回来了，
