+++
title = "尝试rust ffi，rust调用clang"
date = 2019-09-02 10:53:54
[taxonomies]
tags = ["rust", "ffi"]
categories = ["Rust"]

+++

最近试图想用rust做一个小工具，但是rust上目前还没有对应的库，而C语言中有对应的库。想尝试使用这个库，我需要先了解一下rust ffi（Foreign Function Interface）。

要想将rust的ffi应用到一整个库上，首先我们先从一个文件做起。

这篇文章就是对rust ffi 一次小小尝试的记录。

<!--more-->

## 编写一个c，并编译为库

我们在一个新建的rust项目的src目录下，新建一个`add.c`。

```c
//src/add.c
int add(int a, int b) {
    return a+b;
}
```
在命令行中执行：
```shell
clang -c add.c -o libadd.a
```
将我们新建的c文件编译为一个可以被引入的库文件。现在我们`src`目录下结构就是这样的:

```file
- src
    - add.c
    - libadd.a
    - main.rs
```

## 编写main.rs

我们如果想要在`main.rs`中调用我们在`add.c`中定义的add函数，我们需要将它引入，并且在`unsafe`代码块中使用。
```rust
use std::os::raw::c_int;

extern {
    fn add(a, c_int, b: c_int) -> c_int;
}

fn main() {
    let result = unsafe { add(1, 2) };
    println!("1 + 2 = {}"，result);
}
```

## 编译main.rs

也许这个时候，你会试图想要用`cargo run`来运行你的代码。但是这是行不通的，会报`undefined reference`错误。因为cargo并不知道我们，我们需要链接的库文件在哪里。

试一试这个：
```shell
rustc main.rs -L. -ladd && ./main
```
你可以看到输出了：
```
1 + 2 = 3
```
我们可以使用`rustc --help`查看`-L`和`-l` flag分别代表着什么。
* `-L` 添加lib搜寻的目录
* `-l` 添加lib

## 使用cargo 编译

想要用`cargo`编译，需要在项目根目录下新建一个`build.rs`

cargo 会读取`build.rs`的输出，来设置我们编译时的配置

在这里我们最小化的一个main.rs就是长这样的
```rust
//build.rs
fn main() {
    println!("cargo:rustc-link-search=./src");
    println!("cargo:rustc-link-lib=add");
}
```
具体更多的可以被`cargo`识别的配置可以在这里找到：[build script](https://doc.rust-lang.org/cargo/reference/build-scripts.html)


## 参考

* [Linking and Building](https://s3.amazonaws.com/temp.michaelfbryan.com/linking/index.html)
* [Build Script](https://doc.rust-lang.org/cargo/reference/build-scripts.html)
