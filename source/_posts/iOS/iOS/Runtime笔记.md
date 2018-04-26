---
title: Runtime笔记
date: 2018-04-26 10:41:58
category:
- iOS
tags:
- iOS
- Objective-C
- Runtime
---

## 前言

iOS开发也有几年了，之前一直听说有runtime，也看过其他框架源码中使用过，但是对于具体的原理还是比较模糊，这次通过此次分享让大家加深对于runtime理解。

Runtime的特性主要是消息(方法)传递，如果消息(方法)在对象中找不到，就进行转发，具体怎么实现的呢。我们从下面几个方面探寻Runtime的实现机制。

* Runtime介绍
* Runtime消息传递
* Runtime消息转发
* Runtime应用

## Runtime介绍

> Objective-C 扩展了 C 语言，并加入了面向对象特性和 Smalltalk 式的消息传递机制。而这个扩展的核心是一个用 C 和 编译语言 写的 Runtime 库。它是 Objective-C 面向对象和动态机制的基石。

> Objective-C 是一个动态语言，这意味着它不仅需要一个编译器，也需要一个运行时系统来动态得创建类和对象、进行消息传递和转发。理解 Objective-C 的 Runtime 机制可以帮我们更好的了解这个语言，适当的时候还能对语言进行扩展，从系统层面解决项目中的一些设计或技术问题。了解 Runtime ，要先了解它的核心 - 消息传递 （Messaging）。

`Runtime`其实有两个版本: “`modern`” 和 “`legacy`”。我们现在用的 `Objective-C 2.0` 采用的是现行 (`Modern`) 版的 Runtime 系统，只能运行在 `iOS` 和 `macOS 10.5` 之后的 `64` 位程序中。而 macOS 较老的32位程序仍采用 `Objective-C 1` 中的（早期）`Legacy` 版本的 `Runtime` 系统。这两个版本最大的区别在于当你更改一个类的实例变量的布局时，在早期版本中你需要重新编译它的子类，而现行版就不需要。

