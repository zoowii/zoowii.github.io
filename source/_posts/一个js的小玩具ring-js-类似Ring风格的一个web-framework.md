---
title: '一个js的小玩具ring.js,类似Ring风格的一个web-framework'
date: 2014-11-10 00:00:00
tags: JavaScript
---
前段时间一时兴起,弄了个新玩具,ring.js, 项目地址[https://github.com/zoowii/ring.js](https://github.com/zoowii/ring.js)

这是一个非常简陋的类似于Clojure的Ring库的node.js框架

## 使用Demo

```
function helloHandler(req, name) {
    return 'hello, ' + name;
}

function fileTestHandler(req, path) {
    return new ring.FileStream('/Users/zoowii/SystemVersion.plist');
}

app = ringMiddlewares.routesMiddleware(app, defroutes(
    GET('/', 'index', 'index'),
    GET('/hello/:name', helloHandler),
    GET("/test/:id/update", "test-handler", "test"),
    GET('/test/file/:*path', fileTestHandler, 'file-test'),
    context("/user", [
        GET("/:id/view/:project/:*path", "view_user_handler", "view_user"),
        POST("/:id/view/:project/:*path", "update_user_handler", "update_user")
    ]),
    ANY("/:*path", '404-handler', '404')
));
app = ringMiddlewares.resourceMiddleware(app, '/static', __dirname);
var server = httpAdapter(app, {
    port: 3000
});
server.start(function () {
    console.log('listening at http://127.0.0.1:3000');
});
```

## ring.js概念

ring.js包括handler, middleware, adapter, request, response这些对象的概念

### handler

handler是一个接受request返回response的函数,整个ring.js的webapp就是一个handler

### middleware

middleware是一个接受若干个handlers和options配置的函数,返回一个新的handler

### adapter

adapter封装底层http库等协议,在adapter上可以运行ring.js的webapp,也就是handler. adapter是一个接受一个handler(暴露的ring.js webapp)和options的函数,options可以包括address, port等信息 当一个新请求到来时,adapter封装这个请求成一个request,交给参数中的handler,然后把返回的response作为http回复

### request

request代表一个http请求,是一个javascript object,结构如下:

```
{
  server_port: (required, int),
  upgrade: (required, boolean), // true/false
  server_name: (required, string),
  remote_addr: (required, string), // 请求发送方或者最后一层代理的地址
  uri: (required, string), // 请求uri, 以'/'开头
  query_string: (optional, map),
  scheme: (required, string), // 传输协议,目前支持http和https, 转换成小写
  http_version: (optional, string), // http协议版本,比如1.1
  request_method: (required, string), // 请求方法, 转换成小写,包括get, post, head, put, delete等
  content_type: (optional, string),
  content_length: (optional, int),
  character_encoding: (optional, string), // 用来转换request body的编码
  ssl_client_cert: (optional, byte[]), // 客户端SSL证书
  headers: (required, map), // http头信息, header name都转换成小写
  body: (optional, stream) // http请求body
}
```

### response

response代表一个http回复,是一个javascript object,或者是一个future对象,如果response是一个map,结构如下:

```
{
  status: (required, int), // >= 100, http status code
  headers: (required, map), // http回复的头信息
  body: (optional, string/其他对象的array/iterator/file/stream/上面对象的future, // http回复的body
}
```

如果response是一个future对象,则监听这个future对象的data事件和end事件,获取到map形式的response chunk并输出
