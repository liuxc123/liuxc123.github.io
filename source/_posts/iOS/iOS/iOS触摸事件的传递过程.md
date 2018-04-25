---
title: iOS触摸事件的传递过程
date: 2018-04-25 10:00:36
categoty:
- iOS
tags:
- iOS
---

当指肚轻触屏幕，整个系统像沉睡的生灵突然被惊醒，然后经历过腥风血雨的一段奇幻旅行，最终又归于沉寂。

整个iOS触摸事件从产生到寂灭大致如下图：

![](https://user-gold-cdn.xitu.io/2017/3/5/e31c8003e10eb77c7c08289cdf8c4370.png?imageView2/0/w/1280/h/960/ignore-error/1)

## 起始阶段

---> cpu处于睡眠状态，等待事件发生
---> 手指触摸屏幕

## 系统响应阶段

---> 屏幕硬件感应到输入，并将感应到的事件传递给输入输出驱动IOKit
—-> IOKit.framework封装整个触摸事件为IOHIDEvent对象
—-> IOKit.framework通过IPC将事件转发给SpringBoard.app

以上是系统层的响应。系统感应到外界的输入，并将相应的输入封装成比较概括的IOHIDEvent对象，然后UIKit通过[IOHIDEvent](http://iphonedevwiki.net/index.php/IOHIDFamily)的类型，判断出相应事件应该由[SpringBoard .app](http://iphonedevwiki.net/index.php/SpringBoard.app)处理，直接通过mach port(IPC进程间通信)转发给SpringBoard.app。

SpringBoard.app就是iOS的系统桌面，当触摸事件发生时，也只有负责管理桌面的SpringBoard.app才知道如何正确的响应。因为触摸发生时，有可能用户正在桌面翻页找App，也有可能正处于在微信中刷朋友圈。

## 桌面响应阶段

—-> SpringBoard.app主线程Runloop收到[IOKit.framewor](https://developer.apple.com/documentation/iokit)k转发来的消息苏醒，并触发对应Mach Port的Source1回调`__IOHIDEventSystemClientQueueCallback()`。

—-> 如果SpringBoard.app监测到有App在前台（记为xxxx.app），SpringBoard.app通过mach port(IPC进程间通信)转发给xxxx.app，如果SpringBoard.app监测到监测无前台App，则SpringBoard.app进入App内部响应阶段的第二段，记触发Source0回调。

## App内部响应阶段

—-> 前台App主线程Runloop收到SpringBoard.app转发来的消息苏醒，并触发对应Mach Port的Source1回调`__IOHIDEventSystemClientQueueCallback()`。
—-> Source1回调内部触发Source0回调`__UIApplicationHandleEventQueue()`
—-> Soucre0回调内部，封装`IOHIDEvent`为`UIEvent`
—-> Soucre0回调内部调用`UIApplication`的`sendEvent:`方法，将`UIEvent`传给`UIWindow`
—-> 平时开发熟悉的触摸事件响应链从这开始了
—-> 通过递归调用UIView层级的`hitTest(_:with:)`，结合`point(inside:with:)`找到`UIEvent`中每一个`UITouch`所属的`UIView`（其实是想找到离触摸事件点最近的那个`UIView`）。这个过程是从`UIView`层级的最顶层往最底层递归查询，但这不是`UIResponder`响应链，事件响应是在`UIEvent`中每一个`UITouch`所属的`UIView`都确定之后方才开始。

但需要注意，以下三种情况`UIView`的`hitTest(_:with:)`不会被调用，也导致其子`UIView`的`hitTest(_:with:)`不会被调用，而之后响应事件是下向上传递的，这直接导致以下三种情况的`UIView`及其子`UIView`不接收任何触摸事件：

1. userInteractionEnabled = NO
2. hidden = YES
3. alpha = 0.0~0.01之间

提示: UIImageView的userInteractionEnabled默认为NO,因此UIImageView以及它的子控件默认是不接收触摸事件的。

当把断点打在某个UIViewhitTest(_:with:)中时，对应的调用堆栈如下：

![](https://user-gold-cdn.xitu.io/2017/3/5/23b0e471d2279b2e4f0eeab8a4033fc0.png?imageView2/0/w/1280/h/960/ignore-error/1)

—-> 根据围绕`UITouch`所属的`UIView`及其祖先`UIView`的gesture recognizers，来确定一个UITouch的gestureRecognizers

—-> UITouch所属的UIView和gestureRecognizers收到此UITouch和相应的UIEvent，并按照UITouch所处的状态调用四大UITouch方法`touchesBegan(_:with:) touchesMoved(_:with:)` `touchesEnded(_:with:)` `touchesCancelled(_:with:)`中的一个。（事件响应开始）

—-> 对于UIView收到的UITouches事件（四大UITouch事件都是如此），则会按照UIResponder响应链一直往上传递，直到某个UIResponder因为主动响应触摸事件，切断了响应链（即不调用下一个UIResponder的响应方法），如果一直没有UIResponder做响应处理，则这些UITouches到达最后的响应者即UIApplication后，就被吃掉了，消失了。

—-> 如果在事件响应过程中，有UIGestureRecognizer成功识别，则此UIGestureRecognizer将独自占有所需要的UITouches，这些UITouches所属的UIView及其他的UIGestureRecognizer的`touchesCancelled(_:with:)`方法将调用（如果在手势的代理中设置可以同时识别两个手势，则允许同时识别的手势均可以收到所需要的UITouches事件）。但与识别成功的UIGestureRecognizer无关的UITouches则会继续按照上述传递逻辑传递。也即允许两个手势同时识别，只要所占有的UITouches不相同。

—-> 如果UIGestureRecognizer识别成功，则调用相应的action，处理对应的逻辑。如果某个UIResponder主动响应了触摸事件，则根据其本身的响应逻辑处理对应的业务，UIControl都是主动响应并切断UITouch的向上传递的。

—-> UITouches事件流动完毕，整个系统重新进入睡眠等待下一个事件

## 总结

从手指触碰到屏幕，UITouch大致经历三个阶段，`系统处理阶段---->SpringBoard.app处理阶段---->前台App处理阶段`，事实上日常开发只需知晓最后一个阶段即可，前两个阶段参考资料也不多，更多的还涉及系统底层，这里仅做简单介绍。

