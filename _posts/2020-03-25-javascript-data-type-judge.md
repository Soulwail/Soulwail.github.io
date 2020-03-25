---
layout: post
title: "判断 JS 数据类型"
date: 2020-03-25
desc: ""
keywords: "javascript, 数据类型"
categories: [Js]
tags: [Js]
icon: icon-javascript
---

首先来提一个问题：`typeof` 是否能正确判断类型？

答案是：不可以。因为历史原因，在判断原始类型时，`typeof null` 会等于 `object`。而且对于对象来说，除了函数，都会转换成 `object`。例如：

```javascript
typeof 1; // 'number'
typeof "1"; // 'string'
typeof null; // 'object'

typeof []; // 'object'
typeof {}; // 'object'
typeof window.alert; // 'function'
```

在来提问一个，`instanceof` 是否能正确判断类型？

答案是：还是不可以，虽然 `instanceof` 是通过原型链来判断的，但是对于对象来说，`Array` 也会被转换成 `object`，而且也不能区分基本类型 `string` 和 `boolean`。例如：

```javascript
function Func() {}
const func = new Func();
console.log(func instanceof Func); // true

const obj = {}
const arr = []
obj instanceof Object; // true
arr instanceof Object; // true
arr instanceof Array; // true

const str = "abc";
const str2 = new String("abc");
str instanceof String; // false
str2 instanceof String; // true
```

所以该怎么办呢？

这时候我们可以使用：`Object.prototype.toString.call()`

所以为什么？

因为每个对象都有一个 `toString()` 方法，当要将对象表示为文本值或以预期字符串的方式引用对象

时，会自动调用该方法。默认情况下，从 object 派生的每个对象都会继承 toString() 方法。如果此方法未在自定义对象中被覆盖，则 toString() 返回 [object type]，其中 type 是对象类型。所以就有一下例子：

```javascript
Object.prototype.toString.call(new Date()); // [object Date]
Object.prototype.toString.call("1"); // [object String]
Object.prototype.toString.call(1); // [object Number]
Object.prototype.toString.call(undefined); // [object Undefined]
Object.prototype.toString.call(null); // [object Null]
```

所以综合上述知识点我们可以封装出一下通用类型判断方法：

```javascript
var type = function(data) {
  var toString = Object.prototype.toString;
  var dataType = data instanceof Element ?
                  'element' : // 为了统一 DOM 节点
                  toString.call(data).match(/\[object (.*?)\]/)[1].toLowerCase()
  return dataType
}
```

使用方法如下：

```javascript
type("a"); // string
type(1); // number
type(window); // window
type(document.querySelector("h1")); // element
```

