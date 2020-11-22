+++
title = "JavaScript 链表的实现"
date = 2019-04-04 10:57:20
[taxonomies]
tags = ["链表", "JavaScript", "algorithm"]
categories = ["Algorithm"]

+++

链表作为数据结构中的基础数据结构由许多的节点（node）组成。节点有两个东西组成：

* 数据
* 指向一下个节点的指针

使用JavaScript实现一个链表的结构需要创建两个类：一个是节点；一个则是链表。
创建好两个类之后，我们需要为我们创建的这个链表结构定义许多方法。

## 链表的方法

通常情况下我们创造一个链表是为了更加方便的操作数据所以链表需要包含一些基本操作方法。

链表需要包含的操作方法有：

* 在之后插入：`append(node,value)`
* 搜索：`find(value)`
* 删除：`remove(value)`

以上的方法只是一个大致的表示，具体实现还是要看代码怎么写。

## 链表类的实现

我们需要创建两个类：

* Node(value)
* Linkedlist(head)

其中链表又头部定位，所以创建时需要传入头部参数

代码的实现也比较简单：
```JavaScript
//define Linkedlist‘s node
function Node(data){
	this.data = data;
}
Node.prototype = {
	constructor:node,
	next:null
};

// Define Linkedlist  and its methods
function Linkedlist(head){
	this.head = head;
}
Linkedlist.prototype = {
	constructor:Linkedlist,
	append:append,
	find:find,
	remove:remove,
};
```
我们使用构造函数来实现`Node`和`LinkedList`两个类，并且在其原型中预先定义好了方法。

## 链表方法的实现

链表方法的实现却也是比较简单的。详情请看代码。

```JavaScript
/*
 * Next define the methods:
 * append(node,value)
 * find(value)
 * remove(value)
 */ 
function append(node,value){
	var NewNode = new Node(value)
	NewNode.next = node.next;
	node.next = NewNode;
}

function find(value){
	node = this.head;
	while(value !== node.data && node.data !== null){
		node = node.next;
	}
	return node;
}

function remove(value){
	node = this.head;
	if (head.data === value) {
		this.head=node.next;
		return;
	}
	while(node.next.data !== value && node.next.data !== null){
		node = node.next;
	}
	if(node.data !== null){
		node.next = node.next.next;
	}
}
```

在这里我们只是实现了链表结构的三个方法，如果需要实现更多的方法，请各位看官手动尝试。

## 总结

使用JavaScript来实习链表这样一个结构目的是为了更加深刻的理解数据结构中的链表的概念。关于链表的更多的知识可以参考：[链表wiki](https://en.wikipedia.org/wiki/Linked_list)

如有错误，欢迎各位指正！也欢迎各位在评论区提出意见。

**********************************************************************************

参考链接：

[github Javascript-algorithms](https://github.com/trekhleb/javascript-algorithms/tree/master/src/data-structures/linked-list)
