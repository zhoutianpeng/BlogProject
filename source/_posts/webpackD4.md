---
title: webpack学习笔记D4
date: 2020-02-07 21:52:54
categories:
 - [ 前端]
tags:
 - [ webpack]
 - [ nodejs]
---

webpack笔记D5，主要包括以下几小节：postcss自动添加css属性前缀、消除无用css、Babel转换ES6、打包后如何调试
<!-- more -->

# 第十三节 postcss自动添加css属性前缀

demo 存在问题

`https://github.com/postcss/postcss-loader`


postcss-loader给css3属性自动添加前缀。

```
-webkit-transform: rotate(45deg);
        transform: rotate(45deg);
```

安装

需要安装两个包postcss-loader 和autoprefixer（自动添加前缀的插件）
```
npm install --save-dev postcss-loader autoprefixer
```

postcss.config.js
postCSS推荐在项目根目录（和webpack.config.js同级），建立一个postcss.config.js文件。

postcss.config.js
```
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```
这就是对postCSS一个简单的配置，引入了autoprefixer插件。让postCSS拥有添加前缀的能力，它会根据 can i use 来增加相应的css3属性前缀。

编写我们的loader配置。
```
{
      test: /\.css$/,
      use: [
            {
              loader: "style-loader"
            }, {
              loader: "css-loader",
              options: {
                 modules: true
              }
            }, {
              loader: "postcss-loader"
            }
      ]
}
```
提取CSS

配置提取CSS的loader配置.
```
{
    test: /\.css$/,
    use: extractTextPlugin.extract({
        fallback: 'style-loader',
        use: [
            { loader: 'css-loader', options: { importLoaders: 1 } },
            'postcss-loader'
        ]
    })

}
```

# 第十四节 消除无用css

purifycss

安装
```
npm  i -D purifycss-webpack purify-css
```
引入glob

因为需要同步检查html模板，所以需要引入node的glob对象使用。在webpack.config.js文件头部引入glob。
```
const glob = require('glob');
```
配置purifycss-webpack

同样在webpack.config.js文件头部引入purifycss-webpack
```
const PurifyCSSPlugin = `require`("purifycss-webpack");
```

配置plugins

引入完成后我们需要在webpack.config.js里配置plugins。代码如下，重点看标黄部分。
```js
plugins:[
    //new uglify() 
    new htmlPlugin({
        minify:{
            removeAttrubuteQuotes:true
        },
        hash:true,
        template:'./src/index.html'

    }),
    new extractTextPlugin("css/index.css"),
    new PurifyCSSPlugin({
        // Give paths to parse for rules. These should be absolute!
        paths: glob.sync(path.join(__dirname, 'src/*.html')),
        })
]
```

# 第十五节 Babel转换ES6

Babel是一个编译JavaScript的平台，它的强大之处表现在可以编译转换实现以下目的：

- 使用下一代的javaScript代码(ES6,ES7….)，即使这些标准目前并未被当前的浏览器完全支持。

- 使用基于JavaScript进行了扩展的语言，比如React的JSX。

Babel的安装与配置

Babel其实是几个模块化的包，其核心功能位于称为babel-core的npm包中，webpack可以把其不同的包整合在一起使用，对于每一个你需要的功能或拓展，你都需要安装单独的包（用得最多的是解析ES6的babel-preset-es2015包和解析JSX的babel-preset-react包）。

```
cnpm  install --save-dev babel-core babel-loader babel-preset-es2015 babel-preset-react
```
在webpack中配置Babel的方法如下：
```js
{
    test:/\.(jsx|js)$/,
    use:{
        loader:'babel-loader',
        options:{
            presets:[
                "es2015","react"
            ]
        }
    },
    exclude:/node_modules/
}
```
现在你已经可以用webapck转换ES6的语法兼容各个浏览器了，我们可以修改一下entry.js的代码如下：
```
import css from './css/index.css';
{
    let a = 'Hello Webpack'
    document.getElementById('title').innerHTML=a; 
}
```
上面的代码使用了ES6的let声明方法。如果你不使用Babel来进行转换，你会发现打包出来的js代码没有作兼容处理，使用了Babel转换的代码是进行处理过的。

