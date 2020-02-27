## Cookie
### 什么是cookie？
http是一个无状态协议，什么是无状态呢？就是说这一次请求和上一次请求是没有任何关系的，互不认识的，没有关联的。这种无状态的的好处是快速。坏处是假如我们想要把www.zhihu.com/login.html和www.zhihu.com/index.html关联起来，必须使用某些手段和工具。

由于http的无状态性，为了使某个域名下的所有网页能够共享某些数据，session和cookie出现了。客户端访问服务器的流程如下：
* 首先，客户端会发送一个http请求到服务器端。
* 服务器端接受客户端请求后，建立一个session，并发送一个http响应到客户端，这个响应头，其中就包含Set-Cookie头部。该头部包含了sessionId。Set-Cookie格式如下，具体请看Cookie详解
Set-Cookie: value[; expires=date][; domain=domain][; path=path][; secure]
* 在客户端发起的第二次请求，假如服务器给了set-Cookie，浏览器会自动在请求头中添加cookie
* 服务器接收请求，分解cookie，验证信息，核对成功后返回response给客户端

### cookie 常用属性项
| 属性项 | 属性项介绍 |
| --- | --- |
| NAME=VALUE	| 键值对，可以设置要保存的 Key/Value，注意这里的 NAME 不能和其他属性项的名字一样|
| Expires |	过期时间，在设置的某个时间点后该 Cookie 就会失效 |
| Domain	| 生成该 Cookie 的域名，如 domain="www.baidu.com" |
| Path |	该 Cookie 是在当前的哪个路径下生成的，如 path=/wp-admin/ |
|Secure |	如果设置了这个属性，那么只会在 SSH 连接时才会回传该 Cookie |


## Session


## cookie与session的区别
cookie以文本格式存储在浏览器上，存储量有限；而session存储在服务端，可以无限量存储多个变量并且比cookie更安全；
