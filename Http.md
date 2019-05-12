# HTTP 学习笔记

主要特点
- 无连接(通信结束就断开)
- 无状态(服务器不会记录客户端的状态)
- 简单快速(一个地址固定对应一个资源)
- 灵活(可以传输不同类型数据)

## 报文
请求报文

- 请求行：请求方法 URL 协议/版本号，GET /learn/list/129.html HTTP/1.1
- 请求头：kye/value
- 空行
- 请求体 JSON 等等

响应报告

- 响应行：协议/版本 状态码 状态短语, HTTP/1.1 200 OK 
- 响应头：key:value
- 空行
- 响应体(例如一个html)


## POST 与 GET 的区别
- GET 参数通过 URL 传递，POST 放在 Request body 中
- GET 在浏览器回退时的无害的，而 POST 会两次提交请求
- GET 请求会被浏览器主动缓存，而 POST 不可以，除非手动设置
- GET 请求参数会被完整保留在浏览器历史记录里，而 POST 中的参数不会保留
- GET 请求在 URL 中传递的参数是有长度限制的(一般2KB,不同浏览器不同)，而 POST 没有限制

状态码
- 1xx: 指示信息
- 2xx：成功
- 3xx：重定向
- 4xx：客户端错误，请求有语法错误或请求无法实现
- 5xx：服务器错误：服务器未能实现合法的请求

## 持久连接(keep-alive)
使用普通模式时，每请求一个资源都要重新建立一次连接（HTTP无连接）。keep-alive是指服务器和客户端的多个请求响应共用一个TCP连接，而不是每次请求响应都新建一个连接完了后关闭
。HTTP 1.1 才开始支持持久连接。

## 管线化
不开启管线化，资源会线性地请求响应。开启之后，客户端可以一次性发生多个请求。

- 管线化机制通过持久连接完成，仅 HTTP/1.1 支持此技术
- 只有 GET 和 HEAD 请求可以进行管线化，而 POST 则有所限制
- 初次创建连接时不应该启动管线化，因为服务器不一定支持 HTTP/1.1

## TCP
[三次握手&四次挥手](https://www.jianshu.com/p/9968b16b607e)

## HTTP2
在 HTTP/1.x 中，此元数据始终以纯文本形式发送

新特性
- 二进制传输，HTTP 2.0 中所有加强性能的核心点在于此。在之前的 HTTP 版本中，我们是通过文本的方式传输数据。在 HTTP 2.0 中引入了新的编码机制，所有传输的数据都会被分割，并采用二进制格式编码。
- 多路复用，避免 HTTP 1.x 的队头阻塞

    HTTP/2复用TCP连接则不同，虽然依然遵循请求-响应模式，但客户端发送多个请求和服务端给出多个响应的顺序不受限制，这样既避免了"队头堵塞"，又能更快获取响应。
    在复用同一个TCP连接时，服务器同时(或先后)收到了A、B两个请求，先回应A请求，但由于处理过程非常耗时，于是就发送A请求已经处理好的部分， 接着回应B请求，完成后，再发送A请求剩下的部分。HTTP/2长连接可以理解成全双工的协议。
    
    [参考](https://www.cnblogs.com/XiongMaoMengNan/p/8425724.html)
    [数据流、消息和帧](https://developers.google.com/web/fundamentals/performance/http2/?hl=zh-cn#_3)
    
- 头信息压缩
- 服务器推送

[参考](https://yuchengkai.cn/docs/zh/cs/#http-2-0)

## HTTPS
HTTP、TCP、IP, 在HTTP基础上加了SSL/TLS层  HTTP、SSL/TLS、TCP、IP

- https协议需要证书
- http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议
- http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443

SSL（Secure Sockets Layer）安全套节字层
TLS (Transport Layer Security) 传输层安全性协议

TCP 三次握手前先进行 SSL/TLS的四次握手过程
http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html
