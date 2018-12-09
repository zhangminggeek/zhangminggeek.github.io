---
title: 跨域问题及解决方式
date: 2018-12-09 23:07:41
categories: http
tags: [跨域, http]
---

## 同源策略
同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。

如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的源。

|URL|结果|原因|
|---|---|---|
|http://store.company.com/dir2/other.html|成功||
|http://store.company.com/dir/inner/another.html|成功||
|https://store.company.com/secure.html|失败|不同协议 ( https和http )|
|http://store.company.com:81/dir/etc.html|失败|不同端口 ( 81和80)|
|http://news.company.com/dir/other.html|失败|不同域名 ( news和store )|

### 源的继承
data：URLs获得一个新的，空的安全上下文。

在页面中用 about:blank 或 javascript: URL 执行的脚本会继承打开该 URL 的文档的源，因为这些类型的 URLs 没有明确包含有关原始服务器的信息。

例如，about:blank 通常作为父脚本写入内容的新的空白弹出窗口的 URL（例如，通过  Window.open()  机制）。 如果此弹出窗口也包含代码，则该代码将继承与创建它的脚本相同的源。data: URL会得到一个新的空的安全上下文。

>注意：在Gecko 6.0之前，如果用户在位置栏中输入 data URLs，data URLs 将继承当前浏览器窗口中网页的安全上下文。

### IE例外
当涉及到同源策略时，Internet Explorer 有两个主要的不同点：

- 授信范围（Trust Zones）：两个相互之间高度互信的域名，如公司域名（corporate domains），不遵守同源策略的限制。
- 端口：IE 未将端口号加入到同源策略的组成部分之中，因此 http://company.com:81/index.html 和http://company.com/index.html  属于同源并且不受任何限制。

这些例外是非标准的，其它浏览器也未做出支持，但会助于开发基于window RT IE的应用程序。

## 跨域网络访问
同源策略控制了不同源之间的交互，例如在使用XMLHttpRequest 或 <img> 标签时则会受到同源策略的约束。这些交互通常分为三类：

- 通常允许跨域写操作（Cross-origin writes）。例如链接（links），重定向以及表单提交。特定少数的HTTP请求需要添加 preflight。
- 通常允许跨域资源嵌入（Cross-origin embedding）。
- 通常不允许跨域读操作（Cross-origin reads）。但常可以通过内嵌资源来巧妙的进行读取访问。例如可以读取嵌入图片的高度和宽度，调用内嵌脚本的方法，或availability of an embedded resource。

以下是可能嵌入跨源的资源的一些示例：

- `<script src="..."></script>` 标签嵌入跨域脚本。语法错误信息只能在同源脚本中捕捉到。
- `<link rel="stylesheet" href="...">` 标签嵌入CSS。由于CSS的松散的语法规则，CSS的跨域需要一个设置正确的Content-Type 消息头。不同浏览器有不同的限制： IE, Firefox, Chrome, Safari (跳至CVE-2010-0051)部分 和 Opera。
- `<img>`嵌入图片。支持的图片格式包括PNG,JPEG,GIF,BMP,SVG,...
- `<video>` 和 `<audio>`嵌入多媒体资源。
- `<object>`, `<embed>` 和 `<applet>` 的插件。
- @font-face 引入的字体。一些浏览器允许跨域字体（ cross-origin fonts），一些需要同源字体（same-origin fonts）。
- `<frame>` 和 `<iframe>` 载入的任何资源。站点可以使用X-Frame-Options消息头来阻止这种形式的跨域交互。

## 解决方式
### JSONP
JSONP 的解决方案就是通过 script 标签进行跨域请求。

- 前端设置好回调函数，并把回调函数当做请求 url 携带的参数。
- 后端接受到请求之后，返回回调函数名和需要的数据。
- 后端响应并返回数据后，返回的数据传入到回调函数中并执行。

```javascript
<!-- 通过原生使用 script 标签 -->
<script>
    function jsonpCallback(data) {
        alert('获取到的数据了，打开控制台瞧瞧');
        console.log(data);
    }
</script>
<script src="http://127.0.0.1:3000?callback=jsonpCallback"></script>
```

