---
title: webpack学习笔记D2
date: 2020-02-04 10:35:44
categories:
 - [ 前端]

tags:
 - [ webpack]
 - [ nodejs]
---

webpack笔记2，主要包括以下几小节：打包css文件、JS代码压缩、HTML发布、css中引入图片
<!-- more -->

# 第五节 打包css文件

Webpack在生产环境中有一个重要的作用就是减少http的请求数，是通过将多个文件打包到一个js里，这样请求数就可以减少好多。本小节对css文件打包。


***Loaders***

Loaders是Webpack最重要的功能之一，也是Webpack盛行的原因。通过使用不同的Loader，Webpack可以对不同的文件格式进行特定处理。

简单的举几个Loaders使用例子：

- 可以把SASS文件的写法转换成CSS，而不在使用其他转换工具。
- 可以把ES6或者ES7的代码，转换成大多浏览器兼容的JS代码。
- 可以把React中的JSX转换成JavaScript代码。

注意：所有的Loaders都需要在npm中单独进行安装，并在webpack.config.js里进行配置。

对Loaders的配置简单梳理一下
- test：用于匹配处理文件的扩展名的表达式，这个选项是必须进行配置的；
- use：loader名称，就是你要使用模块的名称，这个选项也必须进行配置，否则报错；
- include/exclude:手动添加必须处理的文件（文件夹）或屏蔽不需要处理的文件（文件夹）（可选）；
- query：为loaders提供额外的设置选项（可选）。

演示最简单的打包css的demo，在项目中本地安装style-loader、以及css-loader，并在webpack.config.js中做配置
```
npm install style-loader --save-dev
npm install css-loader --save-dev
```
配置webpack.config.js的module，有如下三种方式，其效果是相同的
```js
    module: {
        rules:[
            {
                test:/\.css$/,
                use:['style-loader','css-loader'],
            }
        ]
    }

    module:{
        rules:[
            {
                test:/\.css$/,
                loader:['style-loader','css-loader']
            }
        ]
    }

    module:{
        rules:[
            {
                test:/\.css$/,
                use: [
                    {
                        loader: "style-loader"
                    }, {
                        loader: "css-loader"
                    }
                ]
            }
        ]
    }
```

在src目录下创建 css/index.css文件写入一些样式
```css
body{
    background: red;
    color:#FFF;
}
```
在js代码中做引用
```js
import css from './css/index.css';
```

运行`npm run server`命令



# 第六节 JS代码压缩

在Webpack中可以很轻松的实现JS代码的压缩，它是通uglifyjs-webpack-plugin插件实现的，虽然uglifyjs是插件但是webpack版本里默认已经集成，不需要再次安装。

我们需要在webpack.config.js中引入uglifyjs-webpack-glugin插件

```
const uglify = `require`('uglifyjs-webpack-plugin');
```
引入后在plugins配置里new一个 uglify对象就可以了，代码如下。

```
 plugins:[
        new uglify()
    ],
```
在terminal中使用webpack进行打包出得js代码已经被压缩了。

在VSCode中，可以按Alt+Z使长文件自动换行，查看效果。    

# 第七节 HTML发布

演示最简单的HTML发布demo，在src目录下创建index.html写入简单的代码

需要去掉的JS引入代码，webpack会自动为我们引入JS
```html
<html>
    <head></head>
    <body>
        <div id = "title"></div>
       
        <!-- <script src ="./entry.js"></script>
        <script src ="./entry2.js"></script> -->
    </body>
</html>

```

发布HTML需要使用html-webpack-plugin插件，需要本地安装
```
npm install --save-dev html-webpack-plugin
```
在webpack.config.js中配置html-webpack-plugin插件。
```js
const htmlPlugin= `require`('html-webpack-plugin');
    .
    .
    .

 plugins:[
    new htmlPlugin({
            minify:{
                removeAttributeQuotes:true
            },
            hash:true,
            template:'./src/index.html'
        })
    ]
```
- minify：是对html文件进行压缩
    - removeAttrubuteQuotes是却掉属性的双引号。 
- hash：为了开发中js有缓存效果，所以加入hash，这样可以有效避免缓存JS。
- template：是要打包的html模版路径和文件名称。
终端中使用webpack，进行打包。index.html文件已经被打包到dist目录下，并且自动为我们引入了路口的JS文件。

# 第八节 css中引入图片

在css中引入一张图片作为盒子的背景
```css
#img{
   background-image: url(../images/1.png);
   width:466px;
   height:453px;
}
```

安装file-loader和url-loader

`npm install --save-dev file-loader url-loader`

- file-loader：解决引用路径的问题，拿background样式用url引入背景图来说，webpack最终会将各个模块打包成一个文件，因此我们样式中的url路径是相对入口html页面的，而不是相对于原始css文件所在的路径的。这就会导致图片引入失败。这个问题是用file-loader解决的，file-loader可以解析项目中的url引入（不仅限于css），根据我们的配置，将图片拷贝到相应的路径，再根据我们的配置，修改打包后文件引用路径，使之指向正确的文件。

- url-loader：如果图片较多，会发很多http请求，会降低页面性能。这个问题可以通过url-loader解决。url-loader会将引入的图片编码，生成dataURl。相当于把图片数据翻译成一串字符。再把这串字符打包到文件中，最终只需要引入这个文件就能访问图片了。如果图片较大编码会消耗性能。因此url-loader提供了一个limit参数，小于limit字节的文件会被转为DataURl，大于limit的还会使用file-loader进行copy。

配置url-loader

loader使用时不需要用require引入，在plugins才需要使用require引入。

webpack.config.js文件
```js
 //模块：例如解读CSS,图片如何转换，压缩
    module:{
        rules: [
            {
              test: /\.css$/,
              use: [ 'style-loader', 'css-loader' ]
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
- test:/.(png|jpg|gif)/是匹配图片文件后缀名称。
- use:是指定使用的loader和loader的配置参数。
- limit:是把小于500000B的文件打成Base64的格式，写入JS。

url-loader封装了file-loader。url-loader不依赖于file-loader，即使用url-loader时，只需要安装url-loader即可，不需要安装file-loader，因为url-loader内置了file-loader，url-loader工作分两种情况：

- 1.文件大小小于limit参数，url-loader将会把文件转为DataURL（Base64格式）；

- 2.文件大小大于limit，url-loader会调用file-loader进行处理，参数也会直接传给file-loader。

也就是说，其实只安装一个url-loader就可以
