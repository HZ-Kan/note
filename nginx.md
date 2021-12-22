# 写在前面

​	仅供学习

# 日志

创建 																																								2021/12/16

# 参考

[【狂神说】Nginx最新教程通俗易懂，40分钟搞定！](https://www.bilibili.com/video/BV1F5411J7vK)

[正向代理 反向代理 本质区别？](https://www.zhihu.com/question/36412304/answer/471185862)

[反向代理和正向代理区别](https://www.cnblogs.com/taostaryu/p/10547132.html)

# 简介

​	Nginx（engine x）是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。其特点是占有内存少，并发能力强。

# 正向代理与反向代理

Unlike a [forward proxy](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Proxy_server%23Forward_proxy), which is an intermediary for its associated clients to contact any server, a reverse proxy is an intermediary for its associated servers to be contacted by any client。

正向代理是客户端和其他所有服务器（重点：所有）的代理者，而反向代理是客户端和所要代理的服务器之间的代理。

**总结：**

​	正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端.
​	反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端


