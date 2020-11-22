+++
title = "若干数最小公倍数求法[JavaScript实现]" 
date = 2019-03-04 19:30:12
[taxonomies]
tags = ["javascript", "algorithm"]
categories = ["JavaScript"]
		
+++

## 关于

假设有这么一道题目：

> 求给定任意个整数的最小公倍数。

**你会怎么做？**

我们设想给定一个参数为数组，数组中包含我们要求最小公倍数的最小整数，返回一个值为这若干数的最小公倍数的整数。

**我建议你先自己想一想**

是的，我在学习的过程中遇到了这样一个题目，并且苦思冥想了好些时间，终于让我给找到一个方法。接下来我将来阐述我解答这道题的思路。总共包含一下几个部分：

* 最大公约数算法
* 若干数最小公倍数思路
* 具体JavaScript实现

## 最大公约数算法

学计算机的同学都知道有这样一个算法，名字叫做欧几里得算法，来自古希腊的数学家欧几里得，又叫做辗转相除法。对于学习过的同学，我想你已经可以不必要看这一小节了。关于欧几里得算法它是这样描述的：

> 两个整数的最大公约数等于其中较小的那个数和两数相除余数的最大公约数。最大公约数（Greatest Common Divisor）缩写为GCD。

这样简洁的描述，让我们来JavaScript实现它吧。

```Javascript
function gcd(p,q){
	if(q===0){
		return p;
	}
	return gcd(q,p%q);
}
```

为什么会用到最大公约数的算法呢，接下来的思路里解释。

## 若干数最小公倍数思路

我们求几个数的最小公倍数，最简单的方法当然是把这几个数直接相乘。但是我们要求的是最小公倍数，相乘得到的不一定是最小公倍数，数与数之间可能还有约数，相乘的结果与最小公倍数的差别可能就是多乘了一个两个约数。如果我们所有的约数都除掉的话是不是就是最小的公倍数了呢。

答案是肯定的，这就是为什么我们需要用到 _欧几里得算法_。

所以很容易地我有了第一个想法，就是先把所有结果都相乘然后再除以所有数的最大公约数。这个想法被验证是错误的，我们我得到的是相乘的结果不是相加的结果，不应该除以所有数的最大公因数，相乘所有数与数之间的公约数都会乘进去。

第二个想法就是，我对这个数组遍历，然后对每一个数除以它前面所有数的最大公约数，得到一个新的数组，我把新数组中的所有数都乘起来，就是这几个数的最小公倍数了。这个方法是先把公约数去除掉，然后在相乘。相较于之前的方法在算法的复杂度上可能有所降低。

是的这个方法没有证明，我也无从知晓别处是否有这样的方法，我判断它是否有效的办法就是先用代码来实现它。对我来说这确实是一个没有被证明的算法，希望各位看观能够提提意见，如有错误还请指正。

## 具体JavaScript实现

接下来还请看我的代码实现。

```JavaScript
function smallestCommons(arr){
	function gcd(p,q){
		if(q===0){
			return p;
		}
		return gcd(q,p%q);
	}
	for(var i=0;i<arr.length;i++){
        for(var j=0;j<i;j++){
            arr[j] = arr[i]/gcd(arr[i],arr[j]);
        }
    }
	return arr.reduce((x,y)=>{
		return x*y;
	});

}
```
以上就是我实现的代码，如果有错误，欢迎在评论区指正，谢谢！


**********************

参考链接：

[最小公倍数算法挑战](https://www.w3cschool.cn/codecamp/smallest-common-multiple.html)