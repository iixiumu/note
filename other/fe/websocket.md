http://javascript.ruanyifeng.com/htmlapi/websocket.html
WebSocket 协议在2008年诞生，2011年成为国际标准。所有浏览器都已经支持了。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于服务器推送技术的一种。WebSocket 允许服务器端与客户端进行全双工（full-duplex）的通信。举例来说，HTTP 协议有点像发电子邮件，发出后必须等待对方回信；WebSocket 则是像打电话，服务器端和客户端可以同时向对方发送数据，它们之间存着一条持续打开的数据通道。

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信，完全可以取代 Ajax。

（6）协议标识符是ws（如果加密，则为wss，对应 HTTPS 协议），服务器网址就是 URL。

ws://example.com:80/some/path

WebSocket 握手
浏览器发出的 WebSocket 握手请求类似于下面的样子：

GET / HTTP/1.1
Connection: Upgrade
Upgrade: websocket
Host: example.com
Origin: null
Sec-WebSocket-Key: sN9cRrP/n9NdMgdcy2VJFQ==
Sec-WebSocket-Version: 13
上面的头信息之中，有一个 HTTP 头是Upgrade。HTTP1.1 协议规定，Upgrade表示将通信协议从HTTP/1.1转向该字段指定的协议。Connection字段表示浏览器通知服务器，如果可以的话，就升级到 WebSocket 协议。Origin字段用于提供请求发出的域名，供服务器验证是否许可的范围内（服务器也可以不验证）。Sec-WebSocket-Key则是用于握手协议的密钥，是 Base64 编码的16字节随机字符串。

服务器的 WebSocket 回应如下。

HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: fFBooB7FAkLlXgRSz0BT3v4hq5s=
Sec-WebSocket-Origin: null
Sec-WebSocket-Location: ws://example.com/
上面代码中，服务器同样用Connection字段通知浏览器，需要改变协议。Sec-WebSocket-Accept字段是服务器在浏览器提供的Sec-WebSocket-Key字符串后面，添加“258EAFA5-E914-47DA-95CA-C5AB0DC85B11”字符串，然后再取 SHA-1 的哈希值。浏览器将对这个值进行验证，以证明确实是目标服务器回应了 WebSocket 请求。Sec-WebSocket-Location字段表示进行通信的 WebSocket 网址。

完成握手以后，WebSocket 协议就在 TCP 协议之上，开始传送数据。