+++
title = "JavaScript 解析vmess链接，供clash-linux使用"
date = 2019-04-12 14:35:01
[taxonomies]
tags = ["JavaScript", "vmess", "Clash"]
categories = ["JavaScript"]

+++

今天我决定将我使用JavaScript开发了一个vmess解析工具的记录po给大家看。这个工具主要是为[clash](https://github.com/Dreamacro/clash) linux版本设计的，包含获取vmess订阅链接内容，和解析单个链接等功能。

<!--more-->

## 关于

之前，我曾经使用过Python来解析ssr和ss链接，当时为自己做了这样一个小工具而兴奋不已，但是这样一个过于简单的没有经过系统设计的工具，难以逃脱短命、不被更新和被主人抛弃的命运。当时因为在Linux系统上没有ssr订阅链接的工具而倍感苦恼，下决心自己写下来一个可以订阅ssr的工具，甚至还打算了为我自己的工具做一个界面。然后，在我的脑海中夭折，甚至已经出来的功能都是现学现做的，漏洞甚多，弃。

当时这样的想法算是一时冲动，但是同现在的想法却有几分相似。我想做这样一件事情是因为：

* clash-linux作为一个工具来说非常的好用
* clash使用yaml配置文件作为它节点的接口，非常适合进一步开发
* clash没有支持vmess链接导入的功能
* 我喜欢折腾

大抵是因为这样，我为自己做了这样一个工具。

## 一一对应

在手动配置clash的config.yml的时候，我便有一些困惑，和其他vmess工具使用配置属性名称都不一样。

一个clash的vmess配置大概长这样：

	{ name: "vmess", type: vmess, server: server, port: 443, uuid: uuid, alterId: 32, cipher: auto, tls: true }

而v2ryN的配置长这样：
```
{
"v": "2",
"ps": "备注别名",
"add": "111.111.111.111",
"port": "32000",
"id": "1386f85e-657b-4d6e-9d56-78badb75e1fd",
"aid": "100",
"net": "tcp",
"type": "none",
"host": "www.bbb.com",
"path": "/",
"tls": "tls"

}
```
两个配置文件的范例分别可以在[clash](https://github.com/Dreamacro/clash) 和[v2rayN wiki](https://github.com/2dust/v2rayN/wiki/%E5%88%86%E4%BA%AB%E9%93%BE%E6%8E%A5%E6%A0%BC%E5%BC%8F%E8%AF%B4%E6%98%8E(ver-2))。

我并没有一下子就建立起来两者的一一对应关系，而是花了一些时间。我在接下来的格式解析的过程中肯定需要使用到这样一个一一对应的关系的：

```JavaScript
var table = {
	ps:"name",
	add:"server",
	port:"port",
	id:"uuid",
	aid:"alterId",
	security:"cipher",
	tls:"tls", // vmess中value为"tls"或者""，而clash中为true/false
	net:"network" //vmess 支持tcp，clash不支持
};
```
其中一些需要注意的点写在了注释中，另一些vmess有的而clash没有的要么是不支持要么是没紧要的。

## JavaScript解码base64

vmess链接的格式为`vmess://[base64编码的节点信息]`所以我自然需要知道在JavaScript如何解码base64的字符串。我的代码如下：

```JavaScript
/* decode base64
 * note: because vmess support return for a encoded string, add a trim char check
 * @param s string 
 * @return string
 */
function decode(s){
	//delete the \n\r 
	s.replace(/\s+/g,"");
	if(s.length%4 !== 0){
		s = s + s.length%4*"=";
		console.log(s);
	}
	var buf = Buffer.from(s, 'base64');
	return buf.toString('utf8');
}
/* decode vmess, ss or ssr link
 * @param s string
 * @return string
 */
function linkDecode(s){
	//judge if supported link
	if(s.startsWith("vmess://")){
		return decode(s.substring(8));
	}else if(s.startsWith("ssr://")){
		return decode(s.substring(6));
	}else if(s.startsWith("ss://")){
		return decode(s.substring(5));
	}else{
		console.warn("a illegal link: "+ `${s}`);
	}
}
```
这里包含两个部分，一个单纯的解码字符串，一个是针对链接来解码，为了以后，我多做了一些工作，不过可能是白做，哈哈哈。

之所以去除字符串中的空白字符，是因为在测试过程中发现v2rayN支持的先加密后换行的格式。

## 解析为json对象

获取到字符串格式的节点信息之后，需要将这些信息提取出来到一个JavaScript对象中，以便做进一步处理

```JavaScript
/* parse string to obj
 * @param s string
 * @return object
 */
function parse(s){
	if(typeof s !== "string"){
		console.warn(`parsing a not-string obj: ${s} aborted!`)
	}else{
		var ob = JSON.parse(s);
		return ob;
	}
}
```
返回包含节点信息的对象ob

## 转换为clash格式

参照我们之前制作的那个table对象，我们再将获得的含有节点信息的对象，一一转换为clash支持的形式，并返回新的对象。我的代码：

```JavaScript
/* convert the obj from vmess link to the obj that clash accept
 * @param ob object
 * @return object
 */
function clashFormat(ob){
	if(typeof(ob)!=="object"){
		console.warn("clash Format:can only format object");
	}else{
		//a table for refer
		var table = {
			ps:"name",
			add:"server",
			port:"port",
			id:"uuid",
			aid:"alterId",
			security:"cipher",
			tls:"tls"
		};
		var defaultV = {
			name:"local",
			type:"vmess",
			server:"127.0.0.1",
			port:1080,
			uuid:"",
			alterId:0,
			cipher:"auto",
			tls:false,
			"skip-cert-verify":true
		};
		for(var key in ob){
			if(key === "tls"){
				if(ob[key]==="tls"){
					defaultV[table[key]] = true;
				}
			}else if(table[key]){
				defaultV[table[key]] = ob[key];
			}
		}
		return defaultV;
	}
}
```
我建立一个默认值的对象，便于修改。对我们获取到的对象中的属性遍历，如果是table中存在的，将defaultV中对应的值赋给ob中对应的值，最后返回defaultV。其中我们需要对一些特殊的属性做一些特殊处理，比如`tls`它在两种格式中值的类型是不一样的，需要做转换。

目前我还没有加入`network`支持，因为我所知的大部分机场都使用的`tcp`而clash还没有支持这个。

## 序列化为字符串

对于获得clash格式的对象，我们需要将它序列化为字符串，便于输出，无论是写入文件还是输出到终端。

```JavaScript
/* parse object to yarml format, return a string
 * note: this only can parse obj returned from clashFormat()
 * @param ob object
 * @return string
 */
function objParseToYaml(ob){
	if(typeof(ob)!=="object"){
		throw Error("Only can parse object");
	}
	var s = JSON.stringify(ob);
	return s;
}
```
事实上，一开始我以为，clash所支持的yaml的配置是需要去除引号，但是保留name值的引号的，因为官方的配置是只保留了name值的引号，所以我费了一番功夫，想要保留这个引号同时去除其他的引号。后来经过我的一些测试发现不需要这么做，一个引号也不用去除。为了保证clash不会报错，我确实一个也没有去除。只是美观不足而已，所以得到的字符串大概长这样

	{"name":"xxxx","type":"vmess","server":"xxxx.xxx.xx","port":"443","uuid":"27848739-7e62-4138-9fd3-098a63964b6b","alterId":"0","cipher":"auto","tls":false}

如果将这个放到配置文件中却是不怎么好看。

到这里我们的工作可以说完成了核心部分了。但是我们还有更进一步的事情需要做，比如输出，比如获取订阅链接。不过都是一些外层的设计，但是对我来说，却是更难的部分。

## 获取订阅链接内容

获取订阅链接的内容，在JavaScript中我们需要调用到`http`和`https`，因为我们无法保证机场使用的都是一样的协议，所以我们两个都引入。

```JavaScript
const http = require("http");
const https = require("https");
```
nodejs中几乎所有的http和https操作都是异步的，所以我们需要考虑到返回值的问题。相对于其他的编程语言来说，获取响应这个部分确实是更加的麻烦了。但是在这样的繁复中至少我们还有语法糖`async/await`，整个编码过程都甜蜜了一些。

一开始我以为，获取订阅链接响应的代码只要这么写就够了：

```JavaScript
/* handle subscribe link, get the response
 * @param url
 * @return Promise resove(string)
 */
function getRes(url){
	if(typeof url !== "string" || !(url.startsWith("http://") || url.startsWith("https://"))){
		throw new Error("Only can handle link format!");
	}
	return new Promise((resolve)=>{
		if(url.startsWith("http://")){
			var req = http.get(url,(res)=>{
				if(res.statusCode !== 200){
					throw new Error("Request failed please check your Internet connection!\n" + `statusCode: ${res.statusCode}`);
				}
				res.on("data",(chunk)=>{
					resolve(chunk)
				});
			});
		}else if(url.startsWith("https://")){
			var req = https.get(url,(res)=>{
				if(res.statusCode !== 200){
					throw new Error("Request failed please check your Internet connection!\n" + `statusCode: ${res.statusCode}`);
				}
				res.setEncoding("utf8");
				res.on("data",(chunk)=>{
					resolve(chunk);
				});
			});
		}
	});
}
```
看起来似乎没什么问题，我返回了一个Promise对象，resolve了我请求到的数据，并且考虑到了种种Error的情况，在接下来的操作中我只需要`await`这个函数完成，返回我们的响应数据，多么美好。

然而我还是太天真了，有一些机场他们更愿意使用`stream`类型来传回数据，就连我们获得的响应也都是突突突一阵一阵的（忽然想起来普朗克），这种情况下，我可能只是获得了第一阵的数据，就开始急急忙忙地进行下一步操作了。所以我们应该等整个请求完成了再`resolve`获得的整个数据。代码看起来如下：

```JavaScript
/* handle subscribe link, get the response
 * @param url
 * @return Promise resove(string)
 */
function getRes(url){
	if(typeof url !== "string" || !(url.startsWith("http://") || url.startsWith("https://"))){
		throw new Error("Only can handle link format!");
	}
	return new Promise((resolve)=>{
		var strings = [];
		if(url.startsWith("http://")){
			var req = http.get(url,(res)=>{
				if(res.statusCode !== 200){
					throw new Error("Request failed please check your Internet connection!\n" + `statusCode: ${res.statusCode}`);
				}
				res.on("data",(chunk)=>{
					strings.push(chunk);
				});
			});
		}else if(url.startsWith("https://")){
			var req = https.get(url,(res)=>{
				if(res.statusCode !== 200){
					throw new Error("Request failed please check your Internet connection!\n" + `statusCode: ${res.statusCode}`);
				}
				res.setEncoding("utf8");
				res.on("data",(chunk)=>{
					if(chunk){
						strings.push(chunk);
					}
				});
			});
		}
		// once the request done, join the chunks together, and resolve it
		req.on('close',()=>{
			resolve(strings.join(""));
		});
	});
}
```

我们在返回的Promise对象中定义了一个字符串数组，并且将每一次获取到的数据都`push`进去，在请求结束之后，将数组中所有的数据`join`到一起，然后`resolve`它，美滋滋。

## 集中处理订阅数据

我们获取到了一堆base64编码后的数据它很长，我们要对这个长长的数据做进一步处理：解码、解析和输出。这个数据在base64编码之前是以换行符分开不同的`vmess`链接的，我们获取该数据后，解码，再以空白字符分割，对每一个链接再进行解码，解析，转换clash格式，序列化和输出。

```JavaScript
// async function used to handle data get from getRes()
// split the data to the list and convert ervery one to the clash supported format
// and parse the link to the yarml string 
async function handleSubData(subUrl){
	var data = "";
	data = await getRes(subUrl);
	var linkList = decode(data).split(/\s+/);
	var configs = [];
	var configStrings = [];
	var names = [];
	for(link of linkList){
		//if link is not empty string
		if(link){
			let	temp = linkDecode(link);
			let tempObj = parse(temp);
			let tempClashObj = clashFormat(tempObj);
			configs.push(tempClashObj);
			names.push(tempClashObj.name);
			let tempClashStr = objParseToYaml(tempClashObj);
			// log like this is to make it easier to copy and paste
			console.log("- " + tempClashStr);
			configStrings.push(tempClashStr);
		}
	}
	console.log(names);
}
/* cli support 
 *
 *
 */
async function shellStart(){
	const rl =readline.createInterface({
		input:process.stdin,
		output:process.stdout
	});

   	rl.question("Input the vmess link: ", (answer)=>{
		handleSubData(answer);
		console.log("Please wait the result!");
		rl.close();
	});
}
```
你需要执行`shellStart()`，然后填入你的订阅链接，就会出来结果。

我输出了所有可以直接复制进去`config.yml`的所有节点信息和由所有节点的name值组成的一个数组，方便操作。

事实上，我可以使用更加美好的输出方式，比如输出到文本文件中，直接生成`config.yml`或者干脆做一个GUI，但是我没有，我只使用了`console.log`，你需要自己复制进去。也许未来我可以把更好的方式添加进去。



## 感想

这个小工具花了一天多的时间。在编码的过程中，我尽量先在笔记中写下自己的思路，然后才开始编码。这种方式对我来说实在是大有裨益，难怪过来人都喜欢这么说:

>一个好的程序员应该花80%的时间编写文档，然后花20%的时间来写代码和debug。

我对于这样一句话，现在有了更加深刻的理解。

## 结尾

这样短小的代码量，还是有好些地方做不好，比如，也许你发现了，我在处理错误上并没有做好，我希望能够在接下来的时间慢慢地改进。

接下来还有很多要做的：

##### todo:

* 更好的错误处理方式，比如说写一个logger
* 更好的输出方式，比如生成`config.yml`
* 更好的交互方式，比如cli+参数运行clash
* 更多的支持，比如支持`ss`
* ...

如果你能有更好的方式或者建议欢迎你在评论区提出，或者直接联系我[github](https://github.com/kingrongH)。

************************************************************************************************************************
参考链接：
	1. [clash](https://github.com/Dreamacro/clash)
	2. [v2rayN wiki](https://github.com/2dust/v2rayN/wiki)

