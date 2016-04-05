---
title: Java执行机制
date: 2013-01-16 00:01:20
tags: Java
---
Java 源码编译-----javac将java源码编译为class

java源文件(.java)=>分析和输入到符号表=>注解(annotation)处理=>语义分析和生成class文件

** class文件的内容 **

class文件的内容如下：

* 结构信息：class文件格式版本号等信息=>所以高版本的jdk生成的.class文件低版本的jre无法运行，反之可以运行
* 元数据：类继承、接口实现、属性和方法的声明、常量池等信息
*方法信息：Java源文件中语句、表达式对应的信息，包括：字节码、异常处理器表、求值栈与局部变量区大小、求值栈的类型记录、调试用符号信息

例子：

    // Hello.java
    public class Hello {
        public static void sayHi() {
            System.out.println("Hello World!");
        }
        public static int add(int x, int y) {
            return x + y;
        }
        public static void main(String[] args) {
            sayHi();
            System.out.println(add(4, 5));
        }
    }
    
使用javac -g Hello.java(使用javac Hello.java也可，但调试信息更少)产生Hello.class文件

再使用javap -c Hello可以反编译字节码，输出如下：

    // Compiled from "Hello.java"
    public class Hello {
    public Hello();
        Code:
           0: aload_0
           1: invokespecial #1
           // Method java/lang/Object."":()V
           4: return        
     
      public static void sayHi();
        Code:
           0: getstatic     #2
           // Field java/lang/System.out:Ljava/io/PrintStream;
           3: ldc           #3
           // String Hello World!
           5: invokevirtual #4
           // Method java/io/PrintStream.println:(Ljava/lang/String;)V
           8: return        
     
      public static int add(int, int);
        Code:
           0: iload_0
           1: iload_1
           2: iadd
           3: ireturn       
     
      public static void main(java.lang.String[]);
        Code:
           0: invokestatic  #5
           // Method sayHi:()V
           3: getstatic     #2
           // Field java/lang/System.out:Ljava/io/PrintStream;
           6: iconst_4
           7: iconst_5
           8: invokestatic  #6
           // Method add:(II)I
          11: invokevirtual #7
          // Method java/io/PrintStream.println:(I)V
          14: return
    }
    
** 类执行机制 **

* 字节码解释执行
    jvm有一套中间码，如invokestatic（调用static方法）、invokevirtual（调用对象实例方法）、invokeinterface（调用接口的方法）和invokespecial（调用private方法和方法）等。
    因此，jvm可以把这写中间码的指令解释执行，就像类似于脚本语言执行。只不过脚本语言执行的源文件是可读的文本文件，
    而jvm解释执行执行的是不可读的.class文件。
    
* 字节码编译执行
    解释执行效率低，所以jvm可以在运行时把.class文件编译成机器码执行，这就是JIT编译。
    所谓的Hotspot 就是执行时对执行频率高的代码编译执行，执行频率低的代码解释执行。