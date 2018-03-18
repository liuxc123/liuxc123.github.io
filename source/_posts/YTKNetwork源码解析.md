---
title: YTKNetwork源码解析
date: 2018-03-18 18:03:05
category: 
- iOS
tags: 
- iOS
- Objective-C
- 源码解析 
---

作为一名iOS开发工作者，大家应该都听过[YTKNetwork框架](https://github.com/yuantiku/YTKNetwork)吧。它是猿题库技术团队开源的一个网络请求框架，内部封装了AFNetworking。它把每个请求实例化，管理它的生命周期，也可以管理多个请求。

在正式讲解源码之前，我会先讲一下该框架所用的架构和设计模式。我总觉得对架构和设计有一定的了解的话，会有助于对源码的理解。

## 1. 架构

-------
先上图：

![](http://upload-images.jianshu.io/upload_images/859001-054321f909402be5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 在这里简单说明一下：

> 1. YTKNetwork框架将每一个请求实例化，YTKBaseRequest是所有请求类的基类，YTKRequest是它的子类。所以如果我们想要发送一个请求，则需要创建并实例化一个继承于YTKRequest的自定义的请求类（CustomRequest）并发送请求。
> 2. YTKNetworkAgent是一个单例，负责管理所有的请求类（例如CustomRequest）。当CustomRequest发送请求以后，会把自己放在YTKNetworkAgent持有的一个字典里，让其管理自己。
> 3. 我们说YTKNetwork封装了AFNetworking，实际上是YTKNetworkAgent封装了AFNetworking，由它负责AFNetworking请求的发送和AFNetworking的回调处理。所以如果我们想更换一个第三方网络请求库，就可以在这里更换一下。而YTKRequest更多的是只是负责缓存的处理。
> 4. YTKNetworkConfig与YTKPriviate的具体职能现在不做介绍，会在后文给出。

OK，现在我们知道了YTKNetwork中类与类之间的关系以及关键类的大致职能，接下来我会告诉你YTKNetwork为什么会采用这种关系来架构，以及采用这种架构会有什么好处。


