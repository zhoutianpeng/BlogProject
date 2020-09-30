---
title: webpack学习笔记D5
date: 2020-02-08 17:17:00
categories:
 - [ 前端]
tags:
 - [ webpack]
 - [ nodejs]
---

webpack笔记D5，主要包括以下几小节：开发和生产环境并存、模块化配置、打包第三方类库、watch的使用方法、webpack优化、静态资源集中拷贝
<!-- more -->

# 第十七节 开发和生产环境并存

## 一、生产环境与开发环境
前端项目是区分开发环境和生产环境的，这两个环境的依赖也是不同的。
- 开发依赖：在开发时用来帮助你进行开发、简化代码或者生成兼容设置的依赖包。你可以打开package.json来查看，devDependencies的下面的这些包为开发使用的包。这些包在生产环境中并没有用处，比如webpack依赖。
- 生产依赖：就是比如我们的js使用了jquery，jquery的程序要在浏览器端起作用，也就是说我们最终的程序也需要这个包，这就是生产依赖。这些包在package.json的dependencies中。

## 二、为项目添加依赖
**npm安装**

假如我们要在项目中使用jquery库，一般有三种安装方法：

- `npm install jquery`
安装完成后，package.json中并不存在这个包的依赖。如果别人clone你的git项目，使用npm install安装后就会缺少这个jquery包。项目就会无法正常运行，所以这也是我们最不赞成的安装方法。

- `npm install jquery --save`
安装完成后，它存在于package.json的dependencies中，也就是说它是生产环境需要依赖的包。

- `npm install jquery --save-dev`
安装完成后，它存在于package.json的devDependencies中，也就是说它是开发环境中需要的，上线并不需要这个包的依赖。

**安装项目依赖包**

- `npm install`安装项目全部的依赖包

- `npm install --production`安装生产环境依赖包

## 三、配置生产和开发并行
简单来说就是通过配置实现自动打包开发环境以及生产环境的代码，实现一键切换的效果。

例如项目配置中设置了一个变量website来保存工程的全局地址，用于静态资源正确找到路径，而生产环境和开发环境的website不一样，需要来回切换，这时候我们需要更好的设置方法。

**修改package.json命令**

其实就是添加一个dev设置，并通过环境变量来进行区分，下面是package.json里的值。
```json
"scripts": {
    "server": "webpack-dev-server --open",
    "dev":"set type=dev&webapck",
    "build": "set type=build&webpack"
  },
```
**修改webpack.config.js文件**

可以利用node的语法来读取type的值。
```js
if(process.env.type== "build"){
    const website={
        publicPath:"http://192.168.0.104:1717/"
    }
}else{
    const website={
        publicPath:"http://api.baidu.com/"
    }
}
```
可以用下面的输出type的值。
```js
console.log( encodeURIComponent(process.env.type) );
```

**Mac下的package.json设置**

需要把set换成export，并且要多加一个&符，具体代码如下。
```json
"scripts": {
    "server": "webpack-dev-server --open",
    "dev":"export type=dev&&webpack",
    "build": "export type=build&&webpack"
  },
```

# 第十八节 模块化配置

webpack模块

我的理解就是将webpack.config.js中的某些配置项抽出来单独写到一个新的js文件中并当作模块导出，这样的做方便维护。

例子：entry入口文件进行模块化设置，单独拿出来制作成一个模块。

首先在根目录，新建一个webpack_config文件夹，然后新建entry_webpack.js文件，代码如下：

entry_webpack.js
```js
    //声明entry变量
    const entry ={};  
    //声明路径属性
    entry.path={
        entry:'./src/entry.js'  
    }
    //进行模块化
    module.exports =entry;
```
配置的模块化代码编写好以后，需要在webpack.config.js中引入，注意这里的引入只能使用require的方法。
```js
const entry = require("./webpack_config/entry_webpack.js")

//然后在入口文件部分，修改成如下代码：
entry:entry.path,
```

# 第十九节 打包第三方类库

## 一、直接引入
引入JQuery

安装JQuery
```
npm install --save jquery
```

修改entry.js文件

安装好后，还需要引入到我们的entry.js中，这里直接使用import进行引入就可以。
```
import $ from 'jquery';
```
这里引入是不需要我们写相对路径的，因为jquery的包是在node_modules里的，只要写一个包名jquery，系统会自动为我们查找的。

```
$('#title').html('Hello word');
```

## 二、用plugin引入

这种方式不需要在入口文件中引入，而是webpack作了全局引入。需要使用插件ProvidePlugin，ProvidePlugin是一个webpack自带的插件。

在webpack.config.js中引入webpack
```
const  webpack = require('webpack');
```
>在webpack.config.js里引入必须使用require，否则会报错的

引入成功后配置plugins模块，代码如下。
```js
plugins:[
    new webpack.ProvidePlugin({
        $:"jquery"
    })
],
```
配置好后，就可以在你的入口文件中使用了，而不用再次引入了。这是一种全局的引入.

# 第二十节 watch的使用方法

watch作用是代码发生变化后，只要保存webpack自动为我们进行打包。

```js
watchOptions: {
    poll:1000, //ms -> 监测修改时间每1s监测一次
    aggregateTimeout:500, //半秒内连续按保存，第二次是不会打包的
    ignored:/node_moules/,
}
```
```
"build": "webpack --watch",
```
js版权配置，作用是可以在打包出的js头部自动添上版权信息
使用BannerPlugin插件，在webpack.config.js中配置
```js
const webpack = require('webpack');

new webpack.BannerPlugin('ztp版权所有')
```

# 第二十一节 webpack优化

本节的优化主要是将第三方js库从打包之后的js中拆分出去，防止单个js文件过大

修改webpack.config.js入口配置
```js
 entry: {
        entry: './src/entry.js',
        jquery: 'jquery',
        vue: 'vue',
        entry2: './src/entry2.js'
    },
```

配置插件
```js
new webpack.optimize.CommonsChunkPlugin({
    name:['jquery','vue'],
    filename:'assets/js/[name].min.js',
    minChunks:2
}),
```
运行`npm run build`

# 第二十二节 静态资源集中拷贝

使用插件集中复制public的静态资源

```
cnpm install --save-dev copy-webpack-plugin
```
在webpack.config.js中配置

```js
const copyWebpackPlugin = require('copy-webpack-plugin');

new copyWebpackPlugin([{
    from:__dirname+'/src/public',
    to:'./public'
}
])
```

# 其他杂项 

## 如何引入JSON配置文件

创建配置文件json

在js中引入json
```
const json = require('../config.json');
```



