---
title: PHP实践经验总结一
date: 2014-02-05 00:00:00
tags: PHP
---
本文记录我在使用PHP的过程中的一些实践经验，主要做备忘用，因为我用PHP不常用，担心忘掉很多东西。如果某个主题比较复杂，会另开新文记录。(虽然PHP只作为偶尔用的语言,花时间应该不多)

* PHP文件，如无必要，PHP文件最末尾不要加上一个多余的?>，这主要为了避免在末尾偶然敲错键盘多敲了一些字符。如果加了?>，后面的支付会被当做普通文本输出，难以即时发现码入错误。这也是遵循PSR的建议。
* PHP作为一个实质上的模板引擎，当然不是只有<?php ?>这一种嵌入PHP代码的方式，因为作为一种模板引擎的话，每次写入<?php echo $hello; ?>太麻烦了，所以PHP支持一种简写的方式：`<?= $hello ?>`，这样就使用方便了很多，不过PHP默认不支持简写，需要在php.ini中修改short_open_tag=On，这样就支持了
* 建议使用composer管理PHP的包依赖，因为PHP本身的文件include, require的方式太不优雅了，不利于管理代码结构，所以如果不是使用一些开源完整的MVC/CMS框架，而是自己从头开始写的话，建议使用composer + PHP namespace管理代码结构，在这个基础上去使用一些MVC框架，比如joomla framework(这是一个MVC框架，不是Joomla CMS系统)。
* 推荐一个PHP解析HTML文本的库，[http://simplehtmldom.sourceforge.net/](http://simplehtmldom.sourceforge.net/)，非常好用，在我用PHP简单爬取一些网页内容的时候帮助很大。
* 因为nginx的rewrite规则和apache2的rewrite规则不一样，而很多目前已有的系统中大多支持apache 的rewrite规则（使用.htaccess文件），所以建议依然使用apache2作为PHP网站的服务器，另外nginx作为前端反向代理以及server静态资源。
* 推荐PHP网站的静态资源固定放在一些文件夹下，比如/static, /media，使得代码结构类似于：/static/js, /static/css, /static/img, /media这样，这样的话，在nginx中可以只需要提供一两个规则提供对/static, /media目录下的访问，将来需要增加一些比如/static/fonts目录的时候也不用改web服务器的配置，并且通过PHP代码中提供一些例如static_url这类的函数来动态生成静态文件的URL的话，可以很容易在本地静态文件和CDN网络文件切换。而很多常见的直接使用/js, /css, /img的网站文件结构这点上我觉得不太好。
* 不要用PHP提供websocket, 长连接，comet等服务，PHP这种每个请求一个进程（线程）的方式（即使用了FastCGI），承担这类应用不太合适。
* 阻塞较长时间的服务，尽量不要在一个请求中完成，建议用消息队列提交到后台程序处理，或者用curl异步提交，处理完成后触发一个url通知完成。
