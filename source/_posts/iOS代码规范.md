---
title: iOS代码规范
date: 2018-03-15 19:59:23
category: hexo
tags: iOS
---
![](http://upload-images.jianshu.io/upload_images/859001-a9d78ce3d5e7114f.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

利用业余时间，整理出来了这份规范，我会将这份规范作为以后我们团队的代码规范，并且我也会根据读者的反馈以及项目的实践不定时更新，希望大家多多指正批评。

这篇规范一共分为三个部分：
1. 核心原则：介绍这篇代码规范所遵循的核心原则。
2. 通用规范：不局限iOS的通用性的代码规范（使用C语言和Swift语言）
3. iOS规范：仅适用于iOS的代码规范（使用Objective-C）

## 一、核心原则
### 原则一：代码应该简洁易懂，逻辑清晰
以为软件是需要人来维护的。这个人在未来很可能不是你。所以首先是为人编写程序，其次才是计算机。

* 不要分追求技巧，降低程序的可读性
* 简洁的代码可以让bug无处藏身。要写出明显没有bug的代码，而不是没有明显bug的代码。

### 原则二：面向变化编程，而不是面向需求编程
需求是暂时的，只有变化才是永恒的。
本次迭代不能仅仅为了当前的需求，要写出拓展性强，易修改的程序才是责任的做法，对自己负责，对公司负责。

### 原则三：先保证程序的正确性，防止过度工程
过度工程（over-engineering）：在正确可用的代码写出之前就过度地考虑拓展，重用的的问题，使得工程过度复杂。
引用《王垠：编程的智慧》里的话：

> 1. 先把眼前的问题解决掉，解决好，再考虑将来的扩展问题。
> 2. 先写出可用的代码，反复推敲，再考虑是否需要重用的问题。
> 3. 先写出可用，简单，明显没有bug的代码，再考虑测试的问题。

## 通用规范
### 关于大括号

-------
* 控制语句(if,for,while,switch)中，大括号开始与行尾
* 函数中，大括号要开始于行首

推荐这样写：

```
//控制语句
white(someCondition){

}

//函数
void function(param1,param2)
{

}
```

### 运算符

-------

#### 1.运算符与变量之间的间隔
##### 1.1 一元运算符与变量之间没有空格：
```
!bValue
~iValue
++iCount
*strSource
&fSum
```
##### 1.2 二元运算符与变量之间必须有空格：

```
fWidth = 5 + 5;
fLength = fWidth * 2;
fHeight = fWidth + fLength;
for(int i = 0; i < 10; i++)
```
#### 2. 多个不同的运算符同时存在时应该使用括号来明确优先级
在多个不同的运算符同时存在的时候应该合理使用括号，不要盲目依赖操作符优先级。
因为有的时候不能保证阅读你代码的人就一定能了解你写的算式里面所有操作符的优先级。

来看一下这个算式：2 << 2 + 1 * 3 - 4

这里的`<<`是移位操作直观上却很容易认为它的优先级很高，所以就把这个算式误认为：(2 << 2) + 13 - 4
但事实上，它的优先级是比加减法还要低的，所以该算式应该等同于：2 << (2 + 13 - 4).
所以在以后写这种复杂一点的算式的时候，尽量多加一点括号，避免让其他人误解（甚至是自己）。

### 变量

-------
#### 1.一个变量有且只有一个功能，尽量不要把一个变量用作多种用途
#### 2.变量在使用前应该初始化，防止未初始化的变量被引用
#### 3.局部变量应该尽量接近使用它的地方
推荐这样写：

```
func someFunction() {
 
  let index = ...;
  //Do something With index
  ...
  ...
  
  let count = ...;
  //Do something With count
  
}
```
不推荐这样写：

```
func someFunction() {
 
  let index = ...;
  let count = ...;
  //Do something With index
  ...
  ...
  
  //Do something With count
}
```

### if语句

-------

#### 1.必须列出所有的分支（列举所有的情况），而且每个分支都必须给出明确的结果。
推荐这样写：

```
var hintStr;
if (count < 3) {
  hintStr = "Good";
} else {
  hintStr = "";
}
```
不推荐这样写：

```
var hintStr;
if (count < 3) {
 hintStr = "Good";
}
```
#### 2.不要使用过多的分支，要善于使用return来提前返回错误的情况
推荐这样写：

```
- (void)someMethod { 
  if (!goodCondition) {
    return;
  }
  //Do something
}
```
不推荐这样写：

```
- (void)someMethod { 
  if (goodCondition) {
    //Do something
  }
}
```
比较典型的例子：

```
-(id)initWithDictionary:(NSDictionary*)dict error:(NSError)err
{
   //方法1. 参数为nil
   if (!dict) {
     if (err) *err = [JSONModelError errorInputIsNil];
     return nil;
    }
    //方法2. 参数不是nil，但也不是字典
    if (![dict isKindOfClass:[NSDictionary class]]) {
        if (err) *err = [JSONModelError errorInvalidDataWithMessage:@"Attempt to initialize JSONModel object using initWithDictionary:error: but the dictionary parameter was not an 'NSDictionary'."];
        return nil;
    }
    //方法3. 初始化
    self = [self init];
    if (!self) {
        //初始化失败
        if (err) *err = [JSONModelError errorModelIsInvalid];
        return nil;
    }
    //方法4. 检查用户定义的模型里的属性集合是否大于传入的字典里的key集合（如果大于，则返回NO）
    if (![self __doesDictionary:dict matchModelWithKeyMapper:self.__keyMapper error:err]) {
        return nil;
    }
    //方法5. 核心方法：字典的key与模型的属性的映射
    if (![self __importDictionary:dict withKeyMapper:self.__keyMapper validation:YES error:err]) {
        return nil;
    }
    //方法6. 可以重写[self validate:err]方法并返回NO，让用户自定义错误并阻拦model的返回
    if (![self validate:err]) {
        return nil;
    }
    //方法7. 终于通过了！成功返回model
    return self;
}
```
> 可以看到，在这里，首先判断出各种错误的情况然后提前返回，把最正确的情况放到最后返回。

#### 3.条件表达式如果很长，则需要将他们提取出来赋值给一个BOOL值

推荐这样写：

```
let nameContainsSwift = sessionName.hasPrefix("Swift")
let isCurrentYear = sessionDateCompontents.year == 2014
let isSwiftSession = nameContainsSwift && isCurrentYear
if (isSwiftSession) { 
   // Do something
}
```
不推荐这样写：

```
if ( sessionName.hasPrefix("Swift") && (sessionDateCompontents.year == 2014) ) { 
    // Do something
}
```
#### 4.条件语句的判断应该是变量在左，常量在右
推荐这样写：

```
if ( count == 6) {
}
```
或者

```
if ( object == nil) {
}
```
或者

```
if ( !object ) {
}
```

不推荐这样写：

```
if ( 6 == count) {
}
```
或者

```
f ( nil == object ) {
}
```

#### 5.每个分支必须的实现代码必须用大括号包围
推荐这样写：

```
if (!error) {
  return success;
}
```

不推荐这样写：
```
if (!error)
    return success;
```
或者

```
if (!error) return success;
```

#### 6.条件过多，过长的时候应该换行
推荐这样写：

```
if (condition1() && 
    condition2() && 
    condition3() && 
    condition4()) {
  // Do something
}
```

不推荐这样写：

```
if (condition1() && condition2() && condition3() && condition4()) {
  // Do something
}
```
### for语句

-------

#### 1.不可在for循环内修改循环变量，防止for循环失去控制。

```
for (int index = 0; index < 10; index++){
   ...
   logicToChange(index)
}
```
#### 2.避免使用continue和break。
continue和break做描述的是“什么时候不做什么”，所以为了读懂二者所在的代码，我们需要在头脑里将他们取反。

其实最好不要让这两个东西出现，移位我们的代码只要体现出“什么时候做什么”就好了，而且通过适当的方法，是可以将这两个东西消灭掉的；

##### 2.1 如果出现了continue，只需要把continue的条件取反即可

```
var filteredProducts = Array<String>()
for level in products {
    if level.hasPrefix("bad") {
        continue
    }
    filteredProducts.append(level)
}
```
我们可以看到，通过判断字符串里是否还有“bad”这个prefix老过滤掉一些值。其实我们是可以通过取反，来避免使用continue的：

```
for level in products {
    if !level.hasPrefix("bad") {
      filteredProducts.append(level)
    }
}
```

##### 2.2 消除while里的break；讲break的条件取反，合并并到主循环里
在while里的block其实就相当于“不存在”，既然是不存在的东西就完全可以子啊最开始的条件语句中将其排除。

while里的break:

```
while (condition1) {
  ...
  if (condition2) {
    break;
  }
}
```
取反并合并到主条件：

```
while (condition1 && !condition2) {
  ...
}
```
##### 2.3 在有返回值的方法里消除break：将break转换为return立即返回
有些朋友喜欢这样做：在有返回值的方法里break之后，再返回某个值。其实完全可以在break的那一行直接返回。

```
func hasBadProductIn(products: Array<String>) -> Bool {
    var result = false    
    for level in products {
        if level.hasPrefix("bad") {
            result = true
        }
    }
   return result
}
```
遇到错误条件直接返回：

```
func hasBadProductIn(products: Array<String>) -> Bool {
    for level in products {
        if level.hasPrefix("bad") {
            return true
        }
    }
   return false
}
```
这样写的话不用特意声明一个变量来特意保存需要返回的值，看起来非常简洁，可读性高。

### Switch语句

-------
#### 1. 每个分支都必须用大括号括起来

推荐这样写：

```
switch (integer) {  
  case 1:  {
    // ...  
   }
    break;  
  case 2: {  
    // ...  
    break;  
  }  
  case 3: {
    // ...  
    break; 
  }
  default:{
    // ...  
    break; 
  }
}
```

#### 2. 使用枚举类型时，不能有default分支， 除了使用枚举类型以外，都必须有default分支

```
RWTLeftMenuTopItemType menuType = RWTLeftMenuTopItemMain;  
switch (menuType) {  
  case RWTLeftMenuTopItemMain: {
    // ...  
    break; 
   }
  case RWTLeftMenuTopItemShows: {
    // ...  
    break; 
  }
  case RWTLeftMenuTopItemSchedule: {
    // ...  
    break; 
  }
}
```
在Switch语句使用枚举类型的时候，如果使用了default分支，在将来就无法通过编译器来检查新增的枚举类型了。

### 函数

-------

#### 1. 一个函数的长度必须限制在50行以内
常来说，在阅读一个函数的时候，如果视需要跨过很长的垂直距离会非常影响代码的阅读体验。如果需要来回滚动眼球或代码才能看全一个方法，就会很影响思维的连贯性，对阅读代码的速度造成比较大的影响。最好的情况是在不滚动眼球或代码的情况下一眼就能将该方法的全部代码映入眼帘。

#### 2. 一个函数只做一件事（单一原则）
每个函数的职责都应该划分的很明确（就像类一样）。

推荐这样写：

```
dataConfiguration()
viewConfiguration()
```
不推荐这样写：

```
void dataConfiguration()
{   
   ...
   viewConfiguration()
}
```

#### 3. 对于有返回值的函数（方法），每一个分支都必须有返回值

推荐这样写：

```
int function()
{
    if(condition1){
        return count1
    }else if(condition2){
        return count2
    }else{
       return defaultCount
    } 
}
```
不推荐这样写：

```
int function()
{
    if(condition1){
        return count1
    }else if(condition2){
        return count2
    }
}
```

#### 4. 对输入参数的正确性和有效性进行检查，参数错误立即返回

推荐这样写：

```
void function(param1,param2)
{
      if(param1 is unavailable){
           return;
      }
    
      if(param2 is unavailable){
           return;
      }
     //Do some right thing
}
```

#### 5. 如果在不同的函数内部有相同的功能，应该把相同的功能抽取出来单独作为另一个函数

原来的调用：

```
void logic() {
  a();
  b()；
  if (logic1 condition) {
    c();
  } else {
    d();
  }
}
```

将a，b函数抽取出来作为单独的函数

```
void basicConfig() {
  a();
  b();
}
  
void logic1() {
  basicConfig();
  c();
}
void logic2() {
  basicConfig();
  d();
}
```

#### 6. 将函数内部比较复杂的逻辑提取出来作为单独的函数

一个函数内的不清晰（逻辑判断比较多，行数较多）的那片代码，往往可以被提取出去，构成一个新的函数，然后在原来的地方调用它这样你就可以使用有意义的函数名来代替注释，增加程序的可读性。

举一个发送邮件的例子：

```
openEmailSite();
login();

writeTitle(title);
writeContent(content);
writeReceiver(receiver);
addAttachment(attachment);

send();
```
中间的部分稍微长一些，我们可以将它们提取出来：

```
void writeEmail(title, content,receiver,attachment)
{
  writeTitle(title);
  writeContent(content);
  writeReceiver(receiver);
  addAttachment(attachment); 
}
```
然后再看一下原来的代码：

```
openEmailSite();
login();
writeEmail(title, content,receiver,attachment)
send();
```

#### 7.避免使用全局变量，类成员（class member）来传递信息，尽量使用局部变量和参数。
在一个类里面，经常会有传递某些变量的情况。而如果需要传递的变量是某个全局变量或者属性的时候，有些朋友不喜欢将它们作为参数，而是在方法内部就直接访问了：

```
 class A {
   var x;
   func updateX() {
      ...
      x = ...;
   }
   func printX() {
     updateX();
     print(x);
   }
 }
```

我们可以看到，在printX方法里面，updateX和print方法之间并没有值的传递，乍一看我们可能不知道x从哪里来的，导致程序的可读性降低了。

而如果你使用局部变量而不是类成员来传递信息，那么这两个函数就不需要依赖于某一个类，而且更加容易理解，不易出错：

```
func updateX() -> String{
    x = ...;
    return x;
 }
 func printX() {
   String x = updateX();
   print(x);
 }
```

### 注释

-------

优秀的代码大部分是可以自描述的，我们完全可以用程代码本身来表达它到底在干什么，而不需要注释的辅助。

但并不是说一定不能写注释，有以下三种情况比较适合写注释：

公共接口（注释要告诉阅读代码的人，当前类能实现什么功能）。
涉及到比较深层专业知识的代码（注释要体现出实现原理和思想）。
容易产生歧义的代码（但是严格来说，容易让人产生歧义的代码是不允许存在的）。
除了上述这三种情况，如果别人只能依靠注释才能读懂你的代码的时候，就要反思代码出现了什么问题。

最后，对于注释的内容，相对于“做了什么”，更应该说明“为什么这么做”。

### Code Review

-------
换行、注释、方法长度、代码重复等这些是通过机器检查出来的问题，是无需通过人来做的。

而且除了审查需求的实现的程度，bug是否无处藏身以外，更应该关注代码的设计。比如类与类之间的耦合程度，设计的可扩展性，复用性，是否可以将某些方法抽出来作为接口等等。

## 三. iOS规范

### 变量

-------
#### 1. 变量名必须使用驼峰格式

类，协议使用大驼峰：

```
HomePageViewController.h
<HeaderViewDelegate>
```
对象等局部变量使用小驼峰：

```
NSString *personName = @"";
NSUInteger totalCount = 0;
```
#### 2. 变量的名称必须同时包含功能与类型

```
UIButton *addBtn //添加按钮
UILabel *nameLbl //名字标签
NSString *addressStr//地址字符串
```
#### 3. 系统常用类作实例变量声明时加入后缀


| 类型 | 后缀 |
| --- | --- |
| UIViewController | VC |
| UIView | View |
| UILabel | Lbl |
| UIButton | Btn |
| UIImage | Img |
| UIImageView | ImagView |
| NSArray | Array |
| NSMutableArray | Marray |
| NSDictionary | Dict |
| NSMutableDictionary | MDdict |
| NSString | Str |
| NSMutableString | MStr |
| NSSet | Set |
| NSMutableSet | Mset |

##### 常量
1. 常量以相关类名作为前缀


