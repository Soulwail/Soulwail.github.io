---
layout: post
title: JS 创建对象
date: 2019-04-17
desc: "JS 创建对象"
keywords: "JavaScript, 创建对象"
categories: [Js]
tags: [JS, 创建对象]
icon: icon-javascript
---

原文链接：alvinyuxt.github.io

- 工厂模式
- 构造函数模式
- 原型模式
- 构造函数和原型组合模式
- 动态原型模式
- 寄生构造模式
- 稳妥构造模式

首先解释几个概念：

> 1. 对象：下面例子中所有的 Bird 函数
> 2. 实例/对象实例：通过 `new Bird()` 或 `Bird()` 返回的对象，如 `var bird1 = new Bird()` 中的 person1
> 3. 原型对象 `Bird.prototype`

## 工厂模式

```javascript
function Bird() {
  var o = new Object();
  o.name = 'bird';
  o.fly = function() {
    console.log('fly');
  }
  return o;
}

var bird1 = Bird();
```

**优点：** 完成了返回一个对象的要求。

**缺点：**

1. 无法通过 `constructor` 识别对象，以为都是来自 `Object`，无法得知来自 `Bird`。
2. 每次通过 `Bird` 创建对象的时候，所有的 `fly` 方法都是一样的，但是却存储了多次，浪费资源。

## 构造函数模式

```javascript
function Bird() {
  this.name = 'bird';
  this.fly = function() {
    console.log('fly');
  }
}

var bird1 = new Bird();
```

**优点：**

1. 通过 `constructor` 或者 `instanceof` 可以识别对象实例的类别。
2. 可以通过 `new` 关键字来创建对象实例，更像⚪⚪语言中创建对象实例。

**缺点：** 多个实例的 `say` 方法都是实现一样的效果，但是却存储了很多次（两个对象实例的 `say` 方法是不同的，因为存放的地址不同）。

**注意：**

1. 构造函数模式隐式的在最后返回 `return this`，所以在缺少 `new` 的情况下，会将属性和方法添加给全局对象，浏览器端就会添加给 `window` 对象。
2. 也可以根据 `return this` 的特性调用 `call` 或者 `apply` 指定 `this`。这一点在后面的继承有很大帮助。

## 原型模式

```javascript
function Bird() {}
Bird.prototype.name = 'bird';
Bird.prototype.fly = function() {
  console.log('fly');
}
Bird.prototype.feather = ['warm']

var bird1 = new Bird();
```

**优点：**

1. `fly` 方法是共享的，所有的实例的 `fly` 方法都指向同一个。
2. 可以动态的添加原型对象的方法和属性，并且直接反应在对象实例上。

  ```javascript
  var bird1 = new Person()
  Bird.prototype.showFeather = function() {
    console.log(this.feather)
  }
  bird1.showFeather(); // warm
  ```

**缺点：**

1. 出现引用的情况下会出现问题，具体情况见下面代码：

  ```javascript
  var bird1 = new Bird();
  var bird2 = new Bird();
  bird.feather.push('colorful');
  console.log(bird2.feather) // ['warm', 'colorful']
  ```

因为 js 对引用类型的赋值都是将地址存储在变量中，所以 `bird1` 和 `bird2` 的 `feather` 属性指向的是同一块存储区域。

2. 第一次调用 `fly` 方法或者 `name` 属性的时候会搜索两次，第一次是在实例上寻找 `fly` 方法，没有找到就去原型对象（`Bird.prototype`）上找 `fly` 方法，找到后就会在实例上添加这些方法或者属性。

3. 所有的方法都是共享的，没有办法创建实例自己的属性和方法，也没有办法像构造函数那样传递参数。

**注意：** 优点 ② 中存在一个问题就是直接通过对象字面量给 `Bird.prototype` 进行赋值的时候会导致 `constructor` 改变，所以需要手动设置，其次就是通过对象字面量给 `Bird.prototype` 进行赋值，会无法作用在之前创建的对象实例上。

```javascript
var bird1 = new Bird();
Bird.prototype = {
  name = 'bird2';
  setName: function(name) {
    this.name = name
  }
}
bird1.setName('bird3'); // Uncaught TypeError: bird1.setName is not a function
```

这是因为对象实例和对象原型直接是通过一个指针连接的，这个指针是一个内部属性 `[[Prototype]]`，可以通过 `__proto__` 访问。我们通过对象字面量修改了 `Bird.prototype` 指向的地址，然而对象实例的 `__proto__` 并没有跟着一起更新，所以这就导致实例还在访问着原来的 `Bird.prototype`，所以建议不要通过这种方式去改变 `Bird.prototype` 属性。

## 构造函数和原型组合模式

```javascript
function Bird(name) {
  this.name = name
}
Bird.prototype.fly = function() {
  console.log('fly')
}

var eagle = new Bird('eagle');
eagle.fly(); // fly
```

**优点：**

1. 解决了原型模式对于引用对象的缺点
2. 解决了原型模式没有办法传递参数的缺点
3. 解决了构造函数模式不能共享方法的缺点

**缺点：** 和原型模式中注意点一样

## 动态原型模式

```javascript
function Bird(name) {
  this.name = name
  if(typeof this.fly != 'function') {
    Bird.prototype.fly = function() {
      console.log('fly')
    }
  }
}
```

**优点：**

1. 可以在初次调用构造函数的时候就完成原型对象的修改
2. 修改能体现在所有的实例中

**缺点：**

**注意：**

1. 只用检查一个在执行后应该存在的方法或者属性就行了。
2. 不能用对象字面量修改原型对象。

## 寄生构造函数模式

```javascript
function Bird(name) {
  var o = new Object();
  o.name = name;
  o.fly = function() {
    console.log('fly');
  }
  return o;
}

var bird1 = new Bird('bird');
```

**优点：** 和工厂模式基本一样，除了多了个 `new` 操作符。

**缺点：** 和工厂模式一样，不能区分实例的类别。

## 稳妥构造模式

```javascript
function Bird(name) {
  var o = new Object();
  o.say = function() {
    console.log(name)
  }
  return o
}

var bird1 = new Bird('bird');
bird1.name; // undefined
bird1.say(); // bird
```

**优点：** 安全，`name` 成为了私有变量，只能通过 `say` 方法去访问。

**缺点：** 不能区分实例的类别。