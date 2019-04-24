---
layout: post
title: Vue 生命周期详解
date: 2019-04-17
desc: "Vue 生命生命周期"
keywords: "vue, lifecycle, 生命周期"
categories: [Vue]
tags: [Vue, 生命周期]
icon:
---

## 生命周期讲解

下面是 Vue 官网上对于生命周期讲解的图。

<div style="text-align: center"><img width="700" src="https://github.com/Soulwail/Soulwail.github.io/raw/master/image_files/lifecycle.png"/></div>

在谈到 Vue 生命周期的时候，我们首先需要创建一个实例，也就是在 `new Vue()` 的对象过程当中，首先执行了 init（init 是 Vue 组件里面默认去执行的），在 init 的过程当中，首先调用了 beforeCreate，然后在 injections（注射）和 reactivity（反应性）的时候，它会再去调用 created。所以在 init 的时候，事件已经调用了，我们在 beforeCreate 的时候千万不要去修改 data 里面赋值的数据，最早也要放在 created 里面去做（添加一些行为）。

当 created 完成之后，它会去判断 instance（实例）里面是否含有 “el”这个选项，如果没有的话，它会调用 `vm.$mount(el)` 这个方法，然后执行下一步；如果有的话，直接执行下一步。紧接着会判断是否含有 “template” 这个选项，如果有的话，它会把 template 解析成一个 `render function`，这是一个 template 编译的过程，结果是解析成了 render 函数：

~~~javascript
render (h) {
  return h('div', {}, this.text)
}
~~~

解释一下，render 函数里面的传参 h 就是 Vue 里面的 createElement 方法，return 返回一个 createElement 方法，其中要传入 3 个参数，第一个参数就是创建的 div 标签；第二个参数传了一个对象，对象里面可以是我们组件上面的 props，或者是事件之类的东西；第三个参数就是 div 标签里面的内容，这里我们指向了 data 里面的 text。

使用 render 函数的结果和我们之前使用 template 解析出来的结果是一样的。render 函数是发生在 beforeMount 和 mounted 之间的，这也从侧面说明了，在 beforeMount 的时候，`$el` 还只是我们在 HTML 里面写的节点，然后到 mounted 的时候，它就把渲染出来的内容挂载到了 DOM 节点上。这中间的过程其实是执行了 render function 的内容。

在使用 .vue 文件开发的过程中，我们在里面写了 template 模板，在经过了 vue-loader 的处理之后，就变成了 `render function`，最终放到了 vue-loader 解析过的文件里面。这样做有什么好处呢？原因是解析 template 变成 `render function` 是一个非常耗时的过程，vue-loader 帮助我们处理了这些内容之后，当我们在页面上执行 vue 代码的时候，效率会变得更高。

beforeMount 在有了 `render function` 的时候才会执行，当执行完 `render function` 之后，就会调用 mounted 这个钩子，在 mounted 挂载完毕之后，这个实例就算走完流程了。

后续的钩子函数执行的过程都是需要外部的触发才会执行。比如说有数据的变化，会调用 beforeUpdate，然后经过 Virtual DOM，最后 updated 更新完毕。当组件被销毁的时候，它会调用 beforeDestroy，以及 destroyed。

这就是 vue 实例从新建到销毁的一个完整流程，以及在这个过程中它会触发哪些生命周期的钩子函数。说道这儿，可能很多同学会问，钩子函数是什么意思？

钩子函数，其实和回调是一个概念，当系统执行到某处时，检查是否有 hook，有则回调。说的更直白一点，每个组件都有属性、方法和事件。所有的生命周期都归于事件，在某个时刻自动执行。

<br>

## renderError

在整个生命周期过程中，Vue 为我们提供了 renderError 方法，这个方法只有在开发的时候它才会被调用，在正式打包上线的过程中，它是不会被调用的。它主要是帮助我们调试 render 里面的一些错误。

~~~javascript
renderError (h, err) {
  return h('div', {}, err.stack)
}
~~~

有且只有当 render 方法里面报错了，才会执行 renderError 方法。

还有一点，renderError 只有在本组件的 render 方法报错的情况下它才会被调用。
