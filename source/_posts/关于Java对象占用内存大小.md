---
title: 关于Java对象占用内存大小
date: 2012-12-19 00:00:00
tags: Java
---
    附加：以下内容很多错误，Java对象中还有很多其他内容， 比如指向类的指针，Flags（哈希值等），形状（是否是数组），如果是数组还有数组长度。后面才是正文内容。并且如果是String对象的话，String对象的正文部分是一个指向char[]的指针，这个char[]数组本身也有上面说的类型，Flags，形状，数组大小等基本信息和实际内容。并且在32bit和64bit的JVM上Java对象的基本结构占用内存也是不同的

昨天面试有一道题是关于Java内存占用的。题目是求出`new String("a") `和`Map map = new HashMap(); map.put("a", "a");`

回来之后我检测了一下，通过观察创建对象前后（创建大量一样的对象求均值）的已使用堆（`runtime.totalMemory() - runtime.freeMemory()`)内存大小的变化来计算Java对象的内存占用情况 。

代码是网上找来的：


    import java.util.HashMap;
    import java.util.Map;
    public class Test5 {
        public static void main(String[] args) throws Exception {
            // Warm up all classes/methods we will use
            runGC();
            usedMemory();
            // Array to keep strong references to allocated objects
            final int count = 100000;
            Object[] objects = new Object[count];
            long heap1 = 0;
            // Allocate count+1 objects, discard the first one
            for (int i = -1; i < count; ++i) {
                Object object = null;
                // Instantiate your data here and assign it to object
                // object = new Object();
                // object = new Integer (i);
                // object = new Long (i);
                // object = new String ("a");
                Map object1 = new HashMap();
                object1.put("a", "a");
                // object1.put(new String("a"), new String("a"));
                // object = new byte [128][1]
                if (i >= 0)
                    objects[i] = object1;
                else {
                    object1 = null; // Discard the warm up object
                    runGC();
                    heap1 = usedMemory(); // Take a before heap snapshot
                }
            }
            runGC();
            long heap2 = usedMemory(); // Take an after heap snapshot:
            final int size = Math.round(((float) (heap2 - heap1)) / count);
            System.out.println("'before' heap: " + heap1 + ", 'after' heap: "
                                + heap2);
            System.out.println("heap delta: " + (heap2 - heap1) + ", {"
                                + objects[0].getClass() + "} size = " + size + " bytes");
            for (int i = 0; i < count; ++i)
                objects[i] = null;
                objects = null;
            }
            
        private static void runGC() throws Exception {
            // It helps to call Runtime.gc()
            // using several method calls:
            for (int r = 0; r < 4; ++r)
                _runGC();
        }
            
        private static void _runGC() throws Exception {
            long usedMem1 = usedMemory(), usedMem2 = Long.MAX_VALUE;
            for (int i = 0; (usedMem1 < usedMem2) && (i < 500); ++i) {
                s_runtime.runFinalization();
                s_runtime.gc();
                Thread.currentThread().yield();
                usedMem2 = usedMem1;
                usedMem1 = usedMemory();
            }
        }
            
        private static long usedMemory() {
            return s_runtime.totalMemory() - s_runtime.freeMemory();
        }

        private static final Runtime s_runtime = Runtime.getRuntime();
    }
    

运行结果表明：

* new String("a")占用24Bytes,
* Map map = new HashMap(); map.put("a", "a"); 占用168Bytes, 
* Map map = new HashMap(); map.put(new String("a"), new String("a"));占用216Bytes.


原理：Java 的String 对象有四个属性：

    privatefinalcharvalue[];
    /** The offset is the first index of the storage that is used. */
    privatefinalintoffset;
    /** The count is the number of characters in the String. */
    privatefinalintcount;
    /** Cache the hash code for the string */
    private inthash;// Default to 0

再考虑Java对象的8字节对齐，java采用utf16编码，每个char 占用2个字节， 另外，java对象有两个字（每个字2字节） 的头部，分别表示垃圾收集、hash码信息以及指向对象的类，即占有4个字节。 24字节应该还是差不多的。

不过网上还有种说法是一个空字符串都占用40个字节，长为1的字符串也是占用40个字节，不清楚怎么算的，昨天面试官说的也是大概这个数据。

引用：[http://www.javaworld.com/javaworld/javatips/jw-javatip130.html?page=1](http://www.javaworld.com/javaworld/javatips/jw-javatip130.html?page=1)