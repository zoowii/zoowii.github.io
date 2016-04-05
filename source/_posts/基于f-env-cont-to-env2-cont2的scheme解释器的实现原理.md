---
title: '基于f(env,cont)=(env2,cont2)的scheme解释器的实现原理'
date: 2014-01-24 00:00:00
tags: Scheme
---
    public class Scheme {
     public static (SEnv, SContinuation) f(env, cont) {
      // scheme解释器的逻辑
      if(cont.nextForm() is 加法) {
       return (env, cont执行nextForm()的加法操作后的结果)
      } else if(cont.nextForm() is 函数调用) { // 如果是形如(f (g 123))的调用，nextForm不是f调用，而是g调用
       env.down()
       put 函数调用g的参数到env中
       var newCont = 由cont生成，newCont指向cont中的nextForm()的函数调用体内部
      }
     }
     public static void main() {
      SEnv env = SEnv.init();
      SContinuation cont = SContinuation.initFromCode(code);
      while(cont.notEmpty()) {
       env, cont = f(env, cont)
      }
     }
    }

关于这个基于`f(env, cont)=(env2, cont2)`的计算模型用java实现的demo见[https://github.com/zoowii/lisp-continuation](https://github.com/zoowii/lisp-continuation)。注意，这个demo无法运行，因为没有完成，只是给出代码结构和主要代码，对一些细节没有完善。