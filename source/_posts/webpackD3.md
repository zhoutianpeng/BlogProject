---
title:  webpack学习笔记D3
date: 2020-02-06 21:52:06
categories:
 - [ 前端]
tags:
 - [ webpack]
 - [ nodejs]
---

webpack笔记D3，主要包括以下几小节：打包css文件、JS代码压缩、HTML发布、css中引入图片
<!-- more -->

# 第九节 css分离和publicPath

出现的问题，打开之前的工程，执行npm run server时terminal报了如下的错误，原因是端口被占用
```js
events.js:183
      throw er; // Unhandled 'error' event
      ^

Error: listen EADDRINUSE 192.168.0.17:1717
    at Server.setupListenHandle [as _listen2] (net.js:1360:14)
    at listenInCluster (net.js:1401:12)
    at doListen (net.js:1510:7)
    at _combinedTickCallback (internal/process/next_tick.js:142:11)
    at process._tickCallback (internal/process/next_tick.js:181:9)
    at Function.Module.runMain (module.js:696:11)
    at startup (bootstrap_node.js:204:16)
    at bootstrap_node.js:625:3
```
解决方案
```
losf -i:端口

kill -9 +pid
```

## 一、css分离

在之前的学习中css文件被打包进了js中，如果有需要将其分离出来可以使用extract-text-webpack-plugin插件

安装
```
npm install --save-dev extract-text-webpack-plugin
```

配置webpack.config.js
```js
const extractTextPlugin = require("extract-text-webpack-plugin");

//设置plugins属性
new extractTextPlugin("/css/index.css")

//重新配置style-loader和css-loader
module:{
        rules: [
            {
              test: /\.css$/,
              use: extractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader"
              })
            },{
               test:/\.(png|jpg|gif)/ ,
               use:[{
                   loader:'url-loader',
                   options:{
                       limit:500000
                   }
               }]
            }
          ]
    },
```

完成如上配置可以进行打包，此时dist目录中css与js文件会分离

## 二、publicPath
利用extract-text-webpack-plugin插件很轻松的就把CSS文件分离了出来，但是CSS中引入图片的路径并不正确，这里推荐使用publicPath方案解决。

publicPath：是在webpack.config.js文件的output选项中，主要作用就是处理静态文件路径的。

在处理前，我们在webpack.config.js 上方声明一个对象，叫website。
```js
const website ={
    publicPath:"http://192.168.1.108:1717/"
}
```
然后在output选项中引用这个对象的publicPath属性。
```js
//出口文件的配置项
    output:{
        //输出的路径，用了Node语法
        path:path.resolve(__dirname,'dist'),
        //输出的文件名称
        filename:'[name].js',
        publicPath:website.publicPath
    },
```
配置完成后，你再使用webpack命令进行打包原来的相对路径改为了绝对路径，这样来讲速度更快。

# 第十节 HTML图片打包

如何把图片放到指定的文件夹下

之前打包后的图片并没有放到images文件夹下，要放到images文件夹下，其实只需要配置的url-loader选项就可以了。
```js
module:{
        rules: [
            {
              test: /\.css$/,
              use: extractTextPlugin.extract({
                fallback: "style-loader",
                use: "css-loader"
              })
            },{
               test:/\.(png|jpg|gif)/ ,
               use:[{
                   loader:'url-loader',
                   options:{
                       limit:5000,
                       outputPath:'images/',
                   }
               }]
            }
          ]
    },
```

> 该demo并未跑通，需要再调试；引入新的插件之后build html文件时会报解析错误

#copy

标签的问题，在webpack中是不喜欢你使用标签来引入图片的，但是我们作前端的人特别热衷于这种写法，国人也为此开发了一个：html-withimg-loader。他可以很好的处理我们在html 中引入图片的问题。因为是国人开发的，文档都是中文，所以学习起来还是比较简单的。所以在学习新课之前我们先解决两个小伙伴的问题。

安装：
```
npm install html-withimg-loader --save
```
配置loader
```
webpack.config.js

{
    test: /\.(htm|html)$/i,
     use:[ 'html-withimg-loader'] 
}
```
然后在终端中可以进行打包了。你会发现images被很好的打包了。并且路径也完全正确。

# 第十一节 打包分类LESS

## 一、打包Less文件

安装Less:

```
npm install --save-dev less
```

安装Less-loader用来打包
```
npm n install --save-dev less-loader
```

配置loader：

在webpack.config.js里配置loader，打包Less还需要style-loader和css-loader

webpack.config.js
```js
{
    test: /\.less$/,
    use: [{
           loader: "style-loader" // creates style nodes from JS strings
        }, {
            loader: "css-loader" // translates CSS into CommonJS
        , {
            loader: "less-loader" // compiles Less to CSS
        }]
}
```

编写less文件

black.less
```css
@base :#000;
#gogo{
    width:300px;
    height:300px;
    background-color:@base;
}
```
这里#gogo是层的ID名称。@base是我们设置的变量名称。

引入entery.js文件中
```
import less from './css/black.less';
```
这样我们就可以把less文件进行打包了。我们可以使用webpack命令打包试一试。

## 二、Lees文件分离

分离Less方法与分离css方法一致，使用extract-text-webpack-plugin插件
```js
{
    test: /\.less$/,
    use: extractTextPlugin.extract({
        use: [{
            loader: "css-loader"
        }, {
            loader: "less-loader"
        }],
        // use style-loader in development
        fallback: "style-loader"
    })
}
```
配置好后，less被分离到了index.css文件里,Less是非常好的CSS扩展，但是Less得转换稍显麻烦

# 第十二节：SASS文件的打包和分离
SASS的配置分离与Less的配置分离类似

## 一、安装SASS打包的loaders

在项目目录下用npm安装两个包

node-sass和sass-loader

- node-sass：因为sass-loader依赖于node-sass，所以需要先安装node-sass

    - `npm n install --save-dev node-sass`
- sass-loader:

    - `npm install --save-dev sass-loader`

注意：在用npm安装时，这个loader很容易安装失败，最好使用cnpm来进行安装。

配置loader
```js
{
                test: /\.scss$/,
                use: [{
                    loader: "style-loader" // creates style nodes from JS strings
                }, {
                    loader: "css-loader" // translates CSS into CommonJS
                }, {
                    loader: "sass-loader" // compiles Sass to CSS
                }]
}
```

Sass文件的编写

编写sass文件拉，并将sass文件引入到entery.js中。
```css
$nav-color: #FFF;
#nav {
  $width: 100%;
  width: $width;
  height:30px;
  background-color: $nav-color;
}
```

启动 npm run server 

## 二、SASS文件分离
```js
{
            test: /\.scss$/,
            use: extractTextPlugin.extract({
                use: [{
                    loader: "css-loader"
                }, {
                    loader: "sass-loader"
                }],
                // use style-loader in development
                fallback: "style-loader"
            })
}
```





