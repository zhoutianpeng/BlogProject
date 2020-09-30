---
title: webpack学习笔记D1
date: 2020-02-02 22:16:40
categories:
 - [ 前端]
tags: 
 - [ webpack]
 - [ nodejs]
---

该系列是记录webpack的初学笔记，主要包括以下几小节：webpack3.0 初识与安装、快速创建demo、webpack入口和出口、服务与热更新
<!-- more -->

# 第一节 webpack3.0 初识与安装
> 该小节对webpack进行简介，并学习如何安装webpack
## 一、webpack的作用
webpack在业界的流行主要是因为前端网页功能丰富，尤其是在SPA单页应用流行后。前端为了简化开发的复杂度出现了许多优秀的解决方案，比如模块化、TypeScript扩展语言、scss less等CSS预处理器，这些方式极大的提高了开发效率，但他们往往需要额外的处理才能够被浏览器识别比如ts代码转为js，手工处理是麻烦的，由此产生了了webpack等打包工具出现的需求。

webpack可以看看作模块打包机，他做的事情简单来说是：分析你的项目结构-->找到js模块以及其他不能被浏览器直接运行的扩展语言（ts、scss）-->将其打包为合适格式供浏览器使用，同时还会做性能的优化

## 二、安装

***全局安装*** 
```
npm install -g webpack
```
官方不推荐全局安装webpack，因为全局安装会锁定webpack版本对于多项目使用不同版本webpack不友好

***本地安装***

创建本地空项目
```js
  npm init
```

```js
 npm install --save-dev webpack
```
--save是指该模块会进行本地安装该依赖会写入到package.json文件中

--save-dev是指保存到开发环境，不进入生产环境

webpack安装失败时需要检查三个方面
- node版本过低，6以下版本过低
- npm被阻挡，可以使用cnpm
- 权限问题 sudo安装

# 第二节 快速创建demo
>该小节创建了一个最简单的demo，直观的学习webpack打包的过程

webpack在项目中使用的时间较少，基本只有在项目初期会使用

Q：如何把旧项目的webpack更新？
- 打开package.json，找到webpack，修改对应的version
- 删除工程的`node_modules`文件夹
- 执行`npm i`

## 一、建立基本的项目结构
打开上节创建的项目（这里使用vs code开发环境），在项目的根目录下创建两个目录src和dist，dist目录用于存放生产环境的代码

demo如下
- 在dist目录中创建index.html,简单的写一些网页，并引入一个名为`bundle.js`的脚本
```html
    <html>
        <head></head>
        <body>
            <div id = "title"></div>
            <script src ="./bundle.js"></script>
        </body>
    </html>
 ```
- 在src目录下创建一个entry.js文件实现一些简单功能
```js
    document.getElementById("title").innerHTML="Hello webPack";
```

## 二、使用webpack进行打包

打开terminal输入命令`webpack src/entry.js dist/bundle.js`，即将entry.js打包到dist目录下bundle.js

使用live-server创建一个临时服务，查看效果

```
npm install -g live-server //安装live-server
live-server //启动临时服务
```

实际操作demo遇到了如下问题
1. `bash: webpack: command not found`，打包时，提示webpack not found，是因为直接输入`webpack src/entry.js dist/bundle.js`命令相当于在搜寻全局的webpack，若使用npm run 自定义命令，则是在项目的模块中搜寻webpack。该demo使用是本地安装的webpack，这里没有创建npm run的自定义命令解决，而是偷懒使用了`node_modules/.bin/webpack 后面接参数选项`的方式避开了问题。

# 第三节 webpack入口和出口
> 该小节创建并编写了webpack的配置文件，并配置了多入口多出口的情况

## 一、使用webpack.config.js结构
```js
    module.exports={
        entry: {},
        output: {},
        module: {}, //打包css、图片转换、图片压缩等配置
        plugins: [], //
        devServer:{} //配置开发服务
    }
```
## 二、entry选项（入口配置）
```js
  entry: {
        entry: './src/entry.js'
    }
```
## 三、output选项（出口配置）
```js
    output: {
        path: path.resolve(__dirname,'dist'),
        filename: 'bundle.js'
    }
```
## 四、多入口、多出口配置
```js
const path = require('path');

module.exports={
    entry: {
        entry: './src/entry.js',
        entry2: './src/entry2.js'
    },
    output: {
        path: path.resolve(__dirname,'dist'),
        filename: '[name].js'
    },
    module: {}, 
    plugins: [], 
    devServer:{} 
}
```

# 第四节 服务与热更新
>
网络不太好，这里换成cnpm安装依赖了
```
npm install cnpm -g --registry=https://registry.npm.taobao.org 
```

使用webpack提供的webpack-dev-server工具可以在开发环境中搭建服务，并实现服务的热更新

首先安装webpack-dev-server，这里使用了2.X版本是因为3.X版本貌似需要安装webpack-cli，然后安装后工程打包有些问题没有深入研究😂。

```
cnpm install webpack-dev-server@2.8.2 --save-dev
```

修改配置文件
```js
module.exports={
    entry: {
        entry: './src/entry.js',
        entry2: './src/entry2.js'
    },
    output: {
        path: path.resolve(__dirname,'dist'),
        filename: '[name].js'
    },
    module: {}, //打包css、图片转换、图片压缩等配置
    plugins: [], //
    devServer:{
        contentBase:path.resolve(__dirname,'dist'), //基本目录结构：监听哪段代码
        host:'localhost',
        compress:true,//服务器压缩
        port:80,
    } 
}
```
创建npm 自定义命令
```json
  "scripts": {
    "server": "webpack-dev-server"
  },
```
