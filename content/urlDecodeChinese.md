+++
title = "网址中的中文编码和解码方法"
date = 2019-04-30 13:15:57
[taxonomies]
tags = ["url", "node"]
categories = ["JavaScript"]

+++

当我们在chrome的地址栏上输入中文的进行搜索之后，我们敲入的关键词在这个时候被编码成了我们看不懂的一串由`%`和十六进制数组成的东西。

比如:

	中文搜索

被编码成了

	%E4%B8%AD%E6%96%87%E6%90%9C%E7%B4%A2

所以如何它是如何编码和解码中文的呢？
<!-- more -->

## 原理

根据阮一峰老师的博客[关于URL编码](http://www.ruanyifeng.com/blog/2010/02/url_encoding.html)，我们得知了以下结论:

* 网址路径采用的是UTF-8编码
* 查询字符串的编码用的是操作系统默认的编码
* `GET/POST`用的是网页的编码（GB2312或者UTF-8）
	* `<meta http-equiv="Content-Type" content="text/html;charset=xxxx">`xxxx是啥就进行什么编码
* 在AJAX请求中，IE采用的是系统编码(GB2312)，Firefox 总是采用UTF-8

所有以上的这些结论都是在将中文字符转化为各自的编码集之后在每个字节前加上`%`得到的结果。

比如：`中文`使用utf-8编码得到`e4b8ade69687`，其中`中`对应`e4b8ad`，`文`对应`e69687`。在一个字节前加上`%`就得到了`%e4%b8%ad%e6%96%87`。

至于utf-8具体是如何编码的，请参见：[阮一峰-字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)。

## 解码方法

了解了网址中中问编码的原理，我决定尝试一下使用nodejs实现一个网址解码的函数：

```JavaScript
function urlDecode(str,charset){
	let bytes = [];
	for(let i=0; i<str.length;i++){
		if(str[i]==="%"){
			bytes.push(parseInt(str.substring(i+1,i+3),16));
			i = i+2;
		}else{
			bytes.push(str.charCodeAt(i));
		}
	}
	let buf = Buffer.from(bytes);
	return buf.toString(charset);
}
```
想法自然就是将所有的字节都分离出来，添加为一个数组，将这个数组转为nodejs的二进制数据流Buffer，然后再转为utf-8。
这个方式只能解码utf-8的字符了，如果是使用GB2312 编码参见：[iconv-lite](https://www.npmjs.com/package/iconv-lite)。

以下是该库所支持的编码方式，不仅可用于解码还可以用于编码。

> All node.js native encodings: utf8, ucs2 / utf16-le, ascii, binary, base64, hex.
> Additional unicode encodings: utf16, utf16-be, utf-7, utf-7-imap.
> All widespread singlebyte encodings: Windows 125x family, ISO-8859 family, IBM/DOS codepages, Macintosh family, KOI8 family, all others supported by iconv library. Aliases like 'latin1', 'us-ascii' also supporte
> All widespread multibyte encodings: CP932, CP936, CP949, CP950, GB2312, GBK, GB18030, Big5, Shift_JIS, EUC-JP.


