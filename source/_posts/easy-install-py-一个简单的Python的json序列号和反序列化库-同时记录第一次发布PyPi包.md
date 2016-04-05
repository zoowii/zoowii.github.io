---
title: easy_install.py, 一个简单的Python的json序列号和反序列化库, 同时记录第一次发布PyPi包
date: 2014-03-03 00:00:00
tags: Python
---
这两天因为一个很小的项目，又把有段时间没用的Django捡起来了

虽然Django让很多有洁癖的Pythonist很不爽，但是Django最大的优点也差不多是仅剩的优点就是强大的管理后台自动生成功能。对一些web前端界面不重要甚至根本没有的项目，这个优点很好很强大。

因为需求关系，需要将Django数据库查询的结果json化输出，所以又翻了下Python的json库。

不过因为Django的ORM是紧耦合的，使用Django自带的django.core.serializers来序列化产生的格式也不适合，找了一下也没找到方便自定义的方法。使用Python的json库使用起来也繁琐不方便，需要定义一个有点复杂的serializer函数并且和Django的ORM一起用有点不适应。也因为我的需求比较简单，觉得做一个满足自己需求的json包是很简单的事情，所以就动笔写了。

项目名称easy_json.py

写好后其实没几行代码，但是在我的使用场景中比json.dumps和Django自带的json库方便太多了，所以打算放到PyPi上共享。其实这才是本文的主题，记录我第一次发布Python包到PyPi上 :)

根据网上找的和我自己实践的结果，需要先在项目目录下新建一个setup.py文件，里面的结构大致如下：


    from setuptools import setup, find_packages
    
    setup(
        name='easy_json',
        version='0.0.1',
        keywords=('json', 'easy_json', 'simple', 'Django'),
        description='a python json serialize/deserialize library which is easy to use',
        license='MIT License',
        install_requires=[],
        author='zoowii',
        author_email='1992zhouwei@gmail.com',
        packages=find_packages(),
        platforms='any',
    )

注意不能随便增加属性，否则很可能验证失败无法上传到PyPi上。

然后可以到[https://pypi.python.org/pypi](https://pypi.python.org/pypi) 右侧的注册链接注册一个PyPi帐号，注意密码要求是16位以上，反正密码弄复杂点就没错了。

然后terminal中cd 进入项目目录。执行 python setup.py sdist  进行打包。

然后执行`python setup.py register`进行登录，会让你选择是使用现有帐号还是新建帐号

然后执行`python setup.py upload`，等一下就上传成功了。

其实过程非常简单，只不过记录一下以纪念第一次发布PyPi包而已。

** 附：**

* Github 项目地址：[https://github.com/zoowii/easy_json.py](https://github.com/zoowii/easy_json.py)
* PyPi 项目地址：[https://pypi.python.org/pypi/easy_json](https://pypi.python.org/pypi/easy_json)
