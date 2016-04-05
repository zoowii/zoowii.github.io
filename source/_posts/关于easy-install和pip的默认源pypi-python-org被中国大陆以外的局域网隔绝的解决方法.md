---
title: 关于easy_install和pip的默认源pypi.python.org被中国大陆以外的局域网隔绝的解决方法
date: 2013-12-29 00:00:00
tags: Python
---
最近在一台windows PC上配置python的环境时发现下载setuptools官方的ez_install.py，并且执行python ez_install.py install安装失败，无法下载。因为有在其他反面这种情况的多次经验，知道是国外的局域网不给力，连接不上中国大陆互联网。

以前我在其他电脑上都是为了加快pip安装速度，使用国内镜像安装，但是现在的问题是默认安装python后连easy_install 和pip都没有，需要先安装它们，而安装它们，如果使用ez_install.py的话网络又不给力，所以要麻烦一点。

首先，去setuptools官网，或者 http://pypi.douban.com/packages/source/s/setuptools/ 这个链接下载最新的setuptools源码，解压后，执行python setup.py install安装。这时候可以发现Python的安装目录中多了一个Scripts文件夹，把这个文件夹加入PATH路径中。

安装好之后，我们发现Scripts文件夹中只有easy_install，而没有pip。而我用pip更习惯。并且easy_install 安装包时也有访问不了pypi源的问题。

`easy_install --help`一下，可以看到有一个-i (--index-url)的选项，这就是指定pypi源的地方了。使用豆瓣的源试一下
`easy_install pip -i http://pypi.douban.com/simple` 安装成功，并且速度很快。

然后pip也可以这样安装各种Python库了，比如

`pip install flask -i http://pypi.douban.com/simple`

但是每次这么指定比较麻烦，可以在%HOME%目录下创建一个.pip的目录，里面创建一个pip.conf的文件，里面假如以下内容：

    [global]
    index-url = http://pypi.douban.com/simple/

接下来就可以直接pip install 库名称 或者依然 `pip install 库名称 -i http://pypi.douban.com/simple` 来安装Python库了

另外，github上可以找到一个shadowsocks的工具，原本是Python实现的，不过后来有了很多其他语言的实现比如golang, nodejs等。这个工具可以使用加密的方式访问国外局域网，避免我们的信息暴露出去，并且也可以访问一下原本国外速度很慢的网站
