---
title: jquery选择地区插件（选择省市区，有地区代码)
date: 2012-11-22 00:00:00
tags: JavaScript
---
项目地址：[https://github.com/zoowii/jquery-city-select](https://github.com/zoowii/jquery-city-select)

使用代码：

    $(function() {
        $.city_select($("#province_select"), $("#my_input"));
    });

    // html
    div#province_select
    input(type='text', id='my_input')


很简单的代码就可以产生一组选择省市地的控件，可以设置默认选项，也可以设置选择时把地区代码写入哪个input的value中。

效果如下：
![](http://zoowiistore.qiniudn.com/city_select%E6%88%AA%E5%9B%BE.png)
