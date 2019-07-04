---
layout: post
title: JavaScript 之构造函数
date: 2019-04-18
desc: JavaScript 之构造函数
keywords: "javascript, constructor, 构造函数"
categories: [Js]
tags: [Js, 构造函数]
icon: icon-javascript
---

典型的面向对象编程语言（比如 C++ 和 Java），存在 “类”（class）这个概念。所谓 “类”，就是对象的模板，对象就是 “类” 的实例。但是，在 JavaScript 语言的对象体系，不是基于 “类” 的，而是基于构造函数（constructor）和原型链（prototype）。

## 对象的概念

“面向对象编程”（Object Oriented Programming，缩写为 OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。

在有了对象的基础知识之后，对 js 中对象做一个简单的总结。如下：\\
**a: 对象是单个实物的抽象。**\\
**b: 对象是一个容器，封装了 ‘属性’ 和 ‘方法’。**

一本书、一辆汽车、一个人都可以是 “对象”。当实物被抽象成 “对象”。当实物被抽象成 “对象”，实物之间的关系就变成了 “对象” 之间的关系，从而就可以模拟现实情况，针对 “对象” 进行编程。

所谓属性，就是对象的一种状态；所谓方法，就是对象的一种行为。

比如说，可以把动物抽象为 animal 对象，属性记录的就是哪一种动物，以及动物的大小和颜色等，方法表示该动物的某种行为（奔跑、猎食、休息等等）。

## 构造函数

“面向对象编程” 的第一步，就是要生成对象。而 js 中面向对象编程是基于构造函数（constructor）和原型链（prototype）的。

前面说过，“对象” 是单个实物的抽象。通常需要一个模板，表示某一类实物的共同特征，然后 “对象” 根据这个模板生成。

js 语言中使用构造函数（constructor）作为对象的模板。所谓构造函数，就是提供一个生成对象的模板，并描述对象的基本结构的函数。一个构造函数，可以生成多个对象，每个对象都有相同的结构。

看一下构造函数的基本结构：

```javascript
// 两种写法相同
var Animal = function() {
  this.height = 80;
};

function Animal() {
  this.height = 80;
}
```

上面代码中，`Animal` 就是构造函数，它提供模板，用来生成对象实例。为了与普通函数区别，构造函数名字的第一个字母通常大写。

构造函数的三大特点：\\
**a：构造函数的函数名的第一个字母通常大写。**\\
**b：函数体内使用 this 关键字，代表所要生成的对象实例。**\\
**c：生成对象的时候，必须使用 new 命令来调用构造函数。**

## new 命令

### 基本原理

new 命令的作用，就是执行一个构造函数，并且返回一个对象实例。使用 `new` 命令时，它后面的函数调用就不是正常的调用。

```javascript
var dog = new Animal();
```

实际上内部是这样的：

```javascript
var dog = {};
dog.__proto__ = Animal.prototype;
Animal.call(dog); // 函数实例的 call 方法，可以指定函数内部 this 的指向
```

可以解释为下面的步骤：\\
**a：创建一个空对象，作为将要返回的对象实例。**\\
**b：将空对象的原型指向了构造函数的 prototype 属性。**\\
**c：将空对象赋值给构造函数内部的 this 关键字。**\\
**d：开始执行构造函数内部的代码。**

也就是说，构造函数内部，this 指向的是一个新生成的空对象，所有针对 this 的操作，都会发生在这个空对象上。构造函数之所以为构造函数，意思是这个函数的目的就是操作一个空对象（即 this 对象），将其构造为需要的样子。

### 基本用法

new 命令的作用，就是调用一个构造函数，并且返回一个对象的实例。

```javascript
function Animal() {
  this.height = 80;
}

var dog = new Animal();
console.log(dog.height); // 80
```

上面的代码中通过 new 命令，让构造函数 Animal 生成一个对象实例，并且赋值给全局变量 dog。这个新生成的对象实例，从构造函数 Animal 中继承 height 属性，也就说明了这个对象实例是没有 height 属性的。在 new 命令执行时，就代表了新生成的对象实例 dog。`this.height` 表示对象实例有一个 `height` 属性，它的值是 180。

使用 `new` 命令时，根据需要，构造函数也可以接受参数。

```javascript
function Animal(name, height) {
  this.name = name;
  this.height = height;
}

var dog = new Animal('大毛', 80);
console.log(dog.name); // '大毛'
console.log(dog.height); // 80

var cat = new Animal('阿黄', 30);
console.log(cat.name); // '阿黄'
console.log(cat.height); // 30
```

用以上的一个例子，来对构造函数的特点和 new 基本原理进行一个梳理。

