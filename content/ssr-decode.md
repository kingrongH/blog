+++
title = "ssr和ss链接解码(decode)"
date = 2018-10-07 08:48:01
[taxonomies]
tags = ["shadowsocksr", "shadowsocks"]
categories = ["Shadowsocksr"]

+++

# 关于

我们所熟知的翻墙工具`Shadowsocksr`和`Shadowocks`都有可以直接通过链接导入节点配置的功能。那么它们到底是怎么从ssr或者ss链接的一堆乱码中提取出节点信息的呢？

# 原理

实际上无论是`ssr`或者`ss`链接都是，经过多次`[Base64](https://zh.wikipedia.org/wiki/Base64 ’Base64‘)`编码方式进行加密的，这种加密方式无法起到真正的加密的效果，正所谓`防君子不防小人`。

由于Base64解码后含有`+`、`/`等url敏感字符所以在针对url的Base64编码中，会将`/`改为`_`，将`+`改为`-`。Base64所解码的字符个数必须是4的倍数，不足四的倍数的需要在字符末尾添加`#`补全为4的倍数。

`ssr` 的编码方式就是采用针对url的Base64编码(UrlBase64Encode)的，在解码过程中我发现，ssr加密前的格式是

    server:server_port:protocol:method:obfs:password/?obfsparam=obfs_param&protoparam=protocol_param&remarks=remarks&group=group

其中`password`及之后的`obfs_param`等参数都需要进一步的解码的，并且`obfs_param`及之后的参数在不同的节点中可能是非必须的。

# 测试

本次测试采用[逗比根据地](https://doub.io/sszhfx/)分享的免费帐号。测试工具采用[开源中国在线工具](http://tool.oschina.net/encrypt?type=3)。链接如下：

    ssr://NDUuNjIuMjM4LjE0Nzo1NjA1OmF1dGhfc2hhMV92NDpjaGFjaGEyMDp0bHMxLjJfdGlja2V0X2F1dGg6Wkc5MVlpNXBieTl6YzNwb1puZ3ZLalUyTURVLz9yZW1hcmtzPTVweXM1WVdONkxTNTZMU201WS0zNXAybDZJZXFPbVJ2ZFdJdWFXOHZjM042YUdaNEx3

去除`ssr://`前缀之后，解码结果如下

    45.62.238.147:5605:auth_sha1_v4:chacha20:tls1.2_ticket_auth:ZG91Yi5pby9zc3poZngvKjU2MDU/?remarks=5pys5YWN6LS56LSm5Y-35p2l6IeqOmRvdWIuaW8vc3N6aGZ4Lw

![ssr_decode1.png](https://i.loli.net/2018/10/07/5bb96788bf5a7.png 'ssr decode')

这里不包含混淆参数和协议参数等，我们只需要对`password`和`remarks`进行单独解码：

  ![password](https://i.loli.net/2018/10/07/5bb96884374a5.png 'password')

  ![remarks](https://i.loli.net/2018/10/07/5bb968f07d86d.png 'remarks')

其中我们可以注意到`remarks`这一项中包含一个`-`，我们需要将它改成`+`才能解码成功。

# 想法

在linux环境下使用python 版本ssr翻越长城的方式是十分不方便的，我们还是得必须自己将节点配置添加到json文件中，有没有可能像windows、ios等其他客户端一样通过ssr链接导入节点配置呢。

在下一节中我们将使用python来实现这个代码。 
