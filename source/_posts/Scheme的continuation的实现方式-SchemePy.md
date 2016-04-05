---
title: Scheme的continuation的实现方式(SchemePy)
date: 2014-01-23 00:00:00
tags: Scheme
---
在Lisp的一些方言中有一种特别的东西，叫做continuation，尤其是在Scheme语言中。

不过其实continuation也不是lisp特有的东西，其他语言中也有本质上相同或类似的东西，也许不叫一样的名字。只不过scheme中直接把continuation的控制权限交给coder自己，能够由程序员自己控制它。

然后，本文说的是怎么实现continuation的，首先要给出continuation的定义。

以状态机的模型定义计算机的话，可以把一个计算机看做是下面这个函数
f(S) = S, 其中S是计算机状态的集合，f就是计算机的程序逻辑

再具体一点，具体到scheme的执行中去，状态集合S就是scheme的上下文环境（env），接下来要执行的程序（continuation），词法作用域（非必须，以下不考虑）

上下文env就是一个记录名称key=>值的记录，这个值可以是基本数值，也可以是一块内存的指针（地址），不过这不影响本文主题，以下直接把所有env中的值看做是对象的应用，不考虑内存的影响增加复杂度。同时env要有一个记录上一层级的指针，指向另一个env。所以上下文env的结构大致如下（一个list结构，列表的一个节点是一个map对象）：

                                                   -> parent = null
                                                   ->key3 => value3
                                                   ->key2 => value2
			- parent(default = null) ----------------------------->- -key1 => value1
			-  key1=> value1
			- key2 => value2
			--<-----------------------------------------另一个env


* continuation就是保存下接下来要执行的代码
* f 就是一个循环，不断接受env和continuation，执行固定的逻辑，返回结果重新再次执行

然后上面的f(S) = S就相当于
`f(env, continuation) = (env2, continuation2)`

这个式子应用到图灵计算机中就是：

* env就是内存中的数据区
* continuation就是内存中的指令序列，和PC计数器
* f就是图灵计算机中从指令序列中按PC计数器读取下一条指令，在CPU中执行，对内存数据进行操作（得到env2），然后PC计数器+1(得到continuation2)



简便起见，用一段简单的scheme代码描述上面过程

给一个continuation的大概结构先：

	{
		code_list: ..., 一个scheme的列表，运行时可能会动态修改
		current: code_list中的当前位置的ID（当前位置可以是code_list中某个元素，或者一个list中的某些位置）
	}
	
看代码：

	(define a 100)
	(define (fib n)
  	(if (< n 3)
      1
      (+ (fib (- n 1)
         (fib (- n 2)))))
	(display (fib a))

解释执行时，最开始的状态是`env= {}, continuation={code_list: ..., current: null}`

然后开始运行后，比如运行到(define a 100)时，执行之前continuation的current指向这个列表，执行则修改env，并且将current指向下一个位置，结束一次执行。

至于scheme中的call/cc，因为continuation也是一个scheme对象，调用call/cc时把这个对象作为参数传递给调用call/cc时的参数（一个函数g），在g的执行过程中，如果调用了这个continuation(命名cont），则把这个cont替换掉当前的continuation并继续执行

先睡了，下次有空再说详细点

PS:
1. 关于这个基于f(env, cont)=(env2, cont2)的计算模型用java实现的demo见[https://github.com/zoowii/lisp-continuation](https://github.com/zoowii/lisp-continuation)。注意，这个demo无法运行，因为没有完成，只是给出代码结构和主要代码，对一些细节没有完善。
