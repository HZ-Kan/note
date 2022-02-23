《白帽子讲WEB安全》

[关于Http请求header之Referer讲解](https://www.jianshu.com/p/1a6abab212ed)

[HTTP请求头之User-Agent](https://www.jianshu.com/p/c5cf6a1967d1)

[HTTP请求头和响应头详解](https://www.jianshu.com/p/9a68281a3c84)

[面试：彻底理解Cookie以及Cookie安全](https://juejin.cn/post/6844904102544031757)

[深入理解Cookie](https://www.jianshu.com/p/6fc9cea6daa2)

# Session与Cookie

​	由于http协议是无状态的，无状态就是说，服务器无法判断用户身份。Cookie是将一些文本信息以键值对的形式保存在客户端，而Session是将某些信息保存在服务器端，当浏览器发送请求时会将cookie一同提交给服务器，服务器再根据session检查cookie，判断用户的状态。

# csrf

![ccc2067b8fa86a449070ce1f62097012.png](https://img-blog.csdnimg.cn/img_convert/ccc2067b8fa86a449070ce1f62097012.png)

# token怎么防御csrf？

​	Token 是在服务端产生的。如果前端使用用户名/密码向服务端请求认证，服务端认证成功，那么在服务端会返回 Token 给前端。前端可以在每次请求的时候带上 Token 证明自己的合法地位。而csrf只能拿到用户的cookie不能拿到token，所以通过不了验证。

# LocalStorage，SessionStorage

[LocalStorage使用总结](https://www.cnblogs.com/st-leslie/p/5617130.html)

[LocalStorage跨域解决方案](https://blog.csdn.net/sflf36995800/article/details/53290457)（当前域localStorage超出限制时，可以使用其他域中的localStorage）
