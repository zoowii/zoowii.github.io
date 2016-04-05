---
title: PHP与Java中运行时获取调用类方法（静态方法）的类名的方法
date: 2013-02-20 00:02:57
tags: PHP, Java
---
因为看不惯FengOffice中的ORM框架（好像是Zend中的，不过没用过Zend，不清楚），觉得太繁琐，所以打算自己用PHP实现一个轻量级，方便，尤其要容易写Model和容易使用。

参考了Yii的orm框架的使用方式，或者说是ActiveRecord的使用方式，写了一个简单的orm，在实现的时候因为要在父类Model类中的静态方法中，动态获取调用它的是哪个类，从而知道使用的是Model的哪个子类。刚开始不清楚怎样获得，后来在一位学弟的提醒下知道了有get_called_class()这个方法可以动态获取调用者是哪个类，从而解决了这个问题。

果然，百度是百度不出什么东西的啊。这个orm框架在稍作完善后再放到github(zoowii)上吧，先不管这个。

因为学校学的主要语言是Java，所以想到Java中怎么解决这个问题，经过百度（因为十八大，谷哥不方便），找到了一篇帖子：
[http://hi.baidu.com/kittopang/item/a04c9ed12ff32aefb2f77711](http://hi.baidu.com/kittopang/item/a04c9ed12ff32aefb2f77711)

因为这篇帖子中的情况是获取当前类，而不是运行时使用父类的静态方法，在这个父类的静态方法中动态获取它的调用者的类名，两者情况还是不一样的。所以我写了点Java代码测试了一下。

    // Test3.java
    public class Test3 {
      // output the current class name of the source file, so always output Test3
      public static void f1() {
        String className = new Object() {
          public String getClassName() {
            String nm = this.getClass().getName();
            return nm.substring(0, nm.lastIndexOf(‘$’));
          }
        }.getClassName();
        System.out.println(className);
      }
     
      // also output Test3
      public static void f2() {
        String className = new SecurityManager() {
          public String getClassName() {
            return getClassContext()[1].getName();
          }
        }.getClassName();
        System.out.println(className);
      }
     
      public static void f3() {
        String clsName = new Throwable().getStackTrace()[1].getClassName();
        System.out.println(clsName);
      }
     
    }
    // Test4.java
    public class Test4 extends Test3 {
      public static void main(String[] args) {
        f1();
        f2();
        f3();
      }
    }
    
编译后执行java Test4后得到输出结果:

    Test3
    
    Test3
    
    Test4
    
可见，只有第三种方法可以达到目的，也就是通过新建一个Throwable对象，调用getStackTrace()获取调用栈信息，对其中的第二个对象调用getClassName()方法。这种通过异常栈的方法获取信息的方法虽然能够达到目的，而且可以获取的信息很多，但到底不优雅，而且性能很差，但在不知道其他解决方案的情况下也只能这样了。如果有人知道其他方法的，请告诉我，谢谢。