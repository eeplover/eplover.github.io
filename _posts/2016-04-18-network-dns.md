---
layout: post
title: 浅尝 DNS
categories: pages
excerpt: DNS原理
tags: network  
---
### 背景
目前的流行网络协议是 TCP/IP 协议，其中的 IP 为第四版的 IPv4。这个IPv4的字长32位，为了方便人脑记忆已经转成四组十进制的数字了。例如 172.18.16.20 这样的格式。当我们利用Internet传输资料的时候，需要用到这个IP，否则资料封包怎么知道要被送到哪里去？然而对于IP的这种数字的玩意儿，记忆力实在是不怎么样。但是要上Internet一定需要IP，怎么办？    
为了解决这个问题，早期的做法是利用某些特定的档案将主机名称与IP做一个对应，如此一来，我们就可以通过主机名称来取得该主机的IP了。人脑对于名字的记忆力可就好多了，那就是 /etc/hosts 这个档案的用途了。   
可惜的是，这个方法还是有缺憾。那就是主机名称与IP的对应修改无法实时同步所有主机，所以用户端电脑每次都要重新下载一次档案才能顺利联网。随着电脑普及数量增多，这个问题日益严重。   
为了解决这个问题，伯克莱大学发展处一套阶层式管理主机名称对应IP的系统，我们称它为 Berkeley Internet Name Domain，BIND。也就是目前全世界使用广泛的域名系统（Domain Name System,DNS）。

<br>

### 什么是DNS ?
Domain Name System.

<br>

### 为什么会有DNS ？
IP 难记，域名好记。

<br>

### 工作原理
当你在浏览器网址框输入 http://www.ksu.edu.tw 时，先去查询本地的 /etc/hosts 这个档案，未找到对应IP就会依据相关设定（在Linux下就是利用 /etc/resolv.conf 这个档案）所提供的 DNS 的 IP 去进行连线查询了。这个服务器（A表示）就会这样工作：

1. 收到用户的查询请求，先查看本身有没有记录，无则向 . 查询    
   由于DNS是阶层式架构，每部主机只负责管理自己下一级主机。
2. 向最顶层的 .(root)服务器 查询    
   A 会主动向 .(root)服务器 询问 http://www.ksu.edu.tw 在哪里呢？但是 .服务器 只记录了 .tw 的资讯(因为台湾只有.tw向.注册而已)。此时会告知：“我不知 这部主机的ip啦，你应该去问 .tw服务器，我这里不管！我跟你说.tw在哪里吧”

3. 向第二层的 .tw服务器 查询    
   A 接着向.tw询问 http://www.ksu.edu.tw 在哪里呢？而.tw管理的又仅有 .edu.tw、.com.tw、.gov.tw ... 这几部主机，经过查询比较找到了.edu.tw的网域。所以这时候.tw又告诉A说：“你要去管理.edu.tw这个网域的主机那里去查询，我有它的IP！”。
4. 向第三层的 .edu.tw服务器 查询    
   同理可证，.edu.tw又告诉A应该要去.ksu.edu.tw进行查询，这里只能告知.ksu.edu.tw的IP而已。
5. 向第四层的 .ksu.edu.tw服务器 查询    
   等A找到.ksu.edu.tw之后，BINGO！.ksu.edu.tw说：“没错，这部主机就是我管理的，我给你说它的IP吧”。所以，A就能够查到 www.ksu.edu.tw的IP喽。
6. 缓存查询结果并返回   
   记录了正确的IP后，A的DNS机器不会在下次有人查询http://www.ksu.edu.tw 的时候再走一次查询流程。它会很聪明的先把查询结果缓存，以方便回应下一次的相同要求。最后将结果回报给client端。当然，那个缓存的查询结果是时间性的，当过了DNS 设定的时间（通常是24小时），那缓存就会被释放。

可用 dig +trace http://www.ksu.edu.tw 来追踪查询细节。   
[在Mac上如何清理DNS cache ？](https://coolestguidesontheplanet.com/clear-the-local-dns-cache-in-osx/)
<br>
<br>

参考：[DNS服务器](http://linux.vbird.org/linux_server/0350dns.php)
