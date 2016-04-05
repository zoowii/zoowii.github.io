---
title: JavaScript的事件机制和执行队列的一个小验证
date: 2013-12-20 00:00:00
tags: JavaScript
---
今天突然想到，javascript的事件机制就是在事件发生后将事件的毁掉函数添加到代码的执行队列后面去，本身是单线程的。然后想到，同一个代码块的代码是一次性添加到执行队列中的，所以只要是定义回调之后的普通代码（非回调），都肯定在之前那个回调函数之前执行，无论这个回调函数对应的事件发生得多快，哪怕立刻发送，间隔时间为0，也肯定在定义这个回调的代码块的后续代码之后执行，因为那段代码块在之前已经被添加到执行队列中了。

间隔时间为0的回调很简单，setTimeout(function(){...}, 0)就可以了

为了验证这个猜测，我写了两段简单的JS代码试验了一下。

代码片段1：

    var state = null;
    
    function test() {
        // console.log('a1');
        if(!state || state==='a3') {
            state='a1';
        } else {
            throw new Error(state);
        }
        setTimeout(function() {
            // console.log('a3');
            if(state === 'a2') {
                state = 'a3';
            } else {
                throw new Error(state);
            }
        }, 0);
        
        if(state ==='a1') {
            state = 'a2';
        } else {
            throw new Error(state);
        }
        // console.log('a2');
    }
    
    test()




这段代码不报错，符合上面的说法


代码片段2（上面函数执行1000次）


    var state = null;
    
    function test() {
        // console.log('a1');
        if(!state || state==='a3') {
            state='a1';
        } else {
            throw new Error(state);
        }
    
        setTimeout(function() {
            // console.log('a3');
            if(state === 'a2') {
                state = 'a3';
            } else {
                throw new Error(state);
            }
        }, 0);
    
        if(state ==='a1') {
            state = 'a2';
        } else {
            throw new Error(state);
        }
    
        // console.log('a2');
    }
    
    console.log('start test');
    
    for(var i=0;i<1000;++i) {
        console.log("loop ", i);
        test();
    }
    
    console.log('end test');



因为整个for循环都是同一个代码段，在任何一次回调发生之前都被加载到执行队列中，所以setTimeout的任何一次回调都发生在1000次test函数执行之后。也就是全部1000次test函数执行完之前都不会执行任何一次setTimeout回调。然后在state变化时，第二次循环就应循环最开始变成a1状态时就应该报错error a2. 执行结果和预测完全符合。


这只是突然想到的一个简单的验证，也在吃证明了javascript是一个语言模型方面非常简单的语言，除了坑太多。
