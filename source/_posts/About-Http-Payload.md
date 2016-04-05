---
title: About-Http-Payload
date: 2013-11-12 00:00:00
tags: Web
---
最近看了下REST协议，然后试验了下，发现Rest发出的POST请求的内容是存放在Http请求的Request Payload里的，而不是像平常在form, jquery post中是存放在post data里的。如果要读取这个Payload的内容，可以把request对象的输入流的内容（正文部分）读出，根据需要转成JSON/XML或其他需要的格式。

网上查了下资料，之所以会有这样的问题，是因为RESST协议规范了Content-Type，服务器端会（或者说应该）根据Content-Type的内容做相应的处理，比如如果Content-Type是text/xml; charset=utf8或者是text/json或application/json等等，就分别做xml/json处理。

可是平常的form表单的Content-Type是application/x-www-form-urlencoded或者multipart上传文件的。参数是经过urlencoded成类似name=zoowii&age=21的形式，而jquery的ajax请求已经帮我们做了设置Content-Type和urlencode参数的工作。Content-Type=application/x-www-form-urlencoded的参数内容是放在request body中的，也就是我们平常用的请求，一般服务器端可以直接从request中读出这个request body的内容。

要解决这个方法也简单，只要写一个实用函数可以根据Content-Type类型预处理HttpRequest并将request payload预处理和解码就好了。

另外，REST确实是一个很方便的协议。
