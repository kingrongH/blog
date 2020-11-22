+++
title = "Rust 错误处理探索"
date = 2019-09-12 10:12:59
[taxonomies]
tags = ["rust", "error"]
categories = ["Rust"]

+++

从JavaScript，从Java，从Python甚至是从C过来，都会不习惯_Rust_的错误处理方式。但这不是rust限制了你错误处理的方式。事实上rust提供了很大的自由度来进行错误处理，灵活的方式。

对于Rust来说，`Option`和`Result`很大程度上造成了你在错误处理上的不适感。

要熟练地使用这些方式并不简单，但当你熟练之后，你应当会十分地enjoy这个过程。

这篇博客简单记录了一下我最近遇到的错误处理的方式。
<!--more-->

## 简单处理 

* `unwrap()`
* `expect()`

对我来说，最为直接而简单的处理`Option`和`Result`的方式就是`unwarp()`，这种方式给你一种在其他语言中编程的体验，但是总是使用这种方式并不是很好的选择。如果你并不能确定你的代码会得到什么样的结果，那么最好不要在生产环境中使用这种方式。

但是有一些时候我们可以毫不犹豫地使用这种方式——但是确定这是什么结果的时候。比如说：

```rust
use regex::Regex;
let gender_re = Regex::new(r"input\[name='sexs'\]\[value='(.+?)'\]").unwrap();
```

另外我觉得也可以在项目开发和测试过程中使用`unwrap()`或者`expect`来处理，来快速看看结果是否符合预期以及获得一些简单的提示。

## 高级

* `ok_or()`和`map_err()`
* `dyn std::error::Error`
* `?`运算符
* 新建错误类型和包装错误

### 介绍

我非常喜欢像`ok_or`、`map_err`等的方式，来转换错误。

* `ok_or()`将`Option<T>`转换为`Result<T, Err>`。
* `map_err()`将`Result<T, E>` 转换为`Result<T, F>`，即转换为另一个错误类型。

这两种方式结合自定义错误和`?`运算符以及自定义的错误可以写出非常简洁的代码。很多时候我们并不喜欢写大段的`match`，虽然我们很喜欢`match`语法。

`dyn std::error::Error` 方式很强大，你给任何一个实现了`std::error::Error` _trait_的错误，它都照单全收。

```rust
fn some_fn(param: Result<&str, &str>) -> Result<i32, Box<dyn std::error::Error>> {
    match param {
        Ok(n) => n.parse::<i32>()?,
        Err(_e) => Ok(0),
    }
}
```

新建一个错误的过程十分明了：

* 定义错误
* 实现`std::fmt::Display` trait
* 实现`std::error::Error` trait
* 如果需要实现包装错误，实现`From`trait

详见[Define an error type](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/define_error_type.html)

### 案例

以下代码是我最近一次使用`reqwest`模拟登录中获取cookie的代码（网址非真实网址）。

```rust
fn get_cookie() -> Result<String, LoginError> {
    let url = "http://www.example.com/example";
    let resp = Client::new().get(url)
        .header(USER_AGENT, "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.131 Safari/537.36")
        .send().map_err(reqwest_result_handle)?;
    let set_cookie = resp.headers().get(SET_COOKIE).ok_or(LoginError::NoSetCookie)?; 
    let set_cookie = (set_cookie.to_str().map_err(|_e| LoginError::Other)?).to_string();
    let re = Regex::new(r"PHPSESSID=([\w\d]+?);.*");
    let re_result = re.unwrap().captures(&set_cookie).ok_or(LoginError::NoSessId)?;
    Ok(re_result[1].to_string())
}
```

其中`LoginError`是自定义的一个错误枚举，`reqwest_result_handle`是一个将_reqwest_的错误转换为`LoginError`的一个函数。以上我总共用了两次`map_err()`和两次`ok_or()`，`map_err`用来将我在使用_reqwest_过程中遇到的`Error`转换为我自定义的`LoginError`，并使用`?`直接返回，节省了许多代码；`ok_or`两次都用在返回的结果是`Option<T>`的时候，我们不希望草率的用`unwrap()`来完成这样一个操作，我们新建了两个不同的错误来接受不存在的结果，并使用`?`直接返回。

以上两者如果全程使用`match`语法来操作，那有可能会是一个mess的火葬场，不信你试试。


## 参考链接

* [Define Error Type](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/define_error_type.html)
* [Boxing Errors](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/boxing_errors.html)
* [Wrapping Errors](https://doc.rust-lang.org/rust-by-example/error/multiple_error_types/wrap_error.html)

