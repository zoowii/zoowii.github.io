---
title: .Net字节码转换到Lua字节码的实现方式
date: 2017-04-13 00:03:21
tags: C#, .Net, Lua, VM, Compiler
---

最近做了个将.Net的字节码文件转换成Lua5.3官方版本字节码的小项目，这样用C#, F#等编程语言编写的代码就可以运行在Lua虚拟机上和Lua互调用了（虽然这种需求不常见，不过Lua虚拟机实现简单方便定制对运行时比较可控也算一个微不足道的优势）。
先不管为什么会有这个奇怪的需求，考虑到将性能差的动态编程语言编译到JVM, CLR等性能好的平台的字节码比较常见，但是反过来将静态编程语言编译到性能差的动态脚本语言的虚拟机上情况比较少见，所以记录这篇文章分享一下。

先简单介绍下.Net和Lua5.3的虚拟机的堆栈结构，因为这两种虚拟机的堆栈结构不一样，字节码指令也差别较大，不能简单对应转换。

.Net字节码的读取可以使用Mono.Cecil库，Lua字节码生成我是写了个assembler来根据伪汇编生成Lua5.3字节码。


.Net虚拟机有Evaluation Stack(指令下文简称eval stack), Call Stack, 参数区，堆，本质是一个基于堆栈的虚拟机
Lua5.3官方的虚拟机有Slots, 堆，本质是一个基于寄存器的虚拟机

首先因为两种虚拟机的堆栈结构都不一样，.Net的字节码指令基于操作栈和内存堆还有eval stack，而Lua的字节码指令都是操作Slots，类似操作一个个寄存器（不是CPU中的寄存器概念）的，所以要实现两种字节码的转换，我采用的方式是目标Lua字节码序列需要实现模拟.Net虚拟机的堆栈结构，下文有具体描述实现方式。


    主要实现流程：
    1. 实现.Net方法到Lua proto的映射
    2. 实现.Net操作指令到Lua指令的映射
    3. 实现.Net的控制流指令到Lua指令的映射
    4. 实现.Net的函数调用，函数参数，函数调用的返回值到Lua字节码的映射
    5. 实现.Net常用函数和类库到Lua的映射（只实现能关联到Lua中的函数和类库）
    6. .Net中提供Lua内置库的mock类，从而实现C#写代码，最终编译到Lua字节码后可以使用Lua内置库
    7. .Net中提供Lua的table类型等类型的内置库
    8. 精简生成的字节码指令，去除因为堆栈结构不一样导致的生成过程中一些无用操作（比如刚压栈就出栈）


将.Net的一个method映射成为Lua中的一个proto，同样有maxstacksize, numOfLocalVariables, 另外有numOfUpvalues。将.Net的非基本对象（数字，布尔类型，字符串等）映射为Lua的table类型对象和proto，.Net中一个有若干方法和属性的对象，映射为Lua中的table和proto, proto中有若干个slots分别通过Lua的closure指令指向不同的其他proto(.Net类型的methods映射的结果)，.Net对象的属性放入table中。

然后最关键的是实现.Net的method方法转换到Lua的proto的实现。

对于.Net虚拟机中的eval stack概念，可以通过在目标Lua字节码的proto中头部固定设置一个slot，存储一个table来模拟.Net中的eval stack，另外提供几个固定slot用来存放计算时临时用的对象（比如存储eval stack长度等)

对于.Net中的call stack概念，在目标Lua字节码的proto中专门开辟一段slot区域用来存储，这段slot区域的开始和结束位置，可以根据.Net中method的maxstacksize和指令操作的最大call stack loc计算出来。

然后就是.Net各种指令到Lua指令的翻译了，是一个苦力活，下面简单举例说下一些指令转换的方式


    每个proto头部先加入指令 newtable %0 0 0;  %0表示slot 0, @0表示upvalue 0, upvalue的处理有空再说

    以下不少翻译后指令结合起来后没必要可以去重精简掉


    .Net的IL指令(用一些基本常用指令介绍)                                                   Lua指令(简要描述)

    stloc 从eval stack 弹出栈顶数据复制到call stack                 len %1 %0; gettable %(callStackSlotStart+loc) %0 %1; loadnil %3 0; settable %0 %1 %3;
    ldloc 从当前函数栈的call stack把某个数据复制到eval stack         len %1 %0; add %1 %1 const 1; settable %0 %1 %(callStackSlotStart+loc)
    ldc_i4 加载4字节int常量到eval stack                             loadk %1 const ConstValue; len %2 %0; settable %0 %2 %1
    add    消耗eval stack的顶部2个值, 计算结果存入eval stack         len %1 %0; gettable %4 %0 %1; loadnil %3 0; settable %0 %1 %3; len %1 %0; gettable %5 %0 %1; settable %0 %1 %3;
                                                                             add %2 %4 %5; len %1 %0; add %1 %1 const 1; settable %0 %1 %2
    call   调用.Net函数                                             先从eval stack取出若干参数放入slot, 然后根据具体情况判断是转换成Lua指令还是全局函数还是某个table的成员函数，
                                                                   比如getupval %2 @(被call的func的upvalueIndex) / gettabup %2 @(env的upvalueIndex) const "函数名" 等，
                                                                   然后call %(func slot) 参数数量+1 返回值数量+1，如果method有返回值，还需要增加把返回值存入eval stack的指令
    br     无条件跳转到目标指令                                      分为跳转到当前指令之前的指令还是之后的指令，如果是之后的指令，需要预先计算出接下来若干.net指令转换到Lua指令后的数量计算偏移量，知道便宜量后就可以在                                                                   proto整体转换完成后在对应位置插入label，本指令转换为jmp $labelName
    beq, bgt, bge, blt, ble, bne   比较两个值(eval stacktop-1和top)，满足一定条件就jmp到目标指令                  len %1 %0; gettable %arg2 %0 %1; loadnil %3 0; settable %0 %1 %3; len %1 %0; 
                                                                     gettable %arg1 %0 %1; settable %0 %1 %3;lt 0 %arg1 %arg2; jmp 1 $labelOfNextIlInstruction; jmp 1 $targetLabel
    brtrue, brfalse 如果eval stack top值是1/0就jmp到目标指令           类似的条件跳转
    ret     结束当前函数栈并返回eval stack中的数据，需要根据当前.Net方法的返回类型是否void来判断返回数据
    newarr  创建一个空数组放入eval-stack顶                                   newtable替代
    newobj  创建一个空的未初始化对象放入eval-stack顶                          newtable替代
    dup     把eval stack栈顶元素复制一份到eval-stack顶                    len %1 %0; gettable %2 %0 %1; add %1 %1 const 1; settable %0 %1 %2
    cgt, clt, ceq   比较两个值(eval stack top-1 和 top)。如果第一个值大于第二个值(如果是Clt/Clt_Un，则是比较是否小于)，则将整数值 1 (int32) 推送到计算堆栈上；反之，将 0 (int32) 推送到计算堆栈上     这里也转换成条件跳转
    ldnull 加载null到eval stack
    ldstr  加载字符串常量到eval stack
    ldarg 从当前函数栈的参数列表中把某个参数复制到eval stack           
    等等 

以上描述得虽然还有内容很少，但是足够实现C#的大部分语法对应的字节码转换到Lua5.3字节码了，比如类型，函数定义，函数参数，函数调用和返回值，if, for, while, break, continue, 变量赋值，比较操作符，数值操作，内置函数操作比如字符串连接和print等。以后有空再补充更多细节