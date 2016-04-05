---
title: 关于JavaScript的内存泄露检测
date: 2013-11-20 00:00:00
tags: Web
---
今天我遇到一个浏览器crash的问题，怀疑可能是JavaScript内存泄露了。然后网上搜了下，找到了Chrome中调试JavaScript内存泄露的方法

先打开Chrome开发者工具。以打开一个标签页为例。打开然后关闭此标签页一次，确保此标签页需要的资源都加载过了。然后进入开发者工具的Profiles标签页，选择Take Heap Snapshot，并Start。然后浏览器就会记录下当前页面的JavaScript所有对象的快照。然后再次打开关闭上述标签页，然后重复捕获JavaScript对象快照一次.此时可以看到如下图：

![](http://zoowiistore.qiniudn.com/zoowii.github.io_blog.20131025-0001.png)


注意到最下面有个Summary，选择Comparison，就可以看到Snapshots的增量变化了。内存泄露注意看Constructor中的closue的数量变化和内容就好，看看有多少对象没有释放，以及对应的代码。