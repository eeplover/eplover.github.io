---
layout: post
title: 几道有趣的js面试题
categories: pages
excerpt: 前端面试题
tags: javascript
---

#### 题目一
实现一个lazyMan方法，有如下要求：    
lazyMan('Hank')   
// 输出   
// Hi! This is Hank   

lazyMan('Hank').eat('banana')   
// 输出   
// Hi! This is Hank   
// Eat banana   

lazyMan('Hank').sleep(10).eat('banana')   
// 输出    
// Hi! This is Hank   
// // 等候10秒   
// Wake up after 10 s   
// Eat banana   

```javascript
function LazyMan(name) {
  var i = this
  var fn = function() {
    console.log(`Hi! This is ${name}`);
    i.next()
  }
  i.eventsArray = [fn]
  setTimeout(function() {
    i.next()
  }, 0)
}

LazyMan.prototype.next = function () {
  var fn = this.eventsArray.shift()
  fn && fn()
}

LazyMan.prototype.sleep = function(time) {
  var i = this
  var fn = function() {
    console.log(`//等候${time}秒`);
    setTimeout(function () {
      console.log(`Wake up after ${time}s`);
      i.next()
    }, time * 1000)
  }
  this.eventsArray.push(fn)
  return this
}

LazyMan.prototype.eat = function(food) {
  var i = this
  var fn = function() {
    console.log(`Eat ${food}`);
    i.next()
  }
  this.eventsArray.push(fn)
  return this
}

function lazyMan(name) {
  return new LazyMan(name)
}
```


#### 题目二
2个数字字符串的相加，即 '19' + '1' --> '20'。

```javascript
// 考虑到大数相加
function add(str1, str2) {
  var a1 = Array.from(str1).reverse()
  var a2 = Array.from(str2).reverse()
  var maxLen = Math.max(a1.length, a2.length)
  var baseNum = 0
  var result = ''
  for (var i = 0; i < maxLen; i++) {
    a1[i] = +a1[i] || 0
    a2[i] = +a2[i] || 0
    result = (baseNum + a1[i] + a2[i]) % 10 + result
    baseNum = (baseNum + a1[i] + a2[i]) > 9 ? 1 : 0
  }
  if (baseNum)
    result = baseNum + result
  return result
}
```

#### 题目三
存在值均为整数的数组[17,2,-10,0,58...]，要求输出任意两项值相加等于50的选项索引。

```javascript
function figureout(arr) {
  var i = 0
  var cache = {}
  var result = []
  for (; i < arr.length; i++) {
    if (cache[50 - arr[i]] !== undefined)
      result.push([cache[50 - arr[i]], i])
    else
      cache[arr[i]] = i
  }
  return result
}
```

#### 题目四
创建一个长度为100，值等于索引的数组。不用for循环~

Array(100).fill(1).map((v,i) => i)

#### 题目五
实现一个订阅/发布模式
```javascript
function Event() {
  this._events = {}
}

Event.prototype = {
  on: function(event, listener) {
    if (!event || typeof event !== 'string' || typeof listener !== 'function')
      return false
    var listeners = this._events[event] || (this._events[event] = [])
    var registerId = listener._rid
    if (!listeners[registerId])
      listener._rid = listeners.length, listeners.push(listener)

    return this
  },
  emit: function(event) {
    var args = Array.prototype.slice.call(arguments, 1)
    this._events[event].forEach(listener => {
      listener.apply(this, args)
    })
  },
  off: function(event, listener) {
    var listeners = this._events[event]
    if (!listeners || listeners.length === 0 || (listener && listener._rid === undefined))
      return this
    if (!listener) {
      this._events[event] = []
      return this
    }
    listeners.splice(listener._rid, 1)
  }
}
```

#### 题目六
二分法查询
```javascript
function binarySearch(arr, value, sIndex, eIndex) {
  if (!Array.isArray(arr) || !arr.length) return -1
  if (typeof sIndex === 'undefined') sIndex = 0
  if (typeof eIndex === 'undefined') eIndex = arr.length - 1
  var mIndex = Math.round((sIndex + eIndex)/2)

  if (arr[mIndex] === value) {
    return mIndex
  } else if (arr[mIndex] > value) {
    return arguments.callee(arr, value, sIndex, mIndex - 1)
  } else {
    return arguments.callee(arr, value, mIndex + 1, eIndex)
  }
}
```

#### 题目七
实现深度复制
```javascript
function clone(obj1, obj2) {
  obj1 = obj1 || {}
  for (var p in obj2) {
    if (Array.isArray(obj2[p]))
      obj1[p] = clone([], obj2[p])
    else if (typeof obj2[p] === 'object')
      obj1[p] = clone({}, obj2[p])
    else
      obj1[p] = obj2[p]
  }
  return obj1
}
```
