---
title: PHP获取payload
date: 2013-08-28 00:00:00
tags: PHP, Web
---
PHP 中的$_POST只是在POST请求的Content-Type为url-encoded-form-.... 时的内容，但如果不是这个Content-Type，一般请求内容是在HTTP请求的BODY（request payload）中（不一定，HTTP请求是可以构造的）。而用$_POST读取这个payload内容就不行了，因为这个BODY内容是在HTTP请求的BODY中，类似于标准输入（事实是不同的），需要用file_get_contents('php://input')读取