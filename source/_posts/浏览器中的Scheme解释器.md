---
title: 浏览器中的Scheme解释器
date: 2012-09-28 00:00:00
tags: JavaScript, Scheme
---
前两天用javascript写了一个scheme解释器，已经可以运行在node.js和浏览器中。代码放在github上了。
[http://github.com/zoowii/SchemeScript](http://github.com/zoowii/SchemeScript)，

** Features **

* 另外，http://aboutzoowii.duapp.com/app/js/SchemeScript 中可以在线演示
* 支持自然数，布尔值(true, false)，字符串("...")，变量定义
* 可以定义函数了，并调用自定义函数，可以嵌套。示例中有求斐波那契数列的
* 有list, cons, car, cdr, list-len, n-th, typeof, +, -, *, /等函数
* 支持lambda表达式，柯里化
* 所有合法的标识符都可以被重定义，包括内置函数名
* 控制结构有if, cond[else]，都是作为内置函数实现的，可以赋给其他标识符，也可以重定义为其他值；cond控制结构的每一个条件分支都可以是多个form，执行时依次执行并返回最后一个form的值
* 定义变量用define，定义函数用defn，也可以用define+lambda表达式定义函数