.babelrc配置

虽然Babel可以直接在webpack.config.js中进行配置，但是考虑到babel具有非常多的配置选项，如果卸载webapck.config.js中会非常的雍长不可阅读，所以我们经常把配置卸载.babelrc文件里。

在项目根目录新建.babelrc文件，并把配置写到文件里。

.babelrc
```json
{
    "presets":["react","es2015"]
}
```
webpack.config.js里的loader配置
```js
{
    test:/\.(jsx|js)$/,
    use:{
        loader:'babel-loader',
    },
    exclude:/node_modules/
}
```
ENV：

现在网络上已经不流行babel-preset-es2015，现在官方推荐使用的是babel-preset-env

首先需要下载：
```
npm n install --save-dev babel-preset-env
```
然后修改.babelrc里的配置文件。其实只要把之前的es2015换成env就可以了。
```json
{
    "presets":["react","env"]
}
```

# 第十六节 打包后如何调试

## 一、source map
JavaScript正变得越来越复杂。大部分源码（尤其是各种函数库和框架）都要经过转换，才能投入生产环境。
常见的源码转换，主要是以下三种情况：

1. 压缩，减小体积。比如jQuery 1.9的源码，压缩前是252KB，压缩后是32KB。
2. 多个文件合并，减少HTTP请求数。
3. 其他语言编译成JavaScript

转换导致了实际运行的代码不同于开发代码，使得debug变得困难重重。

通常，JavaScript的解释器会告诉你，第几行第几列代码出错。但是，这对于转换后的代码毫无用处。举例来说，jQuery 1.9压缩后只有3行，每行3万个字符，所有内部变量都改了名字。你看着报错信息，感到毫无头绪，根本不知道它所对应的原始位置。这就是Source map想要解决的问题。

简单说，Source map就是一个信息文件，里面储存着位置信息。也就是说，转换后的代码的每一个位置，所对应的转换前的位置，出错的时候，除错工具将直接显示原始代码，而不是转换后的代码。这无疑给开发者带来了很大方便。

## 二、webpack devtool
webpack只要通过简单的devtool配置，就会自动给我们生产source maps文件，map文件是一种对应编译文件和源文件的方法，让我们调试起来更简单。

四种选项

在配置devtool时，webpack给我们提供了四种选项。

- source-map:在一个单独文件中产生一个完整且功能完全的文件。这个文件具有最好的source map，会包含出错行列的信息但是它会减慢打包速度；
- cheap-module-source-map:在一个单独的文件中产生一个不带列映射的map，不带列映射提高了打包速度，但是也使得浏览器开发者工具只能对应到具体的行，不能对应到具体的列,会对调试造成不便。
- eval-source-map:使用eval打包源文件模块，在同一个文件中生产干净的完整版的sourcemap，但是对打包后输出的JS文件的执行具有性能和安全的隐患。在开发阶段这是一个非常好的选项，在生产阶段则一定要不开启这个选项。
- cheap-module-eval-source-map:这是在打包文件时最快的生产source map的方法，生产的 Source map 会和打包后的JavaScript文件同行显示，没有影射列，和eval-source-map选项具有相似的缺点。
四种打包模式，有上到下打包速度越来越快，不过同时也具有越来越多的负面作用，较快的打包速度的后果就是对执行和调试有一定的影响。

个人意见是，如果大型项目可以使用source-map，如果是中小型项目使用eval-source-map就完全可以应对，需要强调说明的是，source map只适用于开发阶段，上线前记得修改这些调试设置。

简单的配置：
```js
module.exports = {
  devtool: 'eval-source-map',
  entry:  __dirname + "/app/main.js",
  output: {
    path: __dirname + "/public",
    filename: "bundle.js"
  }
}
```