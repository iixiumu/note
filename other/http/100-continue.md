Expect:100-continue

在使用curl做POST的时候, 当要POST的数据大于1024字节的时候, curl并不会直接就发起POST请求, 而是会分为俩步,

  1. 发送一个请求, 包含一个Expect:100-continue, 询问Server使用愿意接受数据
  2. 接收到Server返回的100-continue应答以后, 才把数据POST给Server

https://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html#sec8.2.3