`Runtime` 基本是用 `C` 和`汇编`写的，可见苹果为了动态系统的高效而作出的努力。你可以在[这里](https://opensource.apple.com/source/objc4/)下到苹果维护的开源代码。苹果和GNU各自维护一个开源的 [runtime](https://github.com/RetVal/objc-runtime) 版本，这两个版本之间都在努力的保持一致。

平时的业务中主要是使用[官方Api](https://developer.apple.com/documentation/objectivec/objective_c_runtime#//apple_ref/doc/uid/TP40001418-CH1g-126286)，解决我们框架性的需求。

高级编程语言想要成为可执行文件需要先编译为汇编语言再汇编为机器语言，机器语言也是计算机能够识别的唯一语言，但是`OC`并不能直接编译为汇编语言，而是要先转写为纯C语言再进行编译和汇编的操作，从`OC`到`C`语言的过渡就是由runtime来实现的。然而我们使用OC进行面向对象开发，而`C`语言更多的是面向过程开发，这就需要将面向对象的类转变为面向过程的结构体。

## Runtime消息传递

一个对象的方法像这样`[obj foo]`，编译器转成消息发送`objc_msgSend(obj, foo)`，`Runtime`时执行的流程是这样的：

* 首先，通过`obj`的`isa`指针找到它的`class`；
* 在`class`的`method list`找`foo`；
* 如果`class`中没有找到`foo`，继续往它的`superclass`中找；
* 一但找到`foo`这个函数，就去执行它的实现`IMP`

但这种实现有个问题，效率低。但是一个`class`往往只有`20%`的函数会被经常调用，可能占总调用次数的`80%`。每个消息都要遍历一次`objc_method_list`并不合理。如果把经常被调用的函数缓存下来，那可以大大提高函数查询的效率。这也就是`objc_class`中另一个重要成员`objc_cache`做的事情 - 再找到`foo`之后，把`foo`的`method_name`作为`key`，`method_imp`作为`value`给存起来。当再次收到`foo`消息的时候，可以直接在`cache`里找到，避免去遍历`objc_method_list`。从前面的源代码可以看到`objc_cache`是存在`objc_class`结构体中的。

objec_msgSend的方法定义如下：

```
OBJC_EXPORT id objc_msgSend(id self, SEL op, ...)
```

那消息传递是怎么实现的呢？我们看看对象(object)，类(class)，方法(method)这几个的结构体：

```
//对象
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};
//类
struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;
#if !__OBJC2__
    Class super_class                                        OBJC2_UNAVAILABLE;
    const char *name                                         OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
#endif
} OBJC2_UNAVAILABLE;
//方法列表
struct objc_method_list {
    struct objc_method_list *obsolete                        OBJC2_UNAVAILABLE;
    int method_count                                         OBJC2_UNAVAILABLE;
#ifdef __LP64__
    int space                                                OBJC2_UNAVAILABLE;
#endif
    /* variable length structure */
    struct objc_method method_list[1]                        OBJC2_UNAVAILABLE;
}                                                            OBJC2_UNAVAILABLE;
//方法
struct objc_method {
    SEL method_name                                          OBJC2_UNAVAILABLE;
    char *method_types                                       OBJC2_UNAVAILABLE;
    IMP method_imp                                           OBJC2_UNAVAILABLE;
}
```

1. 系统首先找到消息的接收对象，然后通过对象的`isa`找到它的类。
2. 在它的类中查找`method_list`，是否有`selector`方法。
3. 没有则查找父类的`method_list`。
4. 找到对应的`method`，执行它的`IMP`。
5. 转发`IMP`的`return`值。

下面讲讲消息传递用到的一些概念：

* 类对象(objc_class)
* 实例(objc_object)
* 元类(Meta Class)
* Method(objc_method)
* SEL(objc_selector)
* IMP
* 类缓存(objc_cache)
* Category(objc_category)

### 类对象(objc_class)

`Objective-C`类是由`Class`类型来表示的，它实际上是一个指向`objc_class`结构体的指针。

```
typedef struct objc_class *Class;
```
查看`objc/runtime.h`中`objc_class`结构体的定义如下：

```
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    long instance_size                                       OBJC2_UNAVAILABLE;
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
```

`struct objc_class`结构体定义了很多变量，通过命名不难发现，结构体里保存了指向父类的指针、类的名字、版本、实例大小、实例变量列表、方法列表、缓存、遵守的协议列表等，一个类包含的信息也不就正是这些吗？没错，类对象就是一个结构体`struct objc_class`，这个结构体存放的数据称为元数据（`motadata`），该结构体的第一个成员变量也是`isa`指针，这就说明了`Class`本身其实也是一个对象，因此我们称之为类对象，类对象在编译期产生用于创建实例对象，是单例。

### 实例(objc_object)

```
/// Represents an instance of a class.
struct objc_object {
    Class isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```

类对象中的元数据存储的都是如何创建一个实例的相关信息，那么类对象和类方法应该从哪里创建呢？
就是从`isa`指针指向的结构体创建，类对象的isa指针指向的我们称之为元类(`metaclass`)，

![](https://user-gold-cdn.xitu.io/2018/4/1/1628088a3e4f0167?imageView2/0/w/1280/h/960/ignore-error/1)

### 元类(Meta Class)

通过上图我们可以看出整个体系构成了一个自闭环，`struct objc_object`结构体`实例`它的`isa`指针指向类对象，类对象的`isa`指针指向了元类，`super_class`指针指向父类的类对象，而元类的`super_class`指针指向了父类的元类，那元类的`isa`指针又指向了自己。

元类(Meta Class)是一个类对象的类。在上面我们提到，所有的类自身也是一个对象，我们可以向这个对象发送消息(即调用类方法)。为了调用类方法，这个类的isa指针必须指向一个包含这些类方法的一个`objc_class`结构体。这就引出了`meta-class`的概念，元类中保存了创建类对象以及类方法所需的所有信息。任何`NSObject`继承体系下的`meta-class`都使用`NSObject`的`meta-class`作为自己的所属类，而基类的`meta-class`的`isa`指针是指向它自己。



