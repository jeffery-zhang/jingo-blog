---
title: Web 端跨域详解
description: 详细介绍 Web 端的跨域问题
date: 2022-04-30 12:31:31
categories:
  - Web
tags:
  - 跨域
  - http
  - https
  - 域名
  - 端口
---

# Web 端跨域详解

详细介绍 Web 端的跨域问题

## 跨域三种情况

当网络请求出现以下情况时, 就属于跨域请求:

1. 协议不同, 如 http 和 https
2. 域名不同
3. 端口不同

## 原理

跨域问题本质上是浏览器行为, 初衷是用于保护用户的访问安全, 属于浏览器同源策略的体现, 这是浏览器最核心和基础的安全功能, 同源策略的限制有以下几种:

1. Cookie、LocalStorage 和 IndexDB 无法读取
2. DOM 和 JS 对象无法获得
3. AJAX 请求不能发送

## 解决跨域

### JSONP

- 原理就是利用 script 标签没有跨域限制, 通过 src 属性设置包含 callback 参数的跨域 GET 请求, 服务端接口将返回数据拼凑到 callback 函数中返回给浏览器, 从而让前端拿到在 callback 函数中获取的数据

```html
<script>
  var script = document.createElement("script");
  script.type = "text/javascript";

  // 传参一个回调函数名给后端, 方便后端返回时执行这个在前端定义的回调函数
  script.src =
    "http://www.domain2.com:8080/login?user=admin&callback=handleCallback";
  document.head.appendChild(script);

  // 回调执行函数
  function handleCallback(res) {
    alert(JSON.stringify(res));
  }
</script>
```

JSONP 的缺点是只能发送 GET 请求

### 跨域资源共享(CORS)

CORS 是一个 W3C 标准, 它允许浏览器向跨域服务器发送 XHR 请求以客服 AJAX 的同源限制, 需要前后端同时支持

CORS 分为简单请求和非简单请求, 如果请求同时满足以下两个条件则是简单请求:

1. 请求方式

- 1. get
- 2. post
- 3. head

2. 请求头

- 1. Accept
- 2. Accept-Language
- 3. Content-Language
- 4. Content-Type 仅限于三个值: application/x-www-form-urlencoded, multipart/form-data, text/plain

若没有同时满足以上两个条件就属于非简单请求

#### 简单请求

发送简单请求时, 浏览器在头信息中增加一个 Origin 字段, 用来说明请求来源(协议 + 域名 + 端口), 服务端根据这个 Origin 来决定是否同意这次请求

CORS 请求设置响应头字段, 都以 Access-Control-开头:

1. Access-Control-Allow-Origin: 必选

- 它的值要么是请求时 Origin 字段的值, 要么是一个\*, 表示接受任意域名的请求

2. Access-Control-Allow-Credentials: 可选

- 它的值是一个布尔值, 表示是否允许发送 Cookie. 默认情况下, Cookie 不包括在 CORS 请求之中. 设为 true, 即表示服务器明确许可, Cookie 可以包含在请求中, 一起发给服务器. 这个值也只能设为 true, 如果服务器不要浏览器发送 Cookie, 删除该字段即可

3. Access-Control-Expose-Headers: 可选

- CORS 请求时, XMLHttpRequest 对象的 getResponseHeader()方法只能拿到 6 个基本字段: Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma. 如果想拿到其他字段, 就必须在 Access-Control-Expose-Headers 里面指定. 上面的例子指定, getResponseHeader('FooBar')可以返回 FooBar 字段的值

#### 非简单请求

非简单请求是那种对服务器有特殊要求的请求, 如请求方法是 PUT 或 DELETE, 或者 Content-Type 字段值为 application/json 等. 非简单请求会在正是通信前增加一次 http 查询请求, 称为预检请求(preflight)

预检请求的请求方法为 OPTIONS, 表示用于询问, 请求头里除了需要包含 Origin, 还需要包含两个字段:

