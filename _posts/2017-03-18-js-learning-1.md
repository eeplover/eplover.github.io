---
layout: post
title: js的一些基本概念
categories: pages
excerpt: js的基本概念
tags: javascript
---

### JavaScript 数据类型和数据结构
最新的ECMAScript定义了七种数据类型    
6种原始类型    
1. Boolean
2. Number
3. String
4. Null
5. Undefined
6. Symbol （EAMAScript 6 新定义）

和Object

#### 数字类型
Number是双精度64位二进制格式的值，取值 Number.MIN_VALUE ~ Number.MAX_VALUE ，也可以用ES6里的Number.isSafeInteger()方法或者Number.MAX_SAFE_INTEGER 和 Number.MIN_SAFE_INTEGER 来检查值是否在双精度浮点数的取值范围内。

#### 符号类型
Symbol 是一种特殊的、不可变的数据类型，可以作为对象属性的标识符使用。    

应用场景1：当你需要查询某个对象是否在数组中，你可以为每个对象设置一个Symbol属性，并设置上相应的值。    

应用场景二：想要查询某个对象中的一个属性来确定当前的状态的时候可以用到Symbol。   

用Symbol作为属性值的好处是：它不会与对象的某个属性命名冲突，也不会在for in或者JSON.stringify对象的时候'污染'对象。当然，你可以通过Object.getOwnPropertySymbols()来查找指定对象的符号属性。

#### Map
Map对象就是简单的键/值映射。其中键和值可以是任意值(对象或者原始值)。   
如果你不确定要使用哪个，请思考下面的问题：

* 在运行之前 key 是否是未知的，是否需要动态地查询 key 呢？
* 是否所有的值都是统一类型，这些值可以互换么？
* 是否需要不是字符串类型的 key ？
* 键值对经常增加或者删除么？
* 是否有任意个且非常容易改变的键值对?
* 这个集合可以遍历么(Is the collection iterated)?    

假如以上全是“是”的话，那么你需要用 Map 来保存这个集。 相反，你有固定数目的键值对，独立操作它们，区分它们的用法，那么你需要的是对象。
#### Set
集合（Set）对象允许你存储任意类型的唯一值（不能重复），无论它是原始值或者是对象引用。
