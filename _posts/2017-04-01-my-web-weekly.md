---
layout: post
title: My Web Weekly
categories: pages
excerpt: My Web Weekly
tags: front-end,html,javascript,css,node
---

小铁前段时间看了个脱口秀节目👉《矮大紧奇谈》，其中讲到日本“道”文化的守、破、离，颇有意思。同理，技术学习大概也要经历这三个阶段：造轮子是守，对轮子的改造是破，最后丢弃旧轮子发明新轮子的过程便是离 👏👏 ...   

呃哼.. 呃哼~ 在技术学习的过程中，最忌闭门造车，一定要多关注前端的最新技术和资讯。比如说多上这些网站看看  

* [GitHub Trending](https://github.com/trending?l=javascript)
* [Echo JS](http://www.echojs.com/)
* [hashnode](https://hashnode.com/amas)
* [medium](https://medium.com/)
* [Smashing Magazine](http://www.smashingmagazine.com/)
* [Web Platform Daily](http://webplatformdaily.org/)
* [JavaScript Weekly](http://javascriptweekly.com/)
* [Node Weekly](http://nodeweekly.com/)
* [HTML5 Weekly](http://html5weekly.com/)
* [Mobile Web Weekly](http://mobilewebweekly.co/)
* [Web Operations Weekly](http://webopsweekly.com/)
* [Web Design Weekly](https://web-design-weekly.com/)
* [CSS Weekly](http://css-weekly.com/)
* [CSS-TRICKS](https://css-tricks.com/)
* [Chromium Status](http://www.chromestatus.com/features)

>ps：逼装完了，该回到现实中了吧小铁~      

这么多别说一一看下来了，对于懒惰的小铁来说要挨个点下来都费劲。所以小铁决定想办法将这些内容进行聚合起来。

于是乎，有了 **[my-web-weekly](https://github.com/eplover/my-web-weekly)** 这东西。该项目借助爬虫抓取指定网页的数据，进行处理汇总，以邮件的方式通知你去阅读。

![web weekly 邮件内内容截图](https://cloud.githubusercontent.com/assets/11499979/24580263/a3f5e65c-1737-11e7-899c-41e8b1eaedb0.png)

<br>

### 网络爬虫（web crawler）

也叫网络蜘蛛（spider），是一种用来自动浏览万维网的网络机器人。

![spider](https://cloud.githubusercontent.com/assets/11499979/24576072/9ead9d02-16e7-11e7-81e7-74de2828c941.gif)

网络爬虫可以将自己所访问的页面保存下来，以便搜索引擎事后生成索引供用户搜索。

嘿嘿，爬虫！！听着就感觉很牛逼~  

NodeJS单线程、事件驱动的特性可以在单台机器上实现极大的吞吐量，非常适合写网络爬虫这种资源密集型的程序。

不过... 这里我们只是抓取几个事先配置好的页面，而且扒取效率不做严格要求，所以借用 [request](https://github.com/request/request) 和 [cheerio](https://github.com/cheeriojs/cheerio) 来实现数据抓取功能，[nodemailer](https://github.com/nodemailer/nodemailer)和[email-templates](https://github.com/crocodilejs/node-email-templates)完成邮件的生成与发布。具体实现欢迎大家去 **[my-web-weekly](https://github.com/eplover/my-web-weekly)** 看源码喽

是不是有种雷声大雨点小的感觉~ 哈哈   

各位看官且给小铁些许时间，小铁他日必定奉上深度分享。
