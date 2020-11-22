+++
title = "Manjaro升级之后tensorflow不见了，并且安装不了"
date = 2018-08-25 06:47:43
[taxonomies]
tags = ["tensorflow", "python"]
categories = ["Problems"]

+++

## 问题描述

最近manjaro升级之后发现我的tensorflow程序运行不了了，终端检查发现我试图重新安装tensorflow也不行。

## 解决方案

出现这个问题的原因的就是manjaro升级之后python的版本也随之升级了，由3.6>>3.7，而tensorflow目前是还不支持3.7的，[#20444](https://github.com/tensorflow/tensorflow/issues/20444 '#20444')

**解决方法**：

1. 关注上述issue，等待tensorflow对python3.7的支持
2. 使用python多版本控制，将你的系统默认的python版本设置为3.6的

对于第二种方法我的采用 `pyenv`这个包来进行我的python版本控制，非常方便。Manjaro可以直接 `pacman -S pyenv`安装。其他的安装和使用方法，详情：[pyenv](https://github.com/pyenv/pyenv 'pyenv')

**使用方法**：

1. `pyenv install --list` 可以查看所有可安装版本
2. `pyenv install 3.6.6` 可以安装3.6.6的版本，以此类推
3. `pyenv global 3.6.6` 将3.6.6设置为系统全局版本
4. `pyenv shell 3.6.6` 将3.6.6设置为终端的使用版本

遇到的坑：

执行`pyenv shell 3.6.6`之后提示

    pyenv: shell integration not enabled. Run `pyenv init' for instructions.

执行 `pyenv init` 之后

    # Load pyenv automatically by appending
    # the following to ~/.zshrc:
    
    eval "$(pyenv init -)"

所以将 `eval "$(pyenv init -)"` 添加到我的终端配置文件 `~/.zshrc` ，这是对于`zsh`而言的，`bash`的情况，添加到`~/.bashrc`即可。重启终端，再执行`pyenv shell 3.6.6` 成功了。


