# log

create  2021-01-06

# 参考

[Ajax MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/AJAX/Getting_Started)

[XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)

[使用 XMLHttpRequest](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest)

[Ajax小白入门到实战请求(2020)](https://www.bilibili.com/video/BV11J411k78L)

[ajax中的onload和readychange区别](https://www.cnblogs.com/doudoublog/p/8608360.html)

[SuperAgent使用文档](https://www.jianshu.com/p/1432e0f29abd)

[你真的会使用XMLHttpRequest吗？](https://segmentfault.com/a/1190000004322487)

# Ajax

![image-20220106224004493](D:/Typora/img/image-20220106224004493.png)

​	AJAX是异步的JavaScript和XML（**A**synchronous **J**avaScript **A**nd **X**ML）。简单点说，就是使用 `XMLHttpRequest` 对象与服务器通信。 它可以使用JSON，XML，HTML和text文本等格式发送和接收数据。AJAX最吸引人的就是它的“异步”特性，也就是说它可以在不重新刷新页面的情况下与服务器通信，交换数据，或更新页面。

# XMLHttpRequest对象

​	`XMLHttpRequest`（XHR）对象用于与服务器交互。通过 XMLHttpRequest 可以在不刷新页面的情况下请求特定 URL，获取数据。这允许网页在不影响用户操作的情况下，更新页面的局部内容。`XMLHttpRequest` 在 [AJAX](https://developer.mozilla.org/zh-CN/docs/Glossary/AJAX) 编程中被大量使用。

## 状态码

```js
// readyState 状态码
// 0: 请求未初始化
// 1: 服务器连接已建立
// 2: 请求已接收
// 3: 请求处理中
// 4: 请求已完成，且响应已就绪

// HTTP 状态码
// 200 - 服务器成功返回网页
// 404 - 请求的网页不存在
// 503 - 服务器暂时不可用
```

## 创建

```js
let xhr = new XMLHttpRequest();
```

## 常用方法

**初始化一个请求：**open

```js
/* open(type, url/file, async)
* type: 请求类型,get、post...
* url/file: 路径/文件
* async: 是否异步,boolean
*/

xhr.open();
```

**请求方式：**onload、 onreadystatechange 

```js
你有两个方法去访问拿到的数据：

httpRequest.responseText – 服务器以文本字符的形式返回
httpRequest.responseXML – 以 XMLDocument 对象方式返回，之后就可以使用JavaScript来处理

xhr.onload = function() {
    ...
};
    
xhr.onreadystatechange = function() {
    ...
};
    
// 区别
    onreadystatechange()的定义是只要返回的状态码只要变化时就回调一次函数，而onload只有状态码为4时才能回调一次函数。

这边提下onprogress()，也就是当状态码为3时，会执行这个函数。
```

**发送请求：**send()

```js
xhr.send();
```

**请求处理中：** onprogress()

```js
xhr.onprogress = function() {
    ...
};
```

**取消请求：**abort()

```js
xhr.abort();
```

**其他方法：**

![image-20220106232212371](D:/Typora/img/image-20220106232212371.png)

## 示例

```js
// 同级目录下创建test.json文件，添加数据
// 创建html文件并在script中添加如下代码，在服务器上运行即可

let xhr = new XMLHttpRequest();
xhr.open('get', 'test.json', true);
xhr.onload = function() {
	console.log(this.responseText);
}
xhr.send()
```

# 其他交互技术

1. jQuery（对ajax的封装）
2. Axios（基于Promise对ajax的封装）
3. [Superagent](https://visionmedia.github.io/superagent/#test-documentation)
4. Fetch（官方API）
5. Prototype（了解即可）
6. Node HTTP