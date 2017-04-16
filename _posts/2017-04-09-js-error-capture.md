---
layout: post
title: JavaScript异常监控【待完善】
categories: pages
excerpt: JavaScript异常监控
tags: JavaScript
---

# 采集

### window.onerror

当Js中抛出一个未被捕获的异常时，会在window上触发error事件，并执行window.onerror()。

**语法**

```JavaScript
window.onerror = function (message, source, lineNo, columnNo, error) {
  ...
}
```

**函数参数**

* message：错误信息（String）。如： “Uncaught ReferenceError: foo is not defined”
* source：发生错误的脚本URL（String）
* lineNo：发生错误处的行号（Number）
* columnNo：发生错误处的列号（Number）
* error：Error对象（Object）

window.error可不是什么新东西，从IE6和FireFox2就开始支持了。但是各浏览器对它的实现存在差异。

| Browser | message | source | lineNo | columnNo | error |
| --- | --- | --- | --- | --- | --- |
| Firefox 42 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| Chrome 46 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| Android Browser 4.4 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| Edge | ✔️ | ✔️ | ✔️ | ✔️ |  |
| IE 11 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| IE 10 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| IE 9, 8 | ✔️ | ✔️ | ✔️ |  |  |
| Safari 9 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| iOS 9 | ✔️ | ✔️ | ✔️ | ✔️ |||


> 这其中最有用的无疑是第五个参数，Error对象。接下来我们来看看它为什么最有用

**Error 对象**

当Js运行的过程中产生错误时，Error对象的实例就会被抛出。通过Error的构造器可以创建一个自定义的错误实例。

```JavaScript
new Error([message[, filename[, lineNumber]]])
```

[进一步了解Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)

Error有一个非标准属性 **Error.prototype.track**。track属性（错误追踪栈）会告诉你错误内容以及错误发生的具体位置，就像下面这个。

```JavaScript
// from Chrome46
Error: foobar
    at new bar (<anonymous>:241:11)
    at foo (<anonymous>:245:5)
    at callFunction (<anonymous>:229:33)
    at Object.InjectedScript._evaluateOn (<anonymous>:875:140)
    at Object.InjectedScript._evaluateAndWrap (<anonymous>:808:34)
```

这对于错误的调试来说真的是太重要了！  

由于不是标准属性，各个浏览器对它的实现自然差异。就像同样的错误追踪栈信息，下面是来自IE11上的。

```JavaScript
// from IE11
Error: foobar
   at bar (Unknown script code:2:5)
   at foo (Unknown script code:6:5)
   at Anonymous function (Unknown script code:11:5)
   at Anonymous function (Unknown script code:10:2)
   at Anonymous function (Unknown script code:1:73)
```

对比发现，二者除了格式上的区别外，追踪错误的细致程度也不同，Chrome46的追踪更细致。

**标准化track**

这时候，急需一个工具能够跨浏览器实现track属性的标准化...

* [TrackKit](https://github.com/occ/TraceKit)
* [stacktrace.js](https://www.stacktracejs.com/)

# 上报
...
