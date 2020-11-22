+++
title = "JavaScript原型继承的方式探讨"
date = 2019-02-18 17:21:13
[taxonomies]
tags = ["JavaScript"]
categories = ["JavaScript"]

+++

## 关于

基于类和子类的继承方式是我们在别的语言如`python`、`Java`和`C++`中接触到最多的一种模式，这种模式无论是对于创建子类还是创建对象来说都是十分方便的。`JavaScript`算是一个特例吧，它的基于原型链的方式很容易将人们搞的迷糊，对于很多接触过`class`继承方法的人来说都是很难接受的一个事情，其中就包括我，我也是花了较长的一个时间才接受摸清这个事情。

虽然说基于原型链来实现面向对象编程的`JavaScript`在代码上有一些混乱，但是如果我们认真琢磨还是不难发现其中的有迹可循的真相。以下是我对`JavaScript`原型继承的方式的一个终结，适合已经了解了一些`JavaScript`面向对象编程的童鞋阅读，如果有错误欢迎指正。

## 原型继承方式

`JavaScript`的标准一直在更新，新的标准出来也不是所有的厂商和浏览器都能够跟上，所以这就造成了一部分混乱，JS面向对象课程的内容有老的有新的，但没有全的。这也是我在学习中遇到的困境。对于`JavaScript`原型继承的方式，我个人总结主要有以下集中。

* 基于`Object.create()`方式
* 使用`new`关键字
* 使用桥接函数
* `class`关键字


### 基于`Object.create()`方式

这个方法是`ES6`标准新出的，所以使用的时候需要注意支持与否的问题。

假设我们有一个`Student`对象，有一个`name`属性，有一个`hello`方法：

```JavaScript
function Student(props){
	this.name = props.name || "Unamed";
} 
Student.prototype.hello = function(){
	alert("Hello " + this.name + "!");
}
```
我们想要创建一个新的对象小学生`PrimaryStudent`继承自`Student`，使用`Object.create()`方法我们需要这么做
```JavaScript
function PrimaryStudent(props){
	Student.call(this,props);
}
PrimaryStudent.prototype = Object.create(Student.prototype);
PrimaryStudent.prototype.constructor = PrimaryStudent;  //因为我们重写了PrimaryStudent的原型所以我们需要重新改回它的构造函数
var xiaohua = new PrimaryStudent({name:"xiaohua"});
```
这个时候我们看看`xiaohua`的原型链是不是符合我们的要求

```JavaScript
xiaohua ---> PrimaryStudent.prototype ---> Student.prototype --->Object.prototype ---> null
```
从这个原型链我们就可以看出`PrimaryStudent`确实是继承自`Student`的。

```JavaScript
xiaohua.name; //”xiaohua“
xiaohua.hello(); //”Hello xiaohua!“
```

### 使用`new`关键字

准确说这种方法和上面的方法本质上是一模一样的，区别在在于我们把`PrimaryStudent.prototype = Object.create(Student.prototype);`这一行替换为了`PrimaryStudent.prototype = new Student({})`，其他的代码是一样的。

这两种方法在原型链上都是一样的，调用起来也可以达到一样的效果。

但是我们注意到一个问题，使用这种方式的时候我们不得不在`Student()`中加上参数，因为这是该构造函数所必须要求的，这种导致一个问题，我们的`PrimaryStudent.prototype`的属性中多了一个`name`，冗杂的属性，这并不是我们所需要的。

```JavaScript
PrimaryStudent.prototype.hasOwnProperty("name"); //true
```

我们可以使用接下来的方法来修复。

### 使用桥接函数

使用桥接函数的方法来自`JSON`的发明者道格拉斯，我觉得这个方法是上一个方法的改进，桥接函数的核心是空函数`F()`，创建一个空函数`F`之后，令`F.prototype = Student.prototype`，然后再从`F`上创建新对象并赋值给`PrimaryStudent.prototype`，最后再对`PrimaryStudent`的构造函数进行修正。据此我们得到的`PrimaryStudent`确实是继承自`Student`的，并且没有属性上的冗余。

```JavaScript
function Student(props) {
	this.name = props.name || 'Unnamed';
}

Student.prototype.hello = function () {
 	alert('Hello, ' + this.name + '!');
 }
function PrimaryStudent(props){
    Student.call(this,props);
}
var F = function(){};
F.prototype = Student.prototype;
PrimaryStudent.prototype = new F();
PrimaryStudent.prototype.constructor = PrimaryStudent;

var xiaohua = new PrimaryStudent({name:"xiaohua"});
```

道格拉斯这个方法可以说是很机智了，我们且来看看`xiaohua`的原型链：

```JavaScript
xiaohua ---> PrimaryStudent.prototype ---> Student.prototype ---> Object.prototype ---> null
```

符合我们的要求并且`PrimaryStudent`中没有冗余的属性。

```JavaScript
PrimaryStudent.prototype.hasOwnProperty("name");  //false
```

我们发现这个继承的方法可以复用，所以我们可以写以用于继承的函数，专门用来连接“子”与“父”，在此我们却不来实现了，可以在参考链接找一找哦

### Class继承

这种方式可谓是最贴近真实，子类与父类是我们在学习其他语言经常就能接触到的，使用这种方式完成继承的关系，是我们更容易接受的一件事情。但是`JavaScript`并没有实现真正的`Class`的概念，现实仍然是`Class`包装下的原型链，所以我们还是有必要学习Class出现之前的继承方式，更有利于我们理解`JavaScript`原型链的概念。


`Class`关键字是ES6标准新定义的，所以我们在使用的时候也还是要考虑到兼容性问题。

使用`Class`定义一个`Student`：
```JavaScript
Class Student(){
	constructor(props){
		this.name = props.name;
	}
	hello() {
		alert("Hello "+ this.name + "!");
	}
}
```
使用`Class`定义一个`PrimaryStudent`继承自`Student`：

```JavaScript
Class PrimaryStudent extends Student(){
	constructor(props){
		super(props);
	}
} 
```

`Class`的方法简直不能太好用！！！


## 结尾

以上便是我个人在学习过程中总结出来的一些JavaScript的继承的方式，希望能够帮到你，更希望你能够在评论区提出一些意见，感谢观看！


**参考链接：**

* [廖雪峰的JavaScript教程](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/0014344997013405abfb7f0e1904a04ba6898a384b1e925000)
* [Object.create()用法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)






