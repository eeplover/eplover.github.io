---
layout: post
title: CDNs的工作原理
categories: pages
excerpt: CDNs的工作原理
tags: network CDNs cache
---

先看下 **Nicholas C. Zakas** 的这篇文章
[How CDNs work](https://www.nczonline.net/blog/2011/11/29/how-content-delivery-networks-cdns-work/)。    
接下来的内容大部分是对这篇文章的摘抄。
<br>

### 什么是 CDNs ?
Content Delivery Networks，即内容分发网络。怎么理解呢？下面引用 NCZ 的一段描述：

> Physics determines how fast one computer can contact another over physical connections, and so attempting to access a server in China from a computer in the United States will take longer than trying to access a U.S. server from within the U.S. To improve user experience and lower transmission costs, large companies set up servers with copies of data in strategic geographic locations around the world. This is called a CDN, and these servers are called edge servers, as they are closest on the company’s network to the end-user.    

简而言之，CDNs 就是一些（缓存）服务器。设立在用户端与源服务器之间，用来存放符合缓存规则（HTTP headers设置）的数据副本。

<br>

### 为什么要用 CDNs ？
> The goal of a CDNs is to serve content to end-users with high availability and high performance.

<br>

### DNS 解析
[什么是 DNS](/pages/2016/04/18/network-dns.html)   
当用户代理（浏览器）发起一个域名请求，DNS服务器会对请求进行解析查询并返回相应的IP地址。如果该域名请求由CDNs负责处理则返回相应CDN域名地址。   

<br>

### CDNs 内部 DNS 解析
> When a user makes a request to a CDN hostname, DNS will resolve to an optimized server (based on location, availability, cost, and other metrics) and that server will handle the request.    

当用户代理（浏览器）发起一个域名由CDN负责处理的DNS域名请求后，处理该DNS域名请求的CDN服务器会根据DNS解析器的IP地址做地理位置的查询，然后返回一个距离那个地理位置最近的一个CDN服务器的IP。

<br>

### 访问内容
端服务器是跟浏览器缓存工作方式类似的缓存代理。当一个请求来到端服务器，服务器首先检查缓存看一下请求的内容是不是存在。缓存的key是整个包括查询字符串的URL（正如在浏览器中一样）。如果内容在缓存中，并且缓存条目没有过期，那么这份内容就直接从端服务器中提供出去。可如果，内容不在缓存里或者缓存条目已经过期，那么端服务器发一个请求到源服务器去获取信息。源服务器能提供所有在CDN上有的内容。当端服务器从源服务器收到响应，它根据http响应头把内容存储在缓存里。

<br>
<br>

参考：[维基百科](https://en.wikipedia.org/wiki/Content_delivery_network)、[How CDNs work](https://www.nczonline.net/blog/2011/11/29/how-content-delivery-networks-cdns-work/)
