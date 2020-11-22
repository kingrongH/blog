+++
title = "fcitx-rime输入法配置"
date = 2018-08-18 12:10:44
[taxonomies]
tags = ["linux", "rime", "fcitx-rime"]
categories = ["Learn"]

+++

## 由来

使用rime输入已经有一段时间，但是很惭愧，一直使用的是默认的配置，配置rime输入法这件事情一直也都在日程上，至今才开始配置。

对默认的配置有一下两点不满：

* 皮肤太丑
* 词库不全

在这一次配置中着重解决以上两个问题。

## 配置技巧

在 `~/.config/fcitx/rime/` 目录下新建 `default.custom.yaml` 将你需要更改的配置由`default.yaml` 复制到其中的 `patch:` 下，rime会优先考虑补丁(patch)里的设置，如：

    patch:
  
      schema_list:
        - schema: luna_pinyin
        - schema: luna_pinyin_fluency
    #    - schema: bopomofo
    #    - schema: bopomofo_tw 
    #    - schema: cangjie5
    #    - schema: stroke
    #    - schema: terra_pinyin

    
    menu/page_size: 6

具体的某个输入法的设置可以依法炮制，比如新建 `luna_pinyin.custom.yaml`

    patch:
    
      switches:
        - name: ascii_mode
          reset: 0
        states: [ 中文, 西文 ]
        - name: full_shape
          states: [ 半角, 全角 ]
        - name: simplification
          reset: 1
          states: [ 漢字, 汉字 ]
        - name: ascii_punct
          states: [ 。，, ．， ] 

这种做法可以让我们在轻松的保存自己的配置，以便进行同步。


## 添加皮肤

因为看着默认的fcitx的皮肤太丑，共有三个选项：`dark`、`default`和`classic`，任何一个都不让人满意。所以在网络上找了许多资料配置皮肤。

### ibus-rime 
ibus-rime的皮肤配置写入如下文件就好

`~/.config/ibus/rime/default.custom.yaml`

记录一个配置皮肤的网页工具：

  `https://rime.netlify.com/`

### fcitx-rime

在fcitx下尝试了很多遍修改配置的方法并不奏效，所以我觉得fcitx的皮肤配置与输入法是分离的，可以调整fcitx的皮肤但是不能调整输入法的皮肤。

方式：

下载皮肤配置文件并复制到目录 `~/.config/fcitx/skin/`下就可以切换了。

推荐皮肤：

  1. `fcitx-skin-material`
   arch下可以直接使用`pacman -S fcitx-skin-material`进行安装。

  2. `mantis`

      
    git clone https://github.com/fcitx/fcitx-artwork
    cd fcitx-artwork
    cp -r skin ~/.config/fcitx/

## 词库添加

使用的工具 `深蓝词库转换`：

`https://github.com/studyzy/imewlconverter`

下载该工具用wine运行，或者在windows下运行。

然后下载你所需要的搜狗细胞词库：
  
  `https://pinyin.sogou.com/dict/cate/index/`

导入到 `深蓝词库转换`然后转换为 `Rime输入法` 格式。
将转换后的内容写在以 `***.dict.yaml` 命名的文件中，如：

    # luna_pinyin
    # Rime dictionary 计算机
    # encoding: utf-8

    ---
    name: luna_pinyin.computer
    version: "2018.08.18"
    sort: by_weight
    use_preset_vocabulary: true
    import_tables:
      - luna_pinyin
      - luna_pinyin.computer
    ...

    阿姆达尔定律	a mu da er ding lv	1
    阿帕网	a pa wang	1
    埃尔布朗基	ai er bu lang ji	1
    埃尔米特函数	ai er mi te han shu	1
    埃克特	ai ke te	1

其中以下部分需要自己添加，乃格式所需不可缺少。                                  

    ---
    name: luna_pinyin.computer
    version: "2018.08.18"
    sort: by_weight
    use_preset_vocabulary: true
    import_tables:
      - luna_pinyin
      - luna_pinyin.computer
    ...

然后将该文件复制到 `~/.config/fcitx/rime/` 下，在 `luna_pinyin.custom.yaml` 中添加以下内容即可

    #添加计算机词库，来自搜狗细胞词库转换而来
    translator:
      dictionary: luna_pinyin.computer

添加多个词库可以依法炮制，或者将多个词库合并为同一个词库。重新部署，这个时候尝试一下输入词库中的内容，你会发现已经可以了。 
