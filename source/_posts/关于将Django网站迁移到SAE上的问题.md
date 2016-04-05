---
title: 关于将Django网站迁移到SAE上的问题
date: 2014-01-17 00:00:00
tags: Python, Django
---
因为个人算是业余股民，偶尔会需要看看股市行情，但有懒得打开客户端，有的电脑也没装。晚上直接搜也不好查看个性化信息比如自选股什么的，所以之前有做过一个很简单的股票查看网站，只有自己一个人使用。使用Python3+Django开发，用requests抓取数据，使用gunicorn+sqlite部署到自己的VPS上，虽然目前还是0.0.1版，开发出第一个版本后就没改过了，不过勉强够自己使用了。

不过因为自己的VPS是在美国纽约，速度比较慢，所以考虑到速度的原因，想迁移到国内的云服务上，既省得自己管理，便宜还速度快。

所以今天晚上做了下迁移，首先是使用python2.7替换python3，这个很顺利，除了有几个文件有中午注释，文件开头加上# coding: UTF-8就好了。然后把sqlite转成mysql，python manage.py syncdb再用navicat神器把有数据的业务表迁移过去，导出sql，导入sae phpmyadmin就好了。数据库和python环境迁移成功。

然后是代码了， 因为使用了django1.6 ，所以要手动拷贝sae中没有的库到一个lib文件夹下,
`os.path.insert(0, lib文件夹路径)`即可。

django settings中的修改，因为使用了dj_database_url，所以在index.wsgi中用sae.const中的mysql的用户名密码等信息给`os.environ['DATABASE_URL']`赋值即可。

差不多了，上传代码运行，然后报错，在调用requests库时，报conn.sock.settimeout(...)中conn.sock为None。网上一搜是2013年后半年对urllib3库的一次修改造成的BUG，[https://github.com/kennethreitz/requests/blame/master/requests/packages/urllib3/connectionpool.py#L307](https://github.com/kennethreitz/requests/blame/master/requests/packages/urllib3/connectionpool.py#L307)，把第307, 308行的

    else:
        conn.sock.settimeout(read_timeout)

中的"else:"改成"elif conn.sock: "即可。

再次运行，成功。

网址：[http://stockkit.sinaapp.com/](http://stockkit.sinaapp.com/)

不过说一句，SAE的静态文件加载太慢了啊，都是几百毫秒的级别，虽然还是比美国VPS快，但还是快得有限啊。
