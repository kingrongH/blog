+++
title = "Python版ssr通过链接导入节点"
date = 2018-10-08 18:45:12
[taxonomies]
tags = ["shadowsocksr", "shadowsocks"]
categories = ["Python"]

+++

# 关于

很多时候我们都苦于使用python版本的ssr的时候必须自己手动写入配置，所以这一次我们将使用python来进行ssr解析并导入配置。在这一节中我们需要用到许多上一节的内容，所以我建议各位看官，先移步[ssr和ss链接解密](https://www.kingr.top/2018/10/07/ssr-decode/)

本节所有代码都可以在我的[github](https://github.com/kingrongH/shadowsocksr/tree/manyuser/shadowsocks/RSS)中找到。

# 准备

首先我们需要知道ssr或者ss的链接是怎样加密的，在前一节中我们解析了ssr链接的加密过程，我们知道了ssr链接是通过`Base64`的编码方式进行加密的。所以在这一次试验中我们需要用到的主要的库就是`base64`和`re`，这两个库一般linux的python环境中都自动安装了，如果你出现了`no module nameed ***`的问题，那么可以使用`pip`安装一下。
    
    import base64
    import re
    import json
    
# 解码

在上一节中我们使用的在线的`Base64`解码工具进行解码的，这一次我们需要使用Python的`base64`这个库，官方使用文档在此：[Base64官方文档](https://docs.python.org/3/library/base64.html)。这个库的使用方式十分简单，我们一般会使用到的是这个库的两个函数：`url_safe_base64decode(s)`和`base64_decode(s)`，传入我们需要解码的对象，然后返回的是解码后的`byte`类型的字符。

在这里我们需要特别注意的是我们传入的s的长度必须为4的倍数，这一点在上一节中我们实际上已经谈到过了。不再多说直接上代码

    def decoe(s):
      num = len(s)%4
      if num==0:
        s = base64.url_safe_base64decode(s)
      else:
        s = s + '#'*(4-num)
        s = base64.url_safe_base64decode(s)

        return s

这样一个简单的base64 针对url的解码函数就出炉了，因为我们的ssr链接是需要使用在浏览器地址栏的，所以我们需要使用针对url的函数。

# 解析

完成了解码函数之后我们需要对我们的ssr链接进行解析了。我们需要得到的是一个由配置组成的字典例如：

    
    config = {
        "server": "0.0.0.0",
        "server_ipv6": "::",
        "server_port": 8388,
        "local_address": "127.0.0.1",
        "local_port": 1080,

        "password": "m",
        "method": "aes-128-ctr",
        "protocol": "auth_aes128_md5",
        "protocol_param": "",
        "obfs": "tls1.2_ticket_auth_compatible",
        "obfs_param": ""
    }

利用解码函数我们可以得到我们的每一个参数，然后存入字典中，参照上一节的文档我们需要对这个链接进行多次解码，代码如下：

    def Analyze(s):
         config = {
        "server": "0.0.0.0",
        "server_ipv6": "::",
        "server_port": 8388,
        "local_address": "127.0.0.1",
        "local_port": 1080,

        "password": "m",
        "method": "aes-128-ctr",
        "protocol": "auth_aes128_md5",
        "protocol_param": "",
        "obfs": "tls1.2_ticket_auth_compatible",
        "obfs_param": ""
      }

      #第一次解码
      s = decode(s)
      spilted = re.spilt(':',s)  #将多个参数分离开来
      pass_param = spilted[5]
      pass_param_spilted = re.spilt('\/\?',pass_param)
      passwd = decode(pass_param_spilted[0]) #解码得到password

      #匹配param、remarks和group
      try:
        obfs_param = re.search(r'obfsparam=([^&]+)',pass_param_spilted[1]).group[1]
      except:
        obfs_param=""
      try:
        protocol_param = re.search(r'protoparam=([^&]+)', pass_param_spilted[1])
        protocol_param = decode(protocol_param)
      except:
        protocol_param = ''
      try:
        remarks = re.search(r'remarks=([^&]+)', pass_param_spilted[1]).group(1)
        remarks = decode(remarks)
      except:
        remarks = ''
      try:
        group = re.search(r'group=([^&]+)', pass_param_spilted[1]).group(1)
        group = decode(group)
      except:
        group = ''
        
        #将各个值赋值入字典中

      config['server'] = spilted[0]
      config['server_port'] = int(spilted[1])
      config['password'] = passwd
      config['method'] = spilted[3]
      config['protocol'] = spilted[2]
      config['obfs'] = spilted[4]
      config['protocol_param'] = protocol_param
      config['obfs_param'] = obfs_param
      
      return [config,group,remarks]

这些代码大致还原了我们在上一节中使用在线工具解析的过程，其中我们可以看见，在匹配参数的过程中我们一直在使用`try...except...`，这是因为并不是所有的ssr链接中都包含了这些信息，如果不包含，我们就可以将它赋值为空文本。

# 保存配置

在python版本的ssr的使用中我们一般是使用`python local.py -c ***.json -d start` 命令来完成ssr的启动的。所以我们为了便捷和进一步开发的需要需要将我们的配置保存为`*.json`，使用`json`库，[官方文档](https://docs.python.org/3/library/json.html)

    def save_as_json(d,name='conf'):
  
      #直接调用解析函数
      [data_dict,group,remarks] = Analyze(d)
      json_str = json.dumps(data_dict)

      #保存在config目录下
      with open('config/'+naem+'.json','w') as f:
        json.dump(data_dict,f)
      

# 综合利用

我们直接使用在终端输入的方式来获取ssr链接，然后进一步进行解析保存到相应的目录下，这里需要注意一点就是我们的所有使用的ssr code 都是去除了`ssr://`前缀的。

    if __name__ == "__main__":
      ssr = input('ssr link:')
      code = ssr[6:]
      name = input('config name:')
      try:
          save_as_json(code,name)
          print("Successful:please check config at \'config/\'")
      except:
          print("Error:Fail to save config!")

# 更进一步

将这一切做好之后，我们将获得一个可以通过ssr链接导入节点配置的程序，然而在使用过程中还是稍显麻烦，我觉得主要有以下几点值得进一步地探索：

* 文件的名字还是自定义的，能不能通过我们获得的group和remarks信息完成对保存的名字和文件夹的自动设定呢
* 能不能进一步完成ssr的订阅功能呢，像其他客户端一样。

