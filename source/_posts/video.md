---
title: 浏览器播放video踩坑
date: 2020-02-12 23:02:53
categories:
 - [前端]
tags: 
---

## 前言

这一切都要从一个简单的example讲起，就是这个[PIXI播放视频](https://pixijs.io/examples/#/sprite/video.js)，简单来看这就是在页面中播放了一个视频，其本质并不是流媒体形式直接解码播放视频，这个稍后在聊。这篇博文主要是笔者分享了作为前端小白在开发视频相关需求遇到的各种问题，如果需要干货总结，跳过笔者的废话[请戳这里](##summry)。

<!-- more -->

### 小时候，播放视频是一个小小的需求，我在这边，W3C在那边。

### 长大后，播放视频是一个兼容性挑战，我这这边，三观在那边。

令我比较自闭的是，作为浏览器咱能不能稍微重视一下W3C的标准，同一个接口在不同浏览器表现的五花八门，同样是程序员自己人何苦为难自己人。每次在一个新的平台测试代码，效果永远令你先惊喜再自闭，逛各路论坛博客看到issue中使用者与开发者的博弈像极了你在和产品经理极力狡辩这个需求做不了时的情形。

## pixiJS播放video

### 一、pixiJS简介
最开始了解开发内容时~~觉得很简单~~这辈子我再不会有这种错觉了，就是使用pixiJS播放video的需求，pixiJS是一个浏览器端的高性能2D渲染引擎，本质上是对webGL进行了封装，可以让开发者快速开发出交互式图形、动画、游戏等应用。

第一次逛pixiJS的官网时，翻阅examples其实并没有看到特别多有意思的demo，尤其是相比于threeJS的examples，个人理解原因在于这两个项目的定位并不同，不是直观的2D与3D的区别，pixiJS的定位是`The HTML5 Creation Engine`，而threeJS的定位则是`JavaScript 3D library`，作为渲染引擎pixiJS更加注重渲染的高效性，pixi通过一个清晰的树状结构表达了显示元素之间的关系，并在底层针对webGL渲染顺序方式进行了优化，因此也能够找到许多使用pixi开发的优秀应用。

### 二、视频demo跑不起来？
回到需求的开发使用pixiJS播放视频，准确的说是使用pixiJS将视频作为材质texture赋给spirit从而实现了视频的播放，官方提供了可展示的example以及文档，就是前言中引用的链接，所以~~复制粘贴~~实现起来还是比较容易的，但运行时遇到了第一个问题就是demo本地跑不起来，经过排查确定是只有chrome浏览器运行代码会报错，而safair和火狐运行没有问题，在chrome中会抛出如下错误：
```
WebGL: INVALID_VALUE: tex(Sub)Image2D: video visible size is empty
[.WebGL-0x7f861a496800]GL ERROR :GL_INVALID_OPERATION : glTexSubImage2D: level 0 does not exist
```
查询了issue发现也有开发者遇到了相同的问题，而pixi官方也是很给力的把这个BUG修复了，需要升级为pixi 5.x版本就可以解决该问题。

这是git上中关于该BUG的issue，可以认为出现问题的原因是chrome的版本升级

https://github.com/pixijs/pixi.js/issues/5996

https://github.com/pixijs/pixi.js/pull/6088

### 三、视频跑起来太卡？
跑起demo仅仅是第一关，接下来磨人的问题才慢慢出现，首先也是比较棘手的问题就是这个视频播放太卡了，在PC端并没有出现明显的问题，但是在移动端就彻底暴露了，运行的效果用眨眼补帧也救不了，仿佛是在看ppt，看来是需要优化的时候了，等等～我好像也没怎么写代码呀？关于pixiJS的video介绍的资料少得可怜，只能自己动手丰衣足食了，大致的看了下pixiJS的源代码，简单的梳理了一下pixi实现视频材质的过程：pixiJS创建一个`video`元素用于加载视频，每次渲染画面时调用`update()`渲染当前帧的材质，`update()`内部做的事情是利用`video`解码出当前的视频帧然后将生成texture，`update()`是实时将视频帧传递给GPU渲染出texture，函数本身比较慢并且对GPU有所依赖，怀疑这里是瓶颈。

同时也查询了论坛发现了也有开发者人遇到相似的问题，在移动端进行视频渲染会卡顿
> Uploading a new texture to the GPU for each frame sounds like a recipe for lag on mobile. Isn't it? (although it makes sense that's what you have to do in order to display a video)
>
>That's precisely the reason we use prepare for images before displaying them, or do videos have a different solution behind the hood that take care of making things smooth?

论坛中有人提议是否可以提前生成视频材质而不是实时的渲染材质，来避免移动端的计算行能不足导致的卡顿，好在pixiJS的Moderator出来回了这个帖子帮助我们死了这个心

>>I guess my question is - was PIXI video developed with cordova in mind? Is it supposed to work smoothly on cordova?
>
>I think you are misunderstanding. This is not a limitation of PixiJS. This is how WebGL works. We have to upload each frame to render it, there is no other option. Either that works for your use case or it doesn't, but either way there isn't anything we can do about it.
>
>Specifically answering your question: No, PixiJS was not built specifically to work on cordova. We built it to work in the browser, on mobile and desktop. However, we do try to support Ejecta, CocoonJS, Cordova, et al.

从中我们得到信息是提前渲染目前应该无法实现，webGL播放video的方式就是实时的渲染出每一个视频帧材质，即使尝试降低视频的fps以及质量也未获得明显的效果。至此决定换一种方式来播放视频。

这里放一下相关资料的链接
`https://www.html5gamedevs.com/topic/32147-prepare-and-destroy-texture-from-video/`


未完待更🐤🐤🐤


<!--
## canvas渲染video

### 一、实验demo



pixiJS工程中有一个集中加载资源的loader，这个loader不是pixi自己实现的而是使用了开源项目 https://github.com/englercj/resource-loader ，pixi根据自己的需求写了一些中间件挂载到loader中，这些中间件主要是作用是处理图片类型的资源，我记得好想是在load完图片资源后直接处理为texture，对于视频资源loader的做法是创建一个video元素将其src属性设置为资源的URL，



在确认逻辑没有错误后我与官方的example比较了一下，发现是因为pixi版本不同，example的代码使用pixi5以前的版本是跑不起来的，本地安装的是pixi4

找到问题的原因但是还不知道为什么，查阅了资料以及pixi项目github的issue后
-->

## <span id = "summry">开发总结</span>
我还没写

