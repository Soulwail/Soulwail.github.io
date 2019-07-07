---
layout: post
title: XMLHttpRequest 属性解读
date: 2019-06-26
desc: "XMLHttpRequest 属性解读"
keywords: "XMLHttpRequest"
categories: [Http]
tags: [XMLHttpRequest]
icon:
---

## XMLHttpRequest 属性解读

首先在 Chrome console 下创建一个 XMLHttpRequest 实例对象 xhr。如下所示：

![XMLHttpRequest](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/ajax_01.png)

### inherit

试运行一下代码。

~~~javascript
var xhr = new XMLHttpRequest(), i = 0;

for (var key in xhr) {
  if (xhr.hasOwnProperty(key)) {
    i++;
  }
}

console.log(i); // 0
console.log(XMLHttpRequest.prototype.hasOwnProperty('timeout')); // true
~~~

可见 XMLHttpRequest 实例对象没有自由属性。实际上，它的所有属性均来自于 `XMLHttpRequest.prototype`。

追根溯源，XMLHttpRequest 实例对象具有如下的继承关系。（下面以 a << b 表示 a 继承 b）

`xhr` << `XMLHttpRequest.prototype` << `XMLHttpRequestEventTarget.prototype` << `EventTarget.prototype` << `Object.prototype`

由上，xhr 也具有 Object 等原型中的所有方法，如 toString 方法。

~~~javascript
xhr.toString(); // "[object XMLHttpRequest]"
~~~

通常，一个 xhr 实例对象拥有 10 个普通属性和 9 个方法。

### readyState {#ID4-2}

只读属性，readyState 属性记录了 ajax 调用过程中所有可能的状态。它的取值简单明了，如下：

|readyState|对应常量|描述|
|---|---|---|
| 0 (未初始化)|xhr.UNSENT|请求已建立，但未初始化（此时未调用 open 方法）|
| 1 (初始化)|xhr.OPENED|请求已建立，但未发送（已调用 open 方法，但未调用 send 方法）|
| 2 (发送数据)|xhr.HEADRS_RECEIVED|请求已发送（send 方法已调用，已收到响应头）|
| 3 (数据传送中)|xhr.LOADING|请求处理中，因响应内容不全，这时通过 responseBody 和 responseText 获取可能会出现错误|
| 4 (完成)|xhr.DONE|数据接受完毕，此时可以通过 responseBody 和 responseText 获取完整的响应数据|

注意，readyState 是一个只读属性，想要改变它的值是不可行的。

*待补充。。。*

### 参考文档

[Ajax知识体系大梳理](http://louiszhai.github.io/2016/11/02/ajax/ "hp")
