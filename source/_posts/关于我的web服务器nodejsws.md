---
title: 关于我的web服务器nodejsws
date: 2012-09-27 00:00:00
tags: JavaScript, Web
---
之前，因为闲得无聊，用NodeJS写了一个web服务器，叫nodejsws(https://github.com/zoowii/nodejsws)


这几天把这个项目稍微做了一下，不过以前是以提供少量动态内容的静态资源服务器为目的的，现在编程了一个PHP的web服务器，已经成功的在nodejs上安装并运行wordpress了。
nodejs采用nodejs编写，使用php-cgi提供动态服务，不过因为不知道如何在php-cgi运行一次后不自动退出，又不打算使用php-fpm等开源项目（想自己造轮子），没有使用fastcgi，所以还是cgi模式，效率比较低，有空打算把nginx的fastcgi模块的源码看一下。另外nodejsws有空用其他语言改写，觉得nodejs和python一样，写写原型或者一些特定领域的应用比较好，其他的还是用其他语言吧。
nodejsws还有很多的BUG，有空再解决吧。暂时告一段落。