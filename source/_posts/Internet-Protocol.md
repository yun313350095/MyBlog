---
title: 处理响应
date: 2018-12-08
categories: 网络协议
tags:
  - 网络协议
  - 网络API开发
---

### 1.简介
   目前为止我们已经发起了各种各样的请求，也看到了服务器发回的原始 HTTP 数据。服务器返回的原始数据就是所谓的响应。本章我们会分析 HTTP 响应里的各种组成部分。

### 2.常见的HTTP状态码

状态码	            | 状态文本	       | 含义
--------------------|------------------|-----------------------
200                 | ok                    | 客户端请求成功
302                 | 302 Found 临时重定向    | 临时重定向，在Location响应首部的值仍为新的URL(显示重定向)
400	                | Bad Request           |  错误请求。由于语法错误，该请求无法完成
403                 | Forbidden 禁止访问     | 禁止访问，服务器拒绝响应
404                 | Not Found             | 请求的资源不存在
500                 | Internal Server Error | 服务器发生了不可预期的错误

![](/images/http_status_2018_12_09.png)
> 作为一个 web 开发者，你应该熟知上面的响应状态码和其代表的含义。

### 3.常见的HTTP请求头

请求头		        | 说明	        
--------------------|------------------
Accept-charset	    | 用于指定客户端接受的字符集
Accept-Encoding	    | Accept-Encoding	用于指定可接受的内容编码。如：Accept-Encoding：gzip，deflate
Accept-Language	    | 用于指定一种自然语言。如：Accept-Language：zh-CN
Host	            | 用于指定被请求资源的Internet主机和端口号。如：Host：www.taobao.com
User-Agent	        | 客户端将它的操作系统、浏览器和其他属性告诉服务器。 如：User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.98 Safari/537.36
Connection	        | 当前连接是否保持。如：Connection：Keep-Alive

> 一般前端请求后端API接口的时候, 最好添加一些响应头

### 4.常见的HTTP响应头

响应头		        | 说明	        
--------------------|------------------
Server	            | 使用的服务器名称。如：Server：Apache／1.3.6（Unix）
Content-Type	    | 用来指明发送给接收者的实体正文的媒体类型。如：Content-Type：text/html；charset=utf-8
Content-Encoding	| 与请求头Accept-Encoding对应，告诉浏览器服务端采用的是什么压缩编码
Content-Language	| 描述了资源所用的自然语言，与Accept-Language对应
Content-Length	    | 指明实体正文的长度，用以字节方式存储的十进制数字来表示
Keep-Live	        | 保持连接时间。如：Keep-Live：timeout=5，max=120
