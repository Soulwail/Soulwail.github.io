---
layout: post
title: 深度解锁 Webpack（进阶篇）
date: 2020-03-27
desc: "Webpack 学习"
keywords: "Js, Webpack"
categories: [Webpack]
tags: [Webpack]
icon:
---

## 1. 静态资源拷贝

有些时候，我们需要使用已有的 JS 文件、CSS 文件（本地文件），但是不需要 `webpack` 编译。例如，我们在 `public/index.html` 中引入了 `public` 目录下 `js` 或 `css` 文件。这个时候直接打包，那么在构建出来之后，肯定是找不到对应的 `js` / `css` 了。

> `public` 目录结构

```markdown
├── public
│   ├── config.js
│   ├── index.html
│   ├── js
│   │   ├── base.js
│   │   └── other.js
│   └── login.html
```

现在，我们在 `index.html` 中引入了 `./js/base.js`。

```html
<!-- index.html -->
<script src="./js/base.js"></script>
```

这时候，我们 `npm run dev`，会发现有找不到该资源文件的报错信息。

对于这个问题，我们可以手动将其拷贝至构建目录。然后在配置 `CleanWebpackPlugin` 时，注意不要清空对应的文件或文件夹即可，但是如若这个静态文件时不时的还会修改下，那么依赖于手动拷贝，是很容易出问题的。

不要过于相信自己的记性，依赖于手动拷贝的方式，大多数人应该都有过忘记拷贝的经历。

幸运的是，`webpack` 为我们这些记性不好又爱偷懒的人提供了好用的插件 `CopyWebpackPlugin`，它的作用就是将单个文件或整个目录复制到构建目录。

首先安装一下依赖：

```markdown
npm install copy-webpack-plugin -D
```

修改配置（当前，需要做的是将 `public/js` 目录拷贝至 `dist/js` 目录）：

```javascript
// webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = {
  // ...
  plugins: [
    new CopyWebpackPlugin([
      {
        from: 'public/js/*.js',
        to: path.resolve(__dirname, 'dist', 'js'),
        flatten: true,
      },
      // 还可以继续配置其它要拷贝的文件
    ])
  ]
}
```

此时，重新执行 `npm run dev`，报错信息已经消失。

这里说一下 `flatten` 这个参数，设置为 `true`，那么它只会拷贝文件，而不会把文件夹路径都拷贝上，大家可以不设置 `flatten` 时，看下构建结果。

另外，如果我们要拷贝一个目录下的很多文件，但是想过滤掉某个或某些文件，那么 `CopyWebpackPlugin` 还为我们提供了 `ignore` 参数。

```javascript
// webpack.config.js
const CopyWebpackPlugin = require('copy-webpack-plugin');
module.exports = {
  // ...
  plugins: [
    new CopyWebpackPlugin([
      {
        from: 'public/js/*.js',
        to: path.resolve(__dirname, 'dist', 'js'),
        flatten: true,
      }
    ], {
      ignore: ['other.js']
    })
  ]
}
```

例如，这里我们忽略掉 `js` 目录下的 `other.js` 文件，使用 `npm run build` 构建，可以看到 `dist/js` 下不会出现 `other.js` 文件。`CopyWebpackPlugin` 还提供了很多其它的参数，如果当前的配置不能满足你，可以查阅文档进一步修改配置。

## 2. ProvidePlugin

