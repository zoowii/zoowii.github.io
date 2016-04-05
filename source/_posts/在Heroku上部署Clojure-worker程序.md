---
title: 在Heroku上部署Clojure worker程序
date: 2014-04-08 00:00:00
tags: Clojure
---
前几天想起Heroku上可以有worker程序，可以部署在后台一直运行的程序，而不是只可以部署web网站，真好VPS上跑的程序已经够多了，就把一个Clojure写的小程序部署上去了。

在部署这个Clojure程序中，有一些要注意的问题，特此记录。


首先就是，再Clojure项目的Leiningen的project.clj中，需要加入:main 指向起始函数-main所在的Clojure namespace，同时这个clojure namespace需要在ns定义处加上:gen-class限定(起始不一定是-main函数，可以给:gen-class传递参数指定函数前缀，默认是'-'，会对应生成的Java类中的不带前缀的方法)。其实这不属于和Heroku部署特定相关的，而是所有Leiningen项目打包成可启动jar程序都要做的事。

第二点是，再project.clj中显示指定:uberjar-name指定打包成的jar包名称，方便在Procfile中指定路径，另外还要显示指定:min-lein-version 为"2.0.0"（或者更高），这是因为我使用了Lein 2，而Heroku默认的Lein版本不是，不指定会启动失败

第三点，我开始也没发现错误原因，是再看heroku logs的时候才发现的，直接按上面步骤再git push部署后，heroku logs中会报版本错误，这是因为Heroku现在默认是使用Java 6，而我项目中依赖Java 7，所以也需要显示指定Java版本，返回就是在项目根目录下新建一个名为system.properties的文本文件，内容是 java.runtime.version=1.7


到这一步就差不多了，然后就可以git push部署了。

之后还有一个小问题，也许你会发现git push后的程序无法执行，这是因为再Heroku Apps中的你这个应用的配置还是使用的web role数量为1，worker role数量为0，没有worker role当然无法运行了。所以只需要再Heroku管理界面中将web role设为0，worker role设为1即可。


完成。

感慨一下，Heroku实在是一个很好的PaaS平台，开发和部署都非常方便，文档也很不错，也不像GAE和SAE那样限制多多。最赞的是支持worker role。

可惜在天朝速度实在太慢，好在worker role不在乎这点
