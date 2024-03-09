HTTP 1.0中默认是关闭的，需要在http头加入"Connection: Keep-Alive"，才能启用Keep-Alive；

HTTP 1.1中默认启用Keep-Alive，如果加入"Connection: close "，才关闭。

HTTP/1.1 存在一个问题，单个 TCP 连接在同一时刻只能处理一个请求，意思是说：两个请求的生命周期不能重叠，
任意两个HTTP请求从开始到结束的时间在同一个 TCP 连接里不能重叠。虽然 HTTP/1.1 规范中规定了Pipelining来试图解决这个问题，
但是这个功能在浏览器中默认是关闭的。但是，HTTP2 提供了 Multiplexing 多路传输特性，可以在一个 TCP 连接中同时完成多个 HTTP 请求。

维持和服务器已经建立的 TCP 连接，在同一连接上顺序处理多个请求。

和服务器建立多个 TCP 连接。

Chrome允许对同一个Host建立最多六个TCP连接。

