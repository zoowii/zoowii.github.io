---
title: Continuation的树结构与更新遍历查找算法
date: 2014-03-05 00:00:00
tags: Scheme
---
 最近受朋友所托用Python把之前的`f(env, cont)=(env2, cont2)`的Scheme实现模型实现一下。在实现Continuation的数据结构时，因为没有进行CPS变换，所以需要找到下一个需要执行的form（这个form需要是list）作为调用continuation的next()的返回结果的一部分，然后这个form就是下一个要执行的form了。其实这个不断找到下一个要执行的form，剩下的作为一个新的cont的过程，和CPS变换是一样的，都是找到当前要执行的过程，以及剩下要执行的cont，把后者当成前者类似回调一样的东西。这点就不多说了，本文说一下在实现这个cont的next()函数时，continuation的数据结构是怎样的。

continuation大致是一个list，包含了很多元素，每个元素都可能是列表，然后可以嵌套包含，则也就是普通的lisp程序的结构（或者说是语法树）。同时continuation需要有一个position属性记录当前的位置，一个require属性（如果不为空的话）记录某个位置需要一个值来替换（比如call/cc时经常发生这个现象）。按照上文说的，continuation需要一个next()函数找到下一个要执行的form。既然是下一个要执行的，这个form就需要是元素都不是list(否则下一个要执行的就不是它而是它的那个列表的元素了)。也就是说form始终是一个list，其元素都不是list。而position就是记录上一个这样的form的位置，next()函数就是找到下一个这样的form的位置。

说到这里很明显了，满满的即视感，树结构已经很明显了。next()函数返回的form就是非叶子节点中最低层次，所有子节点都是叶子节点的节点了。而next()函数就是在这样的目标节点中遍历。

然后现在实现continuation就变成实现一个满足这个要求的多叉树而已了

可以用一个索引数组记录position的值（require也可以这样记录），然后找到next()的目标节点（也就是新的position）也不难了。

具体的以后再说。