也可以使用 AJAX GET 请求方式来跨域请求（axios GET 方式跨域同理）。
```javascript
<!-- AJAX GET 请求 -->
<script>
    function jsonpCallback(data) {
        alert('获取到的数据了，打开控制台瞧瞧');
        console.log(data);
    }
    $.ajax({
        type: 'GET', // 必须是 GET 请求
        url: 'http://127.0.0.1:3000',
        dataType: 'jsonp', // 设置为 jsonp 类型
        jsonpCallback: 'jsonpCallback' // 设置回调函数
    })
</script>
```

优缺点：

- 兼容性好，低版本的 IE 也支持这种方式。
- 只能支持 GET 方式的 HTTP 请求。
- 只支持前后端数据通信这样的 HTTP 请求，并不能解决不同域下的页面之间的数据交互通信问题

### CORS
#### CORS的配置
跨源资源共享（CORS）是一种机制，它使用其他HTTP请求头告诉浏览器让在一个源（域）上运行的Web应用程序有权访问不同来源服务器中的所选资源。Web应用程序在请求具有与自己不同源（域，协议和端口）的资源时，会发出跨源HTTP请求。

出于安全原因，浏览器会限制从脚本中发起的跨源HTTP请求。例如，XMLHttpRequest和Fetch API遵循同源策略。这意味着使用这些API的Web应用程序只能从加载应用程序的同一源请求HTTP资源，除非来自其他来源的响应包含正确的CORS标头。

{% asset_img images/CORS_principle.png %}

CORS机制支持浏览器和Web服务器之间的安全跨源请求和数据传输。现代浏览器在API中使用CORS，如XMLHttpRequest和Fetch，以帮助减轻跨域的HTTP请求的风险。

服务端设置 Access-Control-Allow-Origin 字段，值可以是具体的域名或者 '*' 通配符，配置好后就可以允许跨域请求数据。

```javascript
<script>
    $.ajax({
    type: 'post',
    url: 'http://127.0.0.1:3000',
    success: function(res) {
        alert('获取到的数据了，打开控制台瞧瞧');
        console.log(res);
    }
})
</script>
```

服务端如何设置跨域字段？ 后端语言设置跨域的方式都不一致，具体可参考后端语言本身的 API。

Node 端设置
```javascript
// 响应头添加 Access-Control-Allow-Origin
res.writeHead(200, {
    'Access-Control-Allow-Origin': '*'
});

// 或者使用了 Express 这样的框架
res.header("Access-Control-Allow-Origin", "*");
```
#### 简单请求和复杂请求
在 CORS 机制中, 把请求分为了 简单请求 和 复杂请求, 一个 HTTP 请求若想要让自己成为一个简单请求就要满足以下条件:

- 首先, 请求方式的限制: 请求方式(method) 只能是 GET POST HEAD 三者中的一个
- 其次就是请求头字段的限制: 请求头字段必须包含在以下集合中, 包括: Accept Accept-Language Content-Language Content-Type DPR Downlink Save-Data Viewport-Width Width.
- 其次就是请求头值的限制: 当请求头中包含 Content-Type 的时候, 其值必须为 text/plain multipart/form-data application/x-www-form-urlencoded(这个是 form 提交默认的 Content-Type) 三者中的一个.

#### 预检请求
跨域资源共享标准新增了一组HTTP首部字段，允许服务器声明哪些源站通过浏览器有权访问哪些资源。另外，规范要求，对哪些可能对服务器数据产生副作用的请求方法（特别是GET以为的HTTP请求，或者搭配某些MIME类型的POST请求，）浏览器必须首先使用OPTION方法发起一个预检请求，从而获知服务端是否允许该跨域请求。服务端确认允许之后，才发起实际的HTTP请求。在预检请求的返回中，服务器端也可以通知客户端，是否需要携带身份凭证（包括Cookie和HTTP认证相关数据）。

所有的简单请求跨域访问都不会触发预检请求。

浏览器在接受到我们发送的跨域请求的指令时, 会自动判断我们的请求是否属于跨域请求, 如果是的话便会发出预检请求, 预检请求的请求头信息也是浏览器根据我们的请求信息自动添加的. 示例项目中, 因为我们的请求是 PUT 类型的, 所以在预检请求的时候会添加 Access-Control-Allow-Methods: PUT 来咨询服务器自己是否可以向它发送这种类型的请求. 同理, 由于我们的请求中有自定义请求头 token 所以, 在预检请求中, 浏览器要和服务器做是否可以添加自定义请求头的协商. 只有当浏览器和服务器之间的预检请求协商通过了, 浏览器才会继续发送真正的 AJAX 请求.

