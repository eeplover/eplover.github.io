---
layout: post
title: JavaScript异常监控
categories: pages
excerpt: JavaScript异常监控
tags: JavaScript
---

> 摘来 [JSTracker：前端异常数据采集](http://taobaofed.org/blog/2015/10/28/jstracker-how-to-collect-data/) 中的一段话，来说明 JavaScript 异常监控的必要性。我四不四很懒 🙄🙄，哈哈

基本上服务器端的代码都是处于 7x24 小时的实时监控状态的，一旦有任何异常对应的开发同学就马上收到报警，并且第一时间处理。 但是对于前端来说，往往是实际用户那里的脚本报错后才知道页面出现异常，这时候已经是故障了。

为了让前端也能和后端一样，需要将线上的 JavaScript 代码监控起来，当用户端浏览器出现异常前端第一时间被通知到。


<br>

> 意识到监控的必要性后，我又做了什么呢？

接下来我看了这些个资料：Sentry上的 [Capture and report JavaScript errors with window.onerror](https://blog.sentry.io/2016/01/04/client-javascript-reporting-window-onerror.html)、淘宝FED的两篇博客 [JSTracker：前端异常数据采集](http://taobaofed.org/blog/2015/10/28/jstracker-how-to-collect-data/) 和 [JSTracker：异常数据处理](http://taobaofed.org/blog/2015/11/06/jstracker-data-processing/)、企鹅的 [badJs](https://github.com/BetterJS) 以及百姓网@刘小杰在infoQ上的一篇演讲 [浏览器端 JavaScript 异常监控](http://www.infoq.com/cn/presentations/javascript-exception-monitoring-for-dummies-in-browser-side)。

顺带撸了 [TrackKit](https://github.com/occ/TraceKit) 和 [stacktrace.js](https://www.stacktracejs.com/) 的源码 ...

<br>

>  👏👏 那 ... 这些资料都讲了什么呀？

好的，🍰来喽。接下来我们结合 TraceKit 的实现来聊一聊 `JavaScript的异常监控`。

异常监控分为两部分：

* 异常信息的捕获与处理
* 异常信息的上报

<br>

# 异常信息的捕获与处理

异常信息的捕获，想必大家都知道下面这种方式。

```javascript
try {
    // 可能报错的内容放这里头
} catch (e) {
    // 异常信息处理
}
```

但这种方式足够灵活，适用于我们手动捕获。但它捕获不了 try 块内的异步代码抛出的异常，所以我们来看看另外一种捕获方式~

# window.onerror

当 Js 中抛出一个未被捕获的异常（包括异步代码中抛出的未被捕获的异常）时，会在 window 上触发 error 事件，并执行 window.onerror()。

**语法**

```javascript
window.onerror = function (message, source, lineNo, columnNo, error) {
  ...
}
```
由上可知，该事件的回调接受如下 5 个参数。

* message：错误信息（String）。如： “Uncaught ReferenceError: foo is not defined”
* source：发生错误的脚本URL（String）
* lineNo：发生错误处的行号（Number）
* columnNo：发生错误处的列号（Number）
* error：Error 对象（Object）

> window.error 可不是什么新东西，从 IE6 和 FireFox2 就已经开始支持了。

但是各浏览器对它的实现存在如下差异。  

<br>

| Browser | message | source | lineNo | columnNo | error |
| --- | :---: | :---: | :---: | :---: | :---: |
| Firefox 42 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| Chrome 46 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| Android Browser 4.4 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| Edge | ✔️ | ✔️ | ✔️ | ✔️ |  |
| IE 11 | ✔️ | ✔️ | ✔️ | ✔️ | ✔️ |
| IE 10 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| IE 9, 8 | ✔️ | ✔️ | ✔️ |  |  |
| Safari 9 | ✔️ | ✔️ | ✔️ | ✔️ |  |
| iOS 9 | ✔️ | ✔️ | ✔️ | ✔️ |  | |

<br>

> 这其中最有用的无疑是第五个参数，Error 对象。

接下来我们就来看看它为什么 **Error 对象** 最有用。

当 Js 运行的过程中产生错误时，Error 对象的实例就会被抛出。通过 Error 的构造器可以创建一个自定义的错误实例。

```javascript
new Error([message[, filename[, lineNumber]]])
```

[进一步了解Error](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)

Error 有一个非标准属性 **Error.prototype.track**。track 属性（错误追踪栈）会告诉你错误内容以及错误发生的具体位置，就像下面这个。

```
// from Chrome46
Error: foobar
    at new bar (<anonymous>:241:11)
    at foo (<anonymous>:245:5)
    at callFunction (<anonymous>:229:33)
    at Object.InjectedScript._evaluateOn (<anonymous>:875:140)
    at Object.InjectedScript._evaluateAndWrap (<anonymous>:808:34)
```

这对于错误的调试来说真的是太重要了，四不四！！  

但是，由于不是标准属性，各个浏览器对它的实现存在差异。

就像同样的错误追踪栈信息，下面是来自 IE11 上的。

```
// from IE11
Error: foobar
   at bar (Unknown script code:2:5)
   at foo (Unknown script code:6:5)
   at Anonymous function (Unknown script code:11:5)
   at Anonymous function (Unknown script code:10:2)
   at Anonymous function (Unknown script code:1:73)
```

对比发现，二者除了格式上的区别外，追踪错误的细致程度也不同。

TraceKit 内部的 computeStackTrace 方法就是用来 **标准化track** 的。

```javascript
// 以下是一些主要代码
TraceKit.computeStackTrace = (function computeStackTraceWrapper() {
    // 获取源码
    function getSource(url) {
        // step 1: ajax 的方式获取源码
        // step 2: 按 `\n` 拆分成数组，并返回
    }

    // 猜测异常所在的方法
    function guessFunctionName(url, lineNo) {
        // step 1: 调用 getScource() 得到存放脚本内容的数组
        // step 2: 结合匹配函数的正则表达式，依次从 lineNo 递减到 lineNo - 10 行遍历数组
        // step 3: 返回匹配的函数名，未找到匹配项则返回 `?`。
    }

    // 收集异常的区域性内容
    function gatherContext(url, line) {
        // 返回内容数组索引从 line - n 到 line + n 的值，这个 n 可以手动配置
    }

    // 返回与正则匹配的字符串在源码中的位置
    function findSourceInUrls(reg, urls) {
        var source, m;
        for (var i = 0, j = urls.length; i < j; ++i) {
            // console.log('searching', urls[i]);
            if ((source = getSource(urls[i])).length) {
                source = source.join('\n');
                if ((m = re.exec(source))) {
                    // console.log('Found function in ' + urls[i]);

                    return {
                        'url': urls[i],
                        'line': source.substring(0, m.index).split('\n').length,
                        'column': m.index - source.lastIndexOf('\n', m.index) - 1
                    };
                }
            }
        }

        // console.log('no match');

        return null;
    }

    // 查找给定代码片段在源码中某一行的列数
    function findSourceInLine() {}

    // 针对不同浏览器标准化 stack
    function computeStackTraceFromStackProp(ex) {}
    function computeStackTraceFromStacktraceProp(ex) {}
    function computeStackTraceFromOperaMultiLineMessage(ex) {}
    function computeStackTraceByWalkingCallerChain(ex) {}

    // 追加一些有用的信息到 stack 的第一帧
    function augmentStackTraceWithInitialElement(ex) {}

    // stack 标准化主函数
    function computeStackTrace(ex, depth) {}

    return computeStackTrace
})
```

以上讲的都是异常信息的采集，下面我们来说说异常信息的上报。

<br>


# 异常信息的上报

```javascript
// 以下是一些主要代码
TraceKit.report = (function reportModuleWrapper() {
    var handlers = [],
        lastException = null,
        lastExceptionStack = null;

    // 添加异常处理函数
    function subscribe(handler) {
        installGlobalHandler();
        handlers.push(handler);
    }

    // 移除异常处理函数
    function unsubscribe(handler) {}

    // 分发异常到所有处理函数中进行处理
    function notifyHandlers(stack, isWindowError, error) {
        var exception = null;
        if (isWindowError && !TraceKit.collectWindowErrors) {
          return;
        }
        for (var i in handlers) {
            if (_has(handlers, i)) {
                try {
                    handlers[i](stack, isWindowError, error);
                } catch (inner) {
                    exception = inner;
                }
            }
        }

        if (exception) {
            throw exception;
        }
    }

    var _oldOnerrorHandler, _onErrorHandlerInstalled;

    // 确保所有全局的异常能够被捕获并处理
    function traceKitWindowOnError(message, url, lineNo, columnNo, errorObj) {
        var stack = null;

        if (lastExceptionStack) {
            TraceKit.computeStackTrace.augmentStackTraceWithInitialElement(lastExceptionStack, url, lineNo, message);
    	    processLastException();
	    } else if (errorObj) {
            stack = TraceKit.computeStackTrace(errorObj);
            notifyHandlers(stack, true, errorObj);
        } else {
            var location = {
              'url': url,
              'line': lineNo,
              'column': columnNo
            };
            location.func = TraceKit.computeStackTrace.guessFunctionName(location.url, location.line);
            location.context = TraceKit.computeStackTrace.gatherContext(location.url, location.line);
            stack = {
              'mode': 'onerror',
              'message': message,
              'stack': [location]
            };

            notifyHandlers(stack, true, null);
        }

        if (_oldOnerrorHandler) {
            return _oldOnerrorHandler.apply(this, arguments);
        }

        return false;
    }

    // 注册全局的异常处理函数
    function installGlobalHandler () {
        if (_onErrorHandlerInstalled === true) {
            return;
        }
        _oldOnerrorHandler = window.onerror;
        window.onerror = traceKitWindowOnError;
        _onErrorHandlerInstalled = true;
    }

    // 处理最近抛出的异常
    function processLastException() {
        var _lastExceptionStack = lastExceptionStack,
            _lastException = lastException;
        lastExceptionStack = null;
        lastException = null;
        notifyHandlers(_lastExceptionStack, false, _lastException);
    }

    // 异常上报函数
    function report(ex) {
        // 避免重复上报
        if (lastExceptionStack) {
            if (lastException === ex) {
                return; // already caught by an inner catch block, ignore
            } else {
              processLastException();
            }
        }

        var stack = TraceKit.computeStackTrace(ex);
        lastExceptionStack = stack;
        lastException = ex;

        // If the stack trace is incomplete, wait for 2 seconds for
        // slow slow IE to see if onerror occurs or not before reporting
        // this exception; otherwise, we will end up with an incomplete
        // stack trace
        window.setTimeout(function () {
            if (lastException === ex) {
                processLastException();
            }
        }, (stack.incomplete ? 2000 : 0));

        throw ex; // re-throw to propagate to the top level (and cause window.onerror)
    }

    report.subscribe = subscribe;
    report.unsubscribe = unsubscribe;
    return report;
}())
```

<br>

TraceKit 提供了 wrap 方法，用于回调函数内部异常的主动上报。

```javascript
TraceKit.wrap = function traceKitWrapper(func) {
    function wrapped() {
        try {
            return func.apply(this, arguments);
        } catch (e) {
            TraceKit.report(e);
            throw e;
        }
    }
    return wrapped;
};
```
