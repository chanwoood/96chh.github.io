---
layout: post
title:  'HTTP 方法'
categories: HTTP
author: CHH
---

* content
{:toc}




## GET 和 POST

GET：获取资源。
POST：传输实体主体。

对比：
1. 都能使用额外的参数，但是 GET 的参数是以查询字符串出现在 URL 中，而 POST 的参数存储在实体主体。
2. GET 的传参方式比 POST 安全性较差，因为 GET 传的参数在 URL 中是可见的。
3. GET 只支持 ASCII 字符，如果参数为中文则可能会出现乱码，而 POST 支持标准字符集。

## HEAD

获取报文首部
与 GET 方法相比，响应不返回报文实体。
主要用于确认 URL 的有效性以及资源更新的日期时间等。

## PUT 和 DELETE

PUT：上传文件
DELETE：删除文件
相同点：不带验证机制，任何人都可以上传或删除，存在安全性问题，一般不使用。

## PATCH

对资源进行修改

## OPTIONS

查询指定的 URL 能够支持的方法
返回格式 Allow: GET, POST, HEAD, OPTIONS 

## CONNECT

先用 CONNECT 方法与代理服务器通信，要求建立隧道。
然后通过隧道与目标服务器通信。使用 SSL（Secure Sokets Layer，安全套接字）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加密后经网络隧道传输。

## TRACE

服务器会将通信路径返回给客户端。
每经过一个服务器，首部字段 Max-Forwards 就会减 1，当数值为 0 时就停止传输。
通常不会使用 TRACE，并且它容易受到 XST 攻击（Cross-Site Tracing，跨站追踪）