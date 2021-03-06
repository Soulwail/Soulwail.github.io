---
layout: post
title: 算法的时间复杂度与空间复杂度
date: 2019-05-21
desc: "算法的时间复杂度与空间复杂度"
keywords: "算法"
categories: [Js]
tags: [算法]
icon:
---

算法（Algorithm）是指用来操作数据、解决程序问题的一组方法。对于同一个问题，使用不同的算法，也许最终得到的结果是一样的，但在过程中消耗的资源和时间却会有很大的区别。

那么我们应该如何去衡量不同算法之间的优势呢？

主要还是从算法所占用的「时间」和「空间」两个维度去考量。

- 时间维度：是指执行当前算法所消耗的时间，我们通常用「时间复杂度」来描述。
- 空间维度：是指执行当前算法需要占用多少内存空间，我们通常用「空间复杂度」来描述。

因此，评价一个算法的效率主要是看它的时间复杂度和空间复杂度情况。然而，有的时候时间和空间却又是「鱼和熊掌」，不可兼得，那么我们就需要从中去取一个平衡点。

下面就来分别介绍一下「时间复杂度」和「空间复杂度」的计算方式。

## 一、时间复杂度

我们想要知道一个算法的「时间复杂度」，很多人首先想到的方法就是把这个算法程序运行一遍，那么它所消耗的时间就自然而然知道了。

这种方式可以吗？当然可以，不过它也有很多弊端。

这种方式非常容易受运行环境的影响，在性能高的机器上跑出来的结果与在性能低的机器上跑的结果会相差很大。而且对测试时使用的数据规模也有很大关系。再者，我们在写算法时，还没有办法完整的去运行。

因此，另一种更为通用的方法就出来了：「**大 O 符号表示法**」，即 $T(n) = O(f(n))$。

我们先来看个例子：

~~~javascript
for (i = 1; i <= n; ++i) {
    j = i;
    j++;
}
~~~

通过「大 O 符号表示法」，这段代码的时间复杂度为：$O(n)$，为什么呢？

在大 O 符号表示法中，时间复杂度的公式是：$T(n) = O(f(n))$，其中 $f(n)$ 表示每行代码执行次数之和，而 O 表示正比例关系，这个公式的全称是：**算法的渐进时间复杂度**。

我们继续看上面的例子，假设每行代码的执行时间都是一样的，我们用 1 颗粒时间来表示，那么这个例子的第一行耗时是 1 个颗粒时间，第二行的执行时间是 n 个颗粒时间，第三行的执行时间也是 n 个颗粒时间，那么总时间就是 1 颗粒时间 + n 颗粒时间 + n 颗粒时间，即（1 + 2n）个颗粒时间，因此，我们可以简化的将这个算法的时间复杂度表示为：$T(n) = O(n)$。

为什么可以这么去简化呢？因为大 O 符号表示法并不是用来真实代表算法的执行时间的，它是用来表示代码执行时间的增长变化趋势的。

所以上面的例子中，如果 n 无限大的时候，$T(n) = time(1 + 2n)$ 中的常量 1 就没有意义了，倍数 2 也意义不大。因此直接简化为 $T(n) = O(n)$ 就可以了。

常见的时间复杂度量级有：

- 常数阶 $O(1)$
- 对数阶 $O(log_2n)$
- 线性阶 O($n$)
- 线性对数阶 $O(nlog_2n)$
- 平方阶 $O(n^2)$
- 立方阶 $O(n^3)$
- K 次方阶 $O(n^k)$
- 指数阶 $O(2^n)$
- 阶乘阶 $O(n!)$

上面从上至下依次的时间复杂度越来越大，执行的效率越来越低。

下面选取一些较为常用的来讲解一下：

### 1、常数阶 $O(1)$

无论代码执行了多少行，只要是没有循环等复杂结构，那这个代码的时间复杂度就都是 $O(1)$，如：

~~~javascript
let i = 1;
let j = 2;
++i;
j++;
let m = i + j;
~~~

上述代码在执行的时候，它消耗的时间并不随着某个变量的增长而增长，那么无论这类代码有多长，即使有几万几十万行，都可以用 $O(1)$ 来表示它的时间复杂度。

### 2、线性阶 $O(n)$

这个在最开始的代码示例中就讲解过了，如：

~~~javascript
for (i = 1; i < n; ++i) {
  j = i;
  j++;
}
~~~

这段代码，`for` 循环里面的代码会执行 n 遍，因此它消耗的时间是随着 n 的变化而变化的，因此这类代码都可以用 $O(n)$ 来表示它的时间复杂度。

### 3、对数阶 $O(log_2n)$

还是先来看代码：

~~~javascript
i = 1;

while (i < n) {
  i = i * 2;
}
~~~

从上面的代码可以看到，在 `while` 循环里面，每次都讲 i 乘以 2，乘完之后，i 距离 n 就越来越近了。我们试着求解一下，假设循环 x 次之后，i 就大于 2 了，此时这个循环就退出了，也就是说 2 的 x 次方等于 n，那么 $x = log_2n$，也就是说，当循环 $log_2n$ 次以后，这个代码就结束了。因此这个代码的时间复杂度为：$O(log_2n)$。

### 4、线性对数阶 $O(nlog_2n)$

线性对数阶 $O(nlog_2n)$ 其实非常容易理解，将时间复杂度为 $O(log_2n)$ 的代码循环 n 遍的话，那么它的时间复杂度就是 $n * O(log_2n)$，也就是 $O(nlog_2n)$。

就拿上面的代码加一点修改来举例：

~~~javascript
for (m = 1; m < n; m++) {
  i = 1;
  while (i < n) {
    i = i * 2;
  }
}
~~~

### 5、平方阶 $O(n^2)$

平方阶 $O(n^2)$ 就更容易理解了，如果把 $O(n)$ 的代码再嵌套循环一遍，它的时间复杂度就是 $O(n^2)$ 了。

举例：

~~~javascript
for (x = 1; x <= n; x++) {
  for (i = 1; i <= n; i++) {
    j = i;
    j++;
  }
}
~~~

这段代码其实就是嵌套了两层 n 循环，它的时间复杂度就是 $O(n * n)$，即 $O(n^2)$。如果将其中一层循环的 n 改为 m，即：

~~~javascript
for (x = 1; x <= m; x++) {
  for (i = 1; i <= n; i++) {
    j = i;
    j++;
  }
}
~~~

那它的时间复杂度就变成了 $O(m * n)$。

### 6、立方阶 $O(n^3)$、K 次方阶 $O(n^k)$

参考上面的 $O(n^2)$ 去理解就好了，$O(n^3)$ 相当于三层 n 循环，其它的类似。

### 7、指数阶 $O(2^n)$

### 8、$O(n!)$

除此之外，其实还有平均时间复杂度、均摊时间复杂度、最坏时间复杂度、最好时间复杂度的分析方法，有点复杂，这里就不展开了。

## 二、空间复杂度
