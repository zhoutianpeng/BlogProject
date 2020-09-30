---
title: vue入门笔记1-从0开始搭建vue开发环境
date: 2020-01-06 00:03:53
categories:
 - [ 前端]
tags: 
 - [ vue]
 - [ nodejs]
---
# 从0开始搭建vue开发环境

该笔记记录在mac平台下搭建vue的开发环境，其中node是使用nvm安装管理（非必须）。

<!-- more -->

## 一、安装nvm
进入nvm的github仓库[nvm Github地址](https://github.com/nvm-sh/nvm)，详细安装过程可以参考README，遇到的问题基本在issues中可以找到解决方案，这里简单记录下过程：



执行nvm安装脚本: 
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash`

编辑bash_profile文件，添加如下配置:
```
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

新建terminal，输入`nvm --version`若输出版本号则安装nvm成功

## 二、安装node
这里使用nvm安装管理node版本

`nvm install v8.16.0`


## 三、安装&介绍vue工程使用的模块
1. webpackage
    - 工程打包模块，webpack会有单独的笔记详细记录使用
    - `npm install webpack -g`

2. vue-cli
    - 项目的构筑工具，它会自动为我们下载开发需要的一些环境，并生成项目的目录结构
    - vue-cli的2.x与3.x版本差异较大，该工程使用的是vue-cli3.x
    - vue-cli 2.x的使用
        - 安装`npm install vue-cli -g`
        - 创建项目 `vue init webpack 项目名`
    - vue-cli 3.x的使用
        - 安装`npm install @vue/cli@3.4.0 -g`
        - 创建项目 `vue create 项目名`

vue-cli2.x与3.x的差别从目录结构上看主要是3.x移除了`build`、`config`目录，大部分配置集成到了`vue.config.js`里面了，使用vue-cli3.x版本创建工程后若没有`vue.config.js`文件需要手动在工程的根目录下创建，`vue.config.js`文件负责配置常用的输出路径、根目录、预处理、devServer配置、pwa、dll、第三方插件；

目前vue-cli已经更新到了4.x, `npm install`模块时若不指定@版本号，npm默认会安装最新稳定的版本；

3. vue-router
    - 路由是单页应用的核心插件
    - `npm install vue-router`

4. vuex
    - 状态管理库，理解为全局数据集中地
    - 小项目不推荐使用，使用bus总线机制处理就可以
    - `npm install vuex`


5. axios（vue-resource 官方以及停止维护）
    - 使用Promise封装的ajax
    - `npm install axios`


6. cube-ui
    - 滴滴开发的移动端UI库
    - 对于vue-cli3.x以上版本在创建完工程后使用如下命令安装cube-ui
        `vue add cube-ui`

## 四、初始化项目
-  `vue create 项目名`

-  在根目录下创建`vue.config.js`，可以根据需要添加如下配置
```js
module.exports = {
    configureWebpack : {
        devServer : {
            port : 8089, 
            open : true, //启动项目后，自动在浏览器打开页面
        }
    }
```

- 启动项目
    `npm run serve`


