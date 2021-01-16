+++
title = "编程语言中的异步IO模型"
date = 2019-09-09 13:44:39
[taxonomies]
tags = ["rust", "aync"]
categories = ["Rust"]

+++

在看先前的拉丁美洲的Rustconf，偶然看到了*Without Boats*大佬的关于零成本异步IO的介绍。

在其中学习到一些异步编程模型，在此做一个总结。 

视频链接：[Without Boats: Zero-Cost Async IO](https://youtu.be/skos4B5x7qE?list=PL85XCvVPmGQjuWUNeFCgl8X2EOC_aAq5N)

<iframe width="889" height="500" src="https://www.youtube.com/embed/skos4B5x7qE?list=PL85XCvVPmGQjuWUNeFCgl8X2EOC_aAq5N" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

<!-- more -->

## 零成本抽象

> What you don’t use, you don’t pay for. And further: What you do use, you couldn’t hand code any better.

这是*Without Boats*在视频里对零成本抽象的介绍。这是由C++发展过来的好东西。我主要有一下两点理解（好吧，就是简单的翻译（逃。。）：

* 对于你用不上的东西，它不会对你的程序性能产生影响
* 对于需要用上的东西，你不能找到更好的方式使性能更佳

提出这个定义的人提出了一个很棒的东西。

## 语言中的异步编程模型

每一种异步IO模型都有相应的典型语言的。

### 绿色线程模型

首先介绍绿色线程，这在rust官网的*The Book*的无畏并发中有设计。Rust官方库中的线程模型是对应的就是OS的线程模型，即1:1，一个OS线程对应的就是一个编程语言的线程，这样换来了Rust中相较于其他编程语言的更小的运行时，意味着更小的开销。不同的编程语言可能有不同的特殊的线程模型，可能是`M`个绿色线程对应`N`个OS线程。所以绿色线程就是不同于OS线程的编程语言中的自己的线程实现。

使用绿色线程来实现异步编程的语言中实现异步的方式就是新创建一个绿色线程，然后在这个绿色线程中执行异步IO代码。

使用绿色线程实现异步IO的语言有：Java和Go

这个方式很棒，但是这个方式不属于零成本抽象的范畴。原因是：这个方式需要在语言层面的Runtime增加去调度绿色线程的代码，这意味着即便你的用户不需要用到异步IO功能，他还是需要为此付出代价。

### **Future** 模型（基于回调）

一个_future_代表着一个还没有准备好的值。

_future_在JavaScript中有另一个名字`Promise`，熟悉JavaScript的同学对这个关键词应该不陌生。在JavaScript中我们给每一个Promise都设置一个回调（resolve, reject）。当我们的Promise准备好的时候，会去调用resolve中的代码或者是reject中的代码。这种方式是非常棒的方式。

在ES2017，JavaScript的作者们将`async`和`await`语法实现到了JavaScript中，这简直极大的方便了我们去编写异步的代码。

通过以上的JavaScript的`Promise`实现。我们可以知道：
1. 使用回调方式的`Future`模型
2. `Future`模型可以实现为`async`和`await`语法

这个方式也非常Amazing，但是这种方式同样不属于零成本抽象的范畴。原因是：使用回调的方式并不是使性能最佳的方式，他还可以有更好的实现。

### *Rust*的异步模型（基于Poll）

基于回调的`Future`模型大抵是这样的：
```rust
trait Future {
    type Output;
    fn schedule<F>(self, callback: F)
    where F: FnOnce(Self::Output);
}
```
基于Poll的`Future`模型大约长这样：
```rust
trait Future {
    type Output;
    fn poll(&mut self, waker: &Waker) -> Poll<Self::Output>;
}
enum Poll {
    Ready<T>,
    Pending,
}
```
观察这两者的区别，我们发现基于回调的模型，我们需要在它有返回值之前就给它设置好一个callback，回调函数；而基于Poll的模型，只是返回一个包含数据的枚举，表示自己准备好了（Ready<T>），还是没有准备好（Pending）。

基于Poll的`Future`模型，给了用户更多的灵活性，但同时意味着，用户必须要做更多的事情。比如，你如果想要得到结果就立即执行相关的代码，你就必须时刻在轮询`poll`这个函数，当它返回Ready时，对相关的数据（T）执行相关的代码。多数时候你不能在主要的线程中执行轮询，所以我们可能需要一个可以管理绿色线程的运行时，比如说`Tokio`，所以这个时候，我们又使用到了绿色线程，但这个时候管理绿色线程的运行时，不是语言本身提供的，而是第三方库提供的。当你用不上异步代码时，你也就不会加载这个多余的运行时。

同样的这个`Future`模型也可以实现为`async`和`await`语法，这是非常棒的，我们期待稳定后的Rust1.39。

这个模型属于零成本抽象的范畴。原因是：它符合了零成本抽象的两条定义。

## 参考连接

[Without Boats Blog](https://boats.gitlab.io/blog/)
[The Book 使用线程同时运行代码](https://rustlang-cn.org/office/rust/book/concurrency/ch16-01-threads.html)