### Server Proxy
通过服务端代理请求的方式也是解决浏览器跨域问题的方案。同源策略只是针对浏览器的安全策略，服务端并不受同源策略的限制，也就不存在跨域的问题。具体步骤如下：

- 前端正常请求服务端提供的接口。比如请求接口：http://localhost:3000 。
- 通过服务端设置代理发送请求，请求到数据后再将需要的数据返回给前端。比如设置的代理请求接口是 cnodejs.org/api/v1/topi… ，服务端代理将数据请求回来之后再将数据 http://localhost:3000 接口返回给前端。
```javascript
// 服务端代理请求代码
// 服务端只是简单的通过正常的 HTTP 请求的方式来代理请求接口数据
// 或者也可以使用 proxy 模块来代理
var url = 'https://cnodejs.org/api/v1/topics';        
https.get(url, (resp) => {
    let data = "";
    resp.on('data', chunk => {
        data += chunk;
    });
    resp.on('end', () => {
        res.writeHead(200, {
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'application/json; charset=utf-8'
        });
        res.end(data);
    });
})
```

### location.hash + iframe
location.hash + iframe 跨域通信的实现是这样的：

- 不同域的 a 页面与 b 页面进行通信，在 a 页面中通过 iframe 嵌入 b 页面，并给 iframe 的 src 添加一个 hash 值。
- b 页面接收到了 hash 值后，确定 a 页面在尝试着与自己通信，然后通过修改 parent.location.hash 的值，将要通信的数据传递给 a 页面的 hash 值。
- 但由于在 IE 和 Chrmoe 下不允许子页面直接修改父页面的 hash 值，所以需要一个代理页面，通过与 a 页面同域的 c 页面来传递数据。
- 同样的在 b 页面中通过 iframe 嵌入 c 页面，将要传递的数据通过  iframe 的 src 链接的 hash 值传递给 c 页面，由于 a 页面与 c 页面同域，c 页面可以直接修改 a 页面的 hash 值或者调用 a 页面中的全局函数。

大致流程：

{% asset_img images/165b1a74b0bc4290.png %}

a 页面代码
```javascript
<script>
    var iframe = document.createElement('iframe');
    iframe.style.display = 'none';
    iframe.src = "http://localhost:8081/b.html#data";
    document.body.appendChild(iframe);

    function checkHash() {
        try {
            var data = location.hash ? location.hash.substring(1) : '';
            console.log('获得到的数据是：', data);
        }catch(e) {}
    }
    window.addEventListener('hashchange', function(e) {
        console.log('监听到hash的变化：', location.hash.substring(1));
    })
</script>
```

b 页面代码
```javascript
<script>
     switch(location.hash) {
         case '#data':
         callback();
         break;
     }
    function callback() {
        var data = "testHash"
        try {
            parent.location.hash = data;
        }catch(e) {
            var ifrproxy = document.createElement('iframe');
            ifrproxy.style.display = 'none';
            ifrproxy.src = 'http://localhost:8080/c.html#' + data;
            document.body.appendChild(ifrproxy);
        }
    }
 </script>  
```

c 页面代码
```javascript
<script>
    // 修改 a 页面的 hash 值
    parent.parent.location.hash = self.location.hash.substring(1);
    // 调用 a 页面的全局函数
    parent.parent.checkHash();
</script>
```

优缺点：
- hash 传递的数据容量有限。
- 数据直接暴露在 url 中。

## 参考文章
[浏览器的同源策略](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
[前端跨域方法论](https://juejin.im/post/5b91d3be5188255c95380b5e)
[【小哥哥, 跨域要不要了解下】JSONP](https://juejin.im/post/5c07fa04e51d451de968906b)
[【小哥哥, 跨域要不要了解下】CORS 基础篇](https://juejin.im/post/5c0a55e76fb9a049ef2665ba)
[【小哥哥, 跨域要不要了解下】CORS 进阶篇](https://juejin.im/post/5c0b5a8851882548e9380afb)

