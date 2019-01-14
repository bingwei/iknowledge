# HTTP

Forward 和 redirect:

+ Forward (转发): 服务器行为，服务器获取跳转页面内容传给用户，用户地址栏不变
+ Redirect (重定向): 客户端行为，本质上为两次请求，地址栏改变，前一次请求对象消失
  + HTTP 301

TODO Cookie 和 Session
https://www.zhihu.com/question/19786827

## Stateful web service

Stateful / stateless application:

+ Stateful: 为每个 client session 存储数据，供下次使用
+ Stateless: 不存储关于 client session 的状态，每次请求都如同第一次请求一样

HTTP 协议是 stateless 的，但可以通过一些手段模拟出 stateful 的体验，即实现 session。Session 是一个抽象的概念（OSI 七层模型第 5 层），是 user 和 server 之间一对一的交互。

Cookie 是实现 session 的一种机制。如果浏览器禁用了 cookie，还可以在 URL 中传递 session id。

### Session

+ Server 返回给 client 一个 _session identifier_
+ Client 在后续的请求中将 session id 作为 request 的一部分
+ Key-value 结构
  + Key: session id
  + Value: session data
+ Client 存储 session id，server 存储 session data 以及 key-value 结构

### Cookie

Cookie 是保存在浏览器中的、包含 session 数据的小文件。

+ 第一次访问网站时，server 发送 session 数据，浏览器保存为 cookie
  + Response header 中包含 `set-cookie` 字段
  + 登录后，cookie 中包含一个 `user_session` 之类的字段
+ 其后访问网站时，浏览器发送 cookie，server 会比较 client-side 的 cookie 与 server-side 的 session 数据
  + request header 中包含 `cookie` 字段
  + 用户保持登录状态
+ 如果在浏览器删除了 cookie，相当于退出登录

Cookie 的一个重要作用是实现 session。不过 cookie 也有其他的作用，例如保存用户名和密码。