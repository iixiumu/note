BasicAuth 的原理和实现. 简单来说, 它是检查你的 Headers 中的Authorization. 从中解析出 username 和 password, 和服务器保存的进行对比, 如果一致则通过, 否则返回 401/403 状态码, 表示客户端没有携带或携带了错误的认证信息

下面详细说一下怎么调用一个有 BasicAuth 认证的服务. 比如你的 username 是 zhangsan, password 是 helloworld(当然不建议用这种太简单的密码), 那么你只需要在你调用接口时在 Header 里加上一个 Header

Authorization: Basic $(base64_encode({username}:{password}))

即可. 更具体的, 也就是

Authorization: Basic emhhbmdzYW46aGVsbG93b3JsZA==.
