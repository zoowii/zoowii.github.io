---
title: 程序语言的词法作用域中同名变量的rename
date: 2015-04-06 00:00:00
tags: 算法
---
TODO 

关于在实现程序语言中,如何实现对不同词法作用域中同名变量的rename操作,从而下一步操作更加方便.

比如把如下代码

```
function func1() {
    var name = "zoowii";
    function func2(name) {
        return "Hello, " + name;
    }
}
```
自动修改为
```
function func1() {
    var name = "zoowii";
    function func2(name_unique_2) {
        return "Hello, " + name_unique_2;
    }
}
```


先记下来,有空再写
