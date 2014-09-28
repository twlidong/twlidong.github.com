---
layout: post
title: "我不知道的URL encode"
date: 2013-12-11 22:42:10 +0800
comments: true
categories: 
---
###为什么使用URL encoding?###

URL在Internet中传输只能是ASCII字符集，正是由于在URL中往往包含很多非ASCII字符之外的字符，才需要将它转化。通常会用“%”和两个十六进制的数字去替换非法的ASCII字符(percent_encode)。

当输入到HTML form表单中的数据被提交的时候，表单中的名字，值都会被编码通过http请求GET, POST等发送给服务器端，默认的编码方式是基于早期版本的general URI perccent-encoding规则以及一些改进。通过这种形式编码的数据的MIME格式是 application/x-www-form-urlencoded。当发送的是HTTP GET请求的时候，application/x-www-form-urlencoded包含在请求URI的query部分。当发送的是HTTP POST请求时，数据本身被放在信息的body中，media-type放在信息的Content-Type header中。

###General URL Sytanx###

对于URL: https://www.example.com:8080/file;p=1?q=2#third

 **Part** | **Data**
 |:---:|---:|
 Scheme | https
 Host | www.example.com
 Port | 8080
 Path | file
 Path parameters  |   p=1
 Query parameters |   q=2
 Fragment | third


###URL grammar###

对于URL是如何组装和如何分割，是有着一定的语法。比如，”://“ 是分割scheme和host, ”/“是分割host和path fragment, query通常跟着”?“之后。这就意味着特定的某些字符会因为语法的需求而被保留。有些是针对所有的URI都保留，有些则是针对某些scheme。所有这些被保留的字符是如果是出现在不被允许使用的地方就需要URL encoded。举个例子”?“如果出现在path fragment中就需要被转换成”%3F“, 从而使它不具备了句法含义(syntactic meaning)。"http://www.example.com/to_be_or_not_to_be?.jpg"转换成"http://www.example.com/to_be_or_not_to_be%3F.jpg", 而不会被误认为"?"在这里是一个query部分。

**The unreserved characters can be encoded, but should not be encoded. The unreserved characters are:**
```
A B C D E F G H I J K L M N O P Q R S T U V W X Y Z
a b c d e f g h i j k l m n o p q r s t u v w x y z
0 1 2 3 4 5 6 7 8 9 - _ . ~
```
**The reserved characters have to be encoded only under certain circumstances. The reserved characters are:**
```
! * ' ( ) ; : @ & = + $ , / ? % # [ ]
```


###Pitfalls###

* **保留字符并不是你想象中的那样被编码**

   * "?"若在query部分可以不被转义

   * "/" 若在query部分可以不被转义

   * "=" 若在path parameter或query parameter部分或者是path segment中可以不被转义

   * ":@-._~!$&'()*+,;=" 若在path segment中将可以不被转义

   * "/?:@-._~!$&'()*+,;=" 若在fragment部分将可以不被转义

* **保留字符在URL不同的部分会被不同的对待**

  " "(空格)在path fragment部分被编码成"%20"(绝对不是"+"),而"+"会被保留, 不会被编码。而" "(空格)在query部分可能会被编码成"+"(为了向后兼容)或者是"%20", 然而"+"则会被转义成"%2B"。、

******************************************************
**实践小结：在ruby中我们经常使用的URL encode的方法大致有三种**

 * URI.escape()是替换掉不安全的字符,但是转义出来的URL在query部分含有"+"并不能和上述标准十分相符,而URI.encode()是URI::Escape#escape的别名
 * CGI.escape()本义是对HTML form表单请求做转义，它的转义比较彻底，看起来也比较残暴（但是也相当好用）,最起码能保证编码出来的URL肯定是合法的
 * 当和URIs打交道, 也可以使用Addressable, 它提供了url encoding, form encoding 和 normalizes URLs.

```ruby
URI.encode('www.example.com/François+?a=3+4')
=> "www.example.com/Fran%C3%A7ois+?a=3+4”
```
"+"该被编码的
```ruby
CGI.escape("www.example.com/François+?a=3+4”)
=> "www.hel.com%2FFran%C3%A7ois%2B%3Fa%3D3%2B4”
```
看起来几乎所有的保留字符都被编码了


引用:【http://blog.lunatech.com/2009/02/03/what-every-web-developer-must-know-about-url-encoding 】









