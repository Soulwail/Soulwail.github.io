---
layout: post
title: "JavaScript 之原型对象与原型链"
date: 2019-04-20
desc: ""
keywords: "javascript, prototype, 原型链"
categories: [Js]
tags: [Js, 原型链]
icon: icon-javascript
---

## `prototype` 和 `__proto__` 的区别

![prototype和__proto__的区别](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/prototype_01.png)

- `prototype` 是**函数**才有的属性
- `__proto__` 是每个**对象**都有的属性
- 但 `__proto__` 不是一个规范属性，只是部分浏览器实现了此属性，对应的标准属性是 `[[Prototypr]]`

**注意：** 大多数情况下，`__proto__` 可以理解为“**构造器的原型**”，即：`__proto__ === constructor.prototype`。（通过 `Object.create()` 创建的对象不适用此等式。下文会介绍）

```javascript
var a = {};
console.log(a.prototype); // undefined
console.log(a.__proto__); // Object {}

var b = function() {}
console.log(b.prototype); // b {}
console.log(b.__proto__); // function() {}
```

<br>

## `__proto__` 属性指向谁

`__proto__` 的指向取决于对象创建时的实现方式。

以下图表列出了三种常见方式创建对象后，`__proto__` 分别指向谁：

### 1、字面量方式

```javascript
var a = {}
// var a = new Object();
```

![字面量方式](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/prototype_02.png)

```javascript
console.log(a.constructor); // function Object() { [native code] } (即构造器 Object)
console.log(a.__proto__ === a.constructor.prototype); // true
```

### 2、构造器方式

```javascript
var A = function() {}
var a = new A();
```

![构造器方式](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/prototype_03.png)

```javascript
console.log(a.constructor); // function() {} (即构造器 function A)
console.log(a.__proto__ === a.constructor.prototype); // true
```

### 3、`Object.create()` 方法

```javascript
var a1 = { a: 1 }
var a2 = Object.create(a1);
```

![Object.create() 方式](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/prototype_04.png)

```javascript
console.log(a2.constructor); // function Object() { [native code] } (即构造器 Object)
console.log(a2.__proto__ === a1); // true
console.log(a2.__proto__ === a2.constructor.prototype); // false
```

<br>

## 什么是原型链

由于 `__proto__` 是任何对象都有的属性，而 js 里万物皆对象，所以会形成一条 `__proto__` 连起来的链条，递归访问 `__proto__` 必须最终到头，并且值是 `null`。

当 js 引擎查找对象的属性时，先查找对象本身是否存在该属性，如果不存在，会在原型链上查找，但是不会查找自身的 `prototype`。

```javascript
var A = function() {}
var a = new A();
```

![原型链](https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/prototype_05.png)

```javascript
console.log(a.__proto__); // Object {} (即构造器 function A 的原型对象)
console.log(a.__proto__.__proto__); // Object {} (即构造器 function Object 的原型对象)
console.log(a.__proto__.__proto__.__proto__); // null
```

## `instanceof`

### 用法

`instanceof` 主要的作用就是判断一个实例是否属于某种类型。

```javascript
let A = function () { }
let a = new A();
a instanceof A // true
```

当然，`instanceof` 也可以判断一个实例是否是其父类型或祖先类型的实例。

```javascript
let Grandfather = function () { }
let Father = function () { }
Father.prototype = new Grandfather();
let son = new Father();
son instanceof Grandfather // true
son instanceof Father // true
```

### 原理

```javascript
function newInstanceof(leftValue, rightValue) {
  let rightProto = rightValue.prototype; // 取右表达式的 prototype 值
  leftValue = leftValue.__proto__; // 取左边表达式的 __proto__ 的值
  while (true) {
    if (leftValue === null) {
      return false
    }
    if (leftValue === rightProto) {
      return true
    }
    leftValue = leftValue.__proto__
  }
}
```

其实 `instanceof` 主要的实现原理就是只要右边的 `prototype` 在左边变量的原型链上即可。因此，`instanceof` 在查找过程中会遍历左边变量的原型链，直到找到右边变量的 `prototype`，如果查找失败，则会返回 false，告诉我们左边变量并非右边的实例。

### 参考文档

[三张图搞懂JavaScript的原型对象与原型链](https://juejin.im/post/5835853f570c35005e413b19)
