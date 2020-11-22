+++
title = "git使用"
date = 2018-08-15 20:50:13
[taxonomies]
tags = ["git"]
categories = ["Learn"]

+++

## git简易的命令使用

因为忽然想将自己的博客网站的hexo本地文件存到github上，以达到多机使用的目的。所以索性学习总结一下git的命令使用

### 新建仓库
  
  使用到的命令：

    git init

  1. 建立一个本地文件夹
  2. cd到该文件夹
  3. 执行 `git init`  即可

### 克隆到本地

  使用到的命令

    git clone
  这个命令的使用还是较为频繁的，很多开源软件的安装都是直接使用的git clone 

  使用
  
    git clone http://www.github.com/某仓库.git 你本地的文件夹

### 添加与提交
  
  使用到的命令
  
    git add 和 git commit
  工作流程：

  1. 使用`git add <filename>` 将推送到缓冲区
  2. 使用`git commit -m "代码改动信息"`将改动的文件提交

  做完这些后文件并没有被存到仓库中，相当于提交到了本地的仓库中  
  另外：
  
  1. 使用 `git status` 命令可以检查是否有改动的文件还未提交 
  2. 使用 `git diff <filename>` 查看改动详情

***

### 待续...