上面代码中，首先，我们创建了一个构造函数 Animal，传入了两个参数 name 和 height。构造函数 Animal 内部使用了 this 关键字来指向将要生成的对象实例。

然后，我们使用 new 命令来创建了两个对象实例 dog 和 cat。

当我们使用 new 来调用构造函数时，new 命令会创建一个控对象 dog，作为将要返回的实例对象。接着，这个空对象的原型会指向构造函数的 Animal 的 prototype 属性。也就是 `dog.__proto__ === Animal.prototype` 的。要注意的是，空对象指向构造函数 Animal 的 prototype 属性，而不是指向构造函数本身。然后，我们将这个空对象赋值给构造函数内部的 this 关键字。也就是说，让构造函数内部的 this 关键字指向一个对象实例。最后，开始执行构造函数内部代码。

因为对象实例 dog 和 cat 是没有 name 和 height 属性的，所以对象实例中的两个属性都是继承自构造函数 Animal 中的。这也就是说明了构造函数是生成对象的函数，是给对象提供模板的函数。

一个问题，如果我们忘记使用 new 命令来调用构造函数，直接调用构造函数了，会发生什么？

这种情况下，构造函数就变成了普通函数，并不会生成实例对象。而且 `this` 这时代表全局对象，将会造成一些意想不到的结果。

```javascript
function Animal() {
  this.height = 80;
}

var dog = Animal();
console.log(dog.height); // TypeError: dog is undefined
console.log(window.height); // 80
```

上面的代码中，当在调用构造函数 Animal 时，忘记加上 new 命令，结果是 this 指向了全局作用域，height 也就变成了全局变量。而变量 dog 变成了 undefined。

因此，应该非常小心，避免出现不使用 `new` 命令、直接调用构造函数的情况。

为了保证构造函数必须与 `new` 命令一起使用，一个解决办法是，在构造函数内部使用严格模式，即第一行加上 `use strict`。

```javascript
function Animal(name, height) {
  'use strict';
  this.name = name;
  this.height = height;
}

var dog = Animal('大毛', 80);
console.log(dog); // TypeError: name is undefined
```

上面代码的 `Animal` 为构造函数，`use strict` 命令保证了该函数在严格模式下下运行。由于在严格模式中，函数内部的 `this` 不能指向全局对象。如果指向了全局，`this` 默认等于 `undefined`，导致不加 `new` 调用会报错（JavaScript不允许对 `undefined` 添加属性）。

另一个解决方法，是在构造函数内部判断是否使用 `new` 命令，如果发现没有使用，则直接返回一个实例对象。

```javascript
function Animal(name, height) {
  // instanceof 运算符返回一个布尔值，表示对象是否为某个构造函数的实例
  if(!(this instanceof Animal)) {
    return new Animal(name, height);
  }

  this.name = name;
  this.height = height;
}

var dog = Animal('大毛');
console.log(dog.name); // '大毛'
```

上面代码中的构造函数，不管加不加 `new`，都会得到同样的结果。

如果构造函数内部有 `return` 语句，而且 `return` 后面跟着一个复杂数据类型（对象、数组等），`new` 命令会返回 `return` 语句指定的对象；如果 `return` 后面跟着一个简单的数据类型（字符串、布尔值、数字等），则会忽略 `return` 语句，返回 `this` 对象。

```javascript
function Animal() {
  this.height = 80;
  return {
    height: 100
  }
}

var dog = new Animal();
console.log(dog.height); // 100
```

```javascript
function Animal() {
  this.height = 80;
  return 100
}

var dog = new Animal();
console.log(dog.height); // 80
```

另一方面，如果对普通函数（内部没有 `this` 关键字的函数）使用 `new` 命令，则会返回一个空对象。

```javascript
function Animal() {
  return 'this is a message'
}

var dog = new Animal();
console.log(dog); // Animal {}
```

上面代码中，对普通函数 Animal 使用 new 命令，会创建一个空对象。这是因为 `new` 命令总是返回一个对象，要么是实例对象，要么是 `return` 语句指定的对象或数组。本例中，`return` 语句返回的是字符串，所以 `new` 命令就忽略了该语句。

### 模拟实现

```javascript
function new2() {
  var obj = new Object(); // 从 Object.prototype 上克隆一个对象

  Constructor = [].shift.call(arguments); // 取得外部传入的构造器

  obj.__prpto__ = Constructor.prototype; // 指向正确的原型

  var ret = Constructor.apply(obj, arguments); // 借用外部传入的阿构造器给 obj 设置属性

  return typeof ret === 'object' ? ret : obj // 确保构造器总是返回一个对象
}
```
