---
title: 23个经典设计模式的Swift实现
date: 2018-04-16 16:27:56
category: 
- iOS
- Design Mode
tags:
- iOS
- Swift
- Design Mode
---

## 前言
> 这是一篇主观的文章，文字解释也尽可能简单，写作目的是一次团队内的知识分享，想让不了解设计模式的同事迅速对这些生词混个脸熟。所以本文适合懂Swift语法，想快速了解23个设计模式大概在讲什么的同学。

## 基本结构
* 比喻 让我联想到的一些事物
* 官方定义 原版定义
* UML 不是原版UML 只保留了我觉得核心的部分
* 代码 Swift实现，这个是本体
* 讲解 假设已经看过代码的一些零散评注

## 目录
* Creational 创建型 5

    * Abstract Factory 抽象工厂模式
    * Builder 建造者模式
    * Factory Method 工厂方法模式
    * Prototype 原型模式
    * Singleton 单例模式

* Structural 结构型 7

    * Adapter 适配器模式
    * Bridge 桥接模式
    * Composite 组合模式
    * Decorator 装饰者模式
    * Facade 外观模式
    * Flyweight 享元模式
    * Proxy 代理模式


* Behavioral 行为型 11

    * Chain of responsibility 责任链模式
    * Command 命令模式
    * Interpreter 解释器模式
    * Iterator 迭代器模式
    * Mediator 中介模式
    * Memento 备忘录模式
    * Observer 观察者模式
    * State 状态模式
    * Strategy 策略模式
    * Template Method 模板方法模式
    * Visitor 访问者模式

-------

## 工厂模式
> 工厂模式顾名思义，就像一个工厂生产你所需要的产品

![](http://cdn1.alphr.com/sites/alphr/files/2016/02/tesla_factory_tour_1.jpg)
> 配图：http://cdn1.alphr.com/sites/alphr/files/2016/02/tesla_factory_tour_1.jpg

### 无工厂 Non-Factory

也就是工厂问题想解决的原始问题。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b674df37c35?imageView2/0/w/1280/h/960/ignore-error/1)

```
protocol Product{}
class ConcreteProductA: Produce {}
class ConcreteProductB: Produce {}

class Client {
    func createProduct(type: Int) -> Produce {
        if type == 0 {
            return ConcreteProductA()
        } else {
            return ConcreteProductB()
        }
    }
}

let c = Client()
c.createProduct(type: 0) // get ConcreteProductA
```
从代码和UML可以看出，为了得到产品A，调用者`Client`要同时依赖`Product`, `ConcreteProductA`和`ConcreteProductB`，并亲自写一个创建产品的方法。

每当需求新增一个产品，就要改动到调用方`Client`。如果这一堆创建代码如果可以抽离出去就好了，于是简单工厂出现了。

### 简单工厂 Simple Factory

简单工厂就做了一件事，**把Client要做的创建工作，挪到了另一个类里。**

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b6c46c23e40?imageView2/0/w/1280/h/960/ignore-error/1)


```
protocol Product{}
class ConcreteProductA: Produce {}
class ConcreteProductB: Produce {}

class Client {
    let s = Factory()
}

class Factory {
    func createProduct(type: Int) -> Product {
        if type == 0 {
            return ConcreteProductA()
        } else {
            return ConcreteProductA()
        }
    }
}

let c = Client()
c.s.createProduct(type: 0) // get ConcreteProductA
```