1. Access-Control-Request-Method: 必选

- 用来列出浏览器的 CORS 请求会用到哪些 HTTP 方法, 如 PUT

2. Access-Control-Request-Headers: 可选

- 该字段是一个逗号分隔的字符串, 指定浏览器 CORS 请求会额外发送的头信息字段, 如 X-Custom-Header

服务器收到预检请求后, 检查了这些字段后, 确认允许跨源请求, 就可以做出回应, 响应头中除了 Access-Control-Allow-Origin 外, 还有其他 CORS 相关字段:

1. Access-Control-Allow-Methods: 必选

- 它的值是逗号分隔的一个字符串, 表明服务器支持的所有跨域请求的方法。注意, 返回的是所有支持的方法, 而不单是浏览器请求的那个方法, 这是为了避免多次预检请求

2. Access-Control-Allow-Headers

- 如果浏览器请求包括 Access-Control-Request-Headers 字段, 则 Access-Control-Allow-Headers 字段是必需的, 它也是一个逗号分隔的字符串, 表明服务器支持的所有头信息字段, 不限于浏览器在预检中请求的字段

3. Access-Control-Allow-Credentials: 可选

- 该字段与简单请求时的含义相同

4. Access-Control-Max-Age: 可选

- 用来指定本次预检请求的有效期, 单位为秒

### 代理跨域

通过启动一个代理服务器作为浏览器和服务器之间的中间服务来跨域, 如用 nginx 配置一个与前端同源的代理服务器, 将请求转发给真实的服务器实现反向代理

### document.domain 和 iframe

此方案仅限主域相同, 子域不同的跨域应用场景, 实现原理: 两个页面都通过 js 强制设置 document.domain 为基础主域, 就实现了同域

```html
<!-- 父窗口 -->
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
  document.domain = "domain.com";
  var user = "admin";
</script>

<!-- 子窗口 -->
<script>
  document.domain = "domain.com";
  // 获取父窗口中变量
  console.log("get js data from parent ---> " + window.parent.user);
</script>
```

### postMessage 跨域

postMessage 是 H5 的 api, 主要用于解决以下问题:

- 页面和其打开的新窗口之间的消息传递
- 多窗口之间消息传递
- 页面与嵌套的 iframe 的消息传递
- 以上场景的跨域消息传递

postMessage(data,origin)方法接受两个参数:

1. data: html5 规范支持任意基本类型或可复制的对象, 但部分浏览器只支持字符串, 所以传参时最好用 JSON.stringify()序列化
2. origin: 协议+主机+端口号, 也可以设置为"\*", 表示可以传递给任意窗口, 如果要指定和当前窗口同源的话设置为"/"

```html
<!-- a.html -->
<iframe
  id="iframe"
  src="http://www.domain2.com/b.html"
  style="display:none;"
></iframe>
<script>
  var iframe = document.getElementById("iframe");
  iframe.onload = function () {
    var data = {
      name: "aym",
    };
    // 向domain2传送跨域数据
    iframe.contentWindow.postMessage(
      JSON.stringify(data),
      "http://www.domain2.com"
    );
  };

  // 接受domain2返回数据
  window.addEventListener(
    "message",
    function (e) {
      alert("data from domain2 ---> " + e.data);
    },
    false
  );
</script>

<!-- b.html -->
<script>
  // 接收domain1的数据
  window.addEventListener(
    "message",
    function (e) {
      alert("data from domain1 ---> " + e.data);

      var data = JSON.parse(e.data);
      if (data) {
        data.number = 16;

        // 处理后再发回domain1
        window.parent.postMessage(
          JSON.stringify(data),
          "http://www.domain1.com"
        );
      }
    },
    false
  );
</script>
```

### WebSocket 跨域

WebSocket protocol 是 HTML5 的一种协议, 它实现了浏览器与服务器全双工通信, 同时允许跨域通讯, 是 server push 技术的一种很好的实现
