+++
title = "使用JavaScript来判断质数"
date = 2018-12-17 12:50:32
[taxonomies]
tags = ["JavaScript", "Algorithm"]
categories = ["JavaScript"]

+++

## 关于

使用`JavaScript`判断一个数是否为质数
质数即是除了他本身更和`1`这两个公约数之外没有别的公约数的数。

## 代码

	'use strict';
	function priN(x){
    	if (x < 2) return false;
    	else
        	for(i=2;i*i<=x;i++){
            	if(x%i===0) return false;
        	}
    return true;
	}

## 总结

使用的方法即是从`2` 开始遍历，直到遍历完所有可能的情况为止。如果找到了公约数则返回`false`，否则返回`true` 。

__这里有一个值得注意的点是我们的遍历结束点是 `i*i<=x`__