`Factory`代替了`Client`对具体`Product`的依赖，那么当需求变化的时候，我们不再需要改动调用方。这固然有所进步，但无法避免的是，**每次变动都要在createProduct的方法内部新增一个if-else分支**，这显然违背了[开闭原则](https://en.wikipedia.org/wiki/Open/closed_principle)。

为了解决这个问题，我们引入另一个模式。

### 工厂方法 Factory Method

官方定义

> 定义一个创建对象的接口，让其子类自己决定实例化哪一个工厂类，工厂模式使其创建过程延迟到子类进行。

![工厂方法](https://user-gold-cdn.xitu.io/2018/3/15/16228b7414ea9071?imageView2/0/w/1280/h/960/ignore-error/1)

```

protocol Product{}
class ConcreteProductA: Produce {}
class ConcreteProductB: Produce {}

class Client {
    let f = Factory()
}

class Factory {
    func createProduct() -> Product? { return nil } //用于继承
    func createProduct(type: Int) -> Product? { //用于调用
        if type == 0 {
            return ConcreteFactoryA().createProduct()
        } else {
            return ConcreteFactoryB().createProduct()
        }
    }
}

class ConcreteFactoryA: Factory {
    override func createProduct() -> Product? {
        // ... 产品加工过程
        return ConcreteProductA()
    }
}

class ConcreteFactoryB: Factory {
    override func createProduct() -> Product? {
        // ... 产品加工过程
        return ConcreteProductB()
    }
}

let c = Client()
c.f.createProduct(type: 0) // get ConcreteProductA
```

对于工厂方法的实现，有众多不同的解法，比如`Factory`只保留一个`createProduct`让子类实现，让`Client`来选择生成哪个具体工厂实例；或是引入一个FactoryMaker的中间层，作为生产工厂的“简单工厂”。我这里采用的方式是Factory既作为工厂父类，让具体工厂决定生产生么产品，又作为接口类，让`Client`可以通过[依赖注入](https://en.wikipedia.org/wiki/Dependency_injection)选择特定工厂。我这样做的目的是，在不引入新的中间层的情况下，最小化Client的依赖。

工厂方法在简单工厂的基础上做了两件事：

* 多了一层抽象，把生产产品的工作延迟到子类执行。
* 把“选择如何生产产品的工作”转化为“选择让哪个具体工厂生产”。

工厂方法的贡献在于，这样做虽然不能完美避免对一个if-else的扩展，但是这个扩展规模被极大限制住了（只需要new一个类）。

工厂方法着重点是解决了单一产品线的派生问题。那如果有多个相关产品线呢？

### 抽象工厂 Abstract Factory

官方定义

> 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228b775f6ff3af?imageView2/0/w/1280/h/960/ignore-error/1)

```
protocol ProductA {}
class ConcreteProductA1: ProductA {}
class ConcreteProductA2: ProductA {}

protocol ProductB {}
class ConcreteProductB1: ProductB {}
class ConcreteProductB2: ProductB {}

class Client {
    let f = Factory()
}

class Factory {
    func createProductA() -> ProductA? { return nil } // 用于继承
    func createProductB() -> ProductB? { return nil } // 用于继承
    func createProductA(type: Int) -> ProductA? { // 用于调用
        if type == 0 {
            return ConcreteFactory1().createProductA()
        } else {
            return ConcreteFactory2().createProductA()
        }
    }
    func createProductB(type: Int) -> ProductB? { // 用于调用
        if type == 0 {
            return ConcreteFactory1().createProductB()
        } else {
            return ConcreteFactory2().createProductB()
        }
    }
}

class ConcreteFactory1: Factory {
    override func createProductA() -> ProductA? {
        // ... 产品加工过程
        return ConcreteProductA1()
    }
    override func createProductB() -> ProductB? {
        // ... 产品加工过程
        return ConcreteProductB1()
    }
}

class ConcreteFactory2: Factory {
    override func createProductA() -> ProductA? {
        // ... 产品加工过程
        return ConcreteProductA2()
    }
    override func createProductB() -> ProductB? {
        // ... 产品加工过程
        return ConcreteProductB2()
    }
}

let c = Client()
c.f.createProductA(type: 0) // get ConcreteProductA1
c.f.createProductA(type: 1) // get ConcreteProductA2
c.f.createProductB(type: 0) // get ConcreteProductB1
c.f.createProductB(type: 1) // get ConcreteProductB2
```
图很吓人，其实很简单。

当我们有两个相关的产品线`ProductA`和`ProductB`, 例如螺丝和螺母，他们派生出`ProductA1`，`ProductB1` 和 `ProductA2`，`ProductB2`，前者我们由工厂`ConcreteFactory1`来制作，后者由 `ConcreteFactory2`来制作。

对于`Client`来说，他只需要知道有一个抽象的工厂能同时生产`ProductA`和`ProductB`就行了，那就是图中的Factory。

**重点来了**，这个抽象的`Factory`是通过“工厂方法”模式把构造过程延迟到子类执行的，也就是说，抽象工厂是建立在工厂方法的基础上的模式。
所以抽象工厂，换句话说，就是多个产品线需要绑定在一起，形成一个抽象的综合工厂，由具体的综合工厂来批量实现“工厂方法”的一种更“高级”的模式。

#### 总结

有点绕，说完这些感觉我已经中文十级了。总之，我想表达的观点是：**这些工厂模式并不是割裂的存在，而是一个递进的思想。**

-------

## Builder 建造者模式

> 建造者模式就像你委托一个室内设计师装修你的新家

![](http://tse2.mm.bing.net/th?id=OIP.N3hIhcOq32Bh6Ezi6q6kGwHaFj&pid=Api)

> 配图: http://tse2.mm.bing.net/th?id=OIP.N3hIhcOq32Bh6Ezi6q6kGwHaFj&pid=Api

**官方定义**

> 将一个复杂的构建与其表示相分离，使得同样的构建过程可以创建不同的表示。

如果说之前的谈到的工厂模式是把创建产品(对象)的工作抽离出去的话，这次要聊的建造者模式，就是把产品内部的组件生产工作抽离出去。这样做的场景，适用于那些有着复杂、规则模块的对象生成流程。换句话说，工厂模式是一个类(工厂)创建另一个类(产品)，而建造者是一个类(产品)自身的属性(组件)构造过程。

对于建造者模式的实现网上也是版本不一：复杂点的版本会引入一个Director的角色，做一个整体上下文，组装更傻瓜化的builder和product。或是抽象一层Builder协议，用不同的具体Builder来构造不同的产品。但我认为这些都模糊了这个模式要传达的焦点，对理解没有帮助，所以这里我选择一个极简的模型。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bd251c56c84?imageView2/0/w/1280/h/960/ignore-error/1)

```
struct Builder {
    var partA: String
    var partB: String
}

struct Product {
    var partA: String
    var partB: String
    init(builder: Builder) {
        partA = builder.partA
        partB = builder.partB
    }
}

// 通过builder完成产品创建工作
let b = Builder(partA: "A", partB: "B")
// 这样产品只需要一个builder就可以完成制作
let p = Product(builder: b)
```

我们让Product的生成由自己发起，但是它的组件(属性)全都委托给Builder来实现，而它只需要依赖一个Builder就完成了自身的生产工作。

-------

## Prototype 原型模式

> 原型模式让你有了一个可以源源不断自我赋值的类。

![](http://thumbs.dreamstime.com/z/cell-division-two-cells-divide-osmosis-background-other-cells-48181492.jpg)
> 配图: http://thumbs.dreamstime.com/z/cell-division-two-cells-divide-osmosis-background-other-cells-48181492.jpg

**官方定义**

> 用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。

原型模式很简单，你只要实现一个**返回你自己的新对象**的方法即可。这里我采用的实现还不是最简单的，这个interface并不是必须的。

原型模式实现了深拷贝。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228be892b4b350?imageView2/0/w/1280/h/960/ignore-error/1)

```
protocol Prototype {
    func clone() -> Prototype
}

struct Product: Prototype {
    var title: String
    func clone() -> Prototype {
        return Product(title: title)
    }
}

let p1 = Product(title: "p1")
let p2 = p1.clone()
(p2 as? Product)?.title // OUTPUT: p1
```

-------

## Singleton 单例模式

> 单例就像一个公司的IT部门，他们是唯一的存在，并且被所有人直接访问。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bef47904c11?imageslim)
配图：The IT Crowd

**官方定义**
> 保证一个类仅有一个实例，并提供一个访问它的全局访问点。

因为第二点常常被忽视，所以过度使用的危害极大，你无从知道调用从何而来，这种`goto`一般的存在会变成维护的噩梦。

单例比较常见的应用是例如数据库，网络框架的全局访问点。

单例其实就是**变种**的原型模式，只不过原型每次返回的是一个拷贝。

![](https://user-gold-cdn.xitu.io/2018/3/15/16228bf3071227f0?imageView2/0/w/1280/h/960/ignore-error/1)

Swift的简单实现：

```
class Singleton {
    static let sharedInstance = Singleton()
    private init() {
        // 用private防止被new
    }
}
let s  = Singleton.sharedInstance
let s2 = Singleton() // ERROR: initializer is inaccessible due to 'private' protection level
```

Swift的完整实现：

```
class Singleton {
    static var singleton: Singleton? = nil
    private init() {}
    static func sharedInstance() -> Singleton {
        if singleton == nil {
            singleton = Singleton()
        }
        return singleton!
    }
}

let s = Singleton.sharedInstance()
let s2 = Singleton() // ERROR: initializer is inaccessible due to 'private' protection level
```

