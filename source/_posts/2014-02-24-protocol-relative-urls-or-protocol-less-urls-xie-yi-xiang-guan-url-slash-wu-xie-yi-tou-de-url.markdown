---
layout: post
title: "Protocol relative  URLs or Protocol-less URLs(协议相关URL/无协议头的URL)"
date: 2014-02-24 22:15:18 +0800
comments: true
categories: 
---

```
<img src="//domain.com/img/logo.png">
```

也许这样的一个URL第一眼看去我们会无节操的骂去，谁***把协议头都忘了写了！！ 当爆出粗口的刹那间，“年少无知”也许就成了最优雅的形容词。

当再看到这段

```
 @font-face {
   font-family: 'Roboto';
   font-style: italic;
   src: url('//themes.googleusercontent.com/font?kit=biUEjW7P-lfzIZFXrcy-wQ') format('woff');
 }
```
的时候，也许已经开始胆怯了。。。

没错，这也许就是一种你不曾知道的URL。[RFC 3986][id]
[id]: http://tools.ietf.org/html/rfc3986#section-4.2

Protocol relative URLs 是指如果浏览器中请求页面的的协议是HTTPS，那么对于像 "//domain.com/img/logo.png" 这样的URL将同样用HTTPS来请求资源，同理如果是HTTP的话，这个URL将会变为"http://domain.com/img/logo.png"。那么对于这样的规范，有什么用处呢？下面我举一个栗子:

你有一个仅支持http请求的应用，突然有一天需要同时支持http和https, 当你想当然的在web服务器中打开ssl端口的时候，一请求页面发现样式乱了，顿时就慌了。才意识到浏览器中诸如：
```
The page at ‘https://a.example.com/1' was loaded over HTTPS, but displayed insecure content from
'http://staic.style.com/static/img/logo.png': this content should also be loaded over HTTPS.
```
这样的警告。

一个支持https请求的网站，应确保所有的内容都是用https加载的，从而防止如上的[mixed content warnings][link]。
[link]: https://blog.servertastic.com/firefox-23-to-block-mixed-content/

当赶紧去把代码中数之不尽hard code的http改为https的时候，可能顿时就一边改一边还窃喜自己顿时就能解决这个问题的时候, 不知又悄悄的引入了一些“不便之处”：当页面是被标准的http请求时，这些被改成https协议的资源，不得不在此时依然通过ssl去连接请求https外部资源。

同时支持http和https，怎样让自己的代码更自然简洁一些，当然是使用 Protocol relative URL。

很简单，不用去管协议，就像这样
```
//some-domain.com/path/to/resource
```
好了，不再有mixed content警告，可以确定任何放在src和href属性中的url都会被浏览器可靠的处理。同时免去了你使用 (request.protocol / request.scheme 或者 window.location.protocol) 去解析请求的协议。


此时，有没有顿然暗爽~


**但是，要注意的是：**

1. 例如存放 js 和 css 文件这样的静态服务器不同时支持https或者是http, 会有潜在的风险~ 比如google analytics曾经就对http和https分别用不同的subdomain来对待， 一个为 http://www.google-analytics.com/ga.js, 一个为https://ssl.google-analytics.com/ga.js
2. 如果你打算支持IE6的话，它会在IE6下不知所措。
3. 还有倘若留意一下上面的写法在IE8下面请求一个样式文件的表现的话，你可能会更失望，请求了两次！！（一次http，一次https）不过你决定抛弃IE8的话，那就忽略这条。详细请参考http://www.stevesouders.com/blog/2010/02/10/5a-missing-schema-double-download/
4. http请求和https请求可能会留有两份不同的cache。
5. 一些邮件客户端（特别是outlook）将不会以为它是http或者https协议，而是会以为是本机上的资源从而使用file://协议

**倘若这些都不是问题，那就尝试一下Protocol relative URL**