`ProvidePlugin` 在我看来，是为懒人准备的，不过也别过度使用，毕竟全局变量不是什么“好东西”。``ProvidePlugin` 的作用就是不需要 `import` 或 `require` 就可以在项目中到处使用。

`ProvidePlugin` 是 `webpack` 的内置插件，使用方法如下：

```javascript
new webpack.ProvidePlugin({
  identifier1: 'module1',
  identifier2: ['module2', 'property2']
})
```

默认寻找路径是当前文件夹 `./**` 和 `node_modules`，当然，你也可以指定全路径。

`React` 大家都知道的，使用的时候，要在每个文件中引入 `React`，不然立刻抛错给你看。还有就是 `jquery`，`lodash` 这样的库，可能在多个文件中使用，但是懒得每次都引入，好嘛，一起来偷个懒，修改下 `webpack` 的配置：

```javascript
const webpack = require('webpack');
module.exports = {
  // ...
  plugins: {
    new webpack.ProvidePlugin({
    	React: 'react',
    	Component: ['react', 'Component'],
  		Vue: ['vue/dist/vue.esm.js', 'default'],
      $: 'jquery',
      _map: ['lodash', 'map']
  	})
  }
}
```

这样配置之后，你就可以在项目中随心所欲的使用 `$`、`_map` 了，并且写 `React` 组件时，也不需要 `import` `React` 和 `Component` 了，如果你想的话，你还可以把 `React` 的 `Hooks` 都配置在这里。

另外呢，`Vue` 的配置后面多了一个 `default`，这是因为 `vue.esm.js` 中使用的是 `export default` 导出的，对于这种，必须要指定 `default`。`React` 使用的是 `module.exports` 导出的，因此不要写 `default`。

另外，就是如果你启动了 `eslint` 的话，记得修改下 `eslint` 的配置文件，增加以下配置：

```javascript
{
  "globals": {
    "React": true,
    "Vue": true,
    // ...
  }
}
```

当然啦，偷懒要有个度，你是要配一大堆全局变量，最终可能会给自己带来麻烦，对自己配置的全局变量一定要负责到底。

## 3. 抽离 CSS

CSS 打包我们前面已经说过了，不过呢，有些时候，我们可能会有抽离 CSS 的需求，即将 CSS 文件单独打包，这可能是因为打包成一个 JS 文件太大，影响加载速度，也有可能是为了缓存，例如，只有 JS 部分有改动。

不管你是因为什么原因要抽离 CSS，只要你有需求，我们就可以去实现。

首先，安装 `loader`：

```markdown
npm install mini-css-extract-plugin -D
```

**`mini-css-extract-plugin` 和 `extract-text-webpack-plugin` 相比：**

1. 异步加载
2. 不会重复编译（性能更好）
3. 更容易使用
4. 只适用 CSS

修改我们的配置文件：

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name].css' // 个人习惯将 css 文件放在单独目录下
    })
  ],
  module: {
    rules: [
      {
        test: /\.(le|c)ss$/,
        use: [
          MiniCssExtractPlugin.loader, // 替换之前的 style-loader
          'css-loader', {
            loader: 'postcss-loader',
            options: {
              plugins: function () {
                return [
                  require('autoprefixer')({
                    "overrideBrowserslist": [
                      "defaults"
                    ]
                  })
                ]
              }
            }
          }, 'less-loader'
        ],
        exclude: /node_modules/
      }
    ]
  }
}
```

现在我们重新编译：`npm run dev`，目录结构如下所示：

```markdown
.
├── dist
│   ├── assets
│   │   ├── alita_e09b5c.jpg
│   │   └── thor_e09b5c.jpeg
│   ├── css
│   │   ├── index.css
│   │   └── index.css.map
│   ├── bundle.fb6d0c.js
│   ├── bundle.fb6d0c.js.map
│   └── index.html
```

前面说了最好新建一个 `.browserslistrc` 文件，这样可以多个 `loader` 共享配置，所以，动手在根目录下新建文件（`.browserslistrc`）。内容如下（你也可以根据自己项目需求，修改为其它配置）：

```javascript
last 2 version
> 0.25%
not dead
```

修改 `webpack.config.js`：

```javascript
// webpack.config.js
const MiniCssExtractPlugin = require('mini-css-extract-plugin');
module.exports = {
  plugins: [
    new MiniCssExtractPlugin({
      filename: 'css/[name].css'
    })
  ],
  module: {
    rules: [
      {
        test: /\.(le|c)ss$/,
        use: [
          MiniCssExtractPlugin.loader, // 替换之前的 style-loader
          'css-loader', {
            loader: 'postcss-loader',
            options: {
              plugins: function () {
                return [
                  require('autoprefixer')()
                ]
              }
            }
          }, 'less-loader'
        ],
        exclude: /node_modules/
      }
    ]
  }
}
```

要测试自己的 `.browserlistrc` 有没有生效也很简单，直接将文件内容修改为 `last 1 Chrome versions`，然后对比修改前后的构建出的结果，就能看出来了。

更多 browserslistrc 配置，可以点击[这里](https://github.com/browserslist/browserslist 'hp')。

更多 mini-css-extract-plugin 配置项，可以查看 mini-css-extract-plugin 文档。

### 3.1 将抽离出来的 css 文件进行压缩

