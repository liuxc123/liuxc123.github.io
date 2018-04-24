---
title: UIViewController和UIView不同加载方式的生命周期
date: 2018-04-24 16:29:40
category:
- iOS
tags: 
- iOS
---

## 1 基本说明

已经做了几年iOS开发了，对于UIViewController和UIView的生命周期一直不是太重视，导致有很多地方模糊。几天专门写了个Demo来来验证一下。

## 2 加载UIViewController

每一种加载方式所调用的加载方法不同，而且还有一些细节地方不同。苹果为我们提供了四种默认的加载方式，接下来我们看看系统的四种方式，犹豫没有什么理论性的东西，我就直接上代码为主了：

* 通过XIB加载。
* 通过StoryBoard加载。
* 通过NSCoding协议加载。
* 通过代码加载。

### 2.1 用XIB加载UIViewController

首先看初始化代码：

```
- (IBAction)loadControllerFromXIB:(id)sender {
    XIBViewController *xibVC = [[XIBViewController alloc]initWithNibName:@"XIBViewController" bundle:[NSBundle mainBundle]];
    [self.navigationController pushViewController:xibVC animated:YES];
}
```
当我们不实现`loadView`的时候打印结果:

```
2017-04-15 12:05:32.974 UIViewController和UIView生命周期加载和卸载[59883:1192231] initWithNibName
2017-04-15 12:05:32.987 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewDidLoad
2017-04-15 12:05:32.987 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewWillAppear
2017-04-15 12:05:32.996 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewWillLayoutSubviews
2017-04-15 12:05:32.997 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewDidLayoutSubviews
2017-04-15 12:05:33.002 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewWillLayoutSubviews
2017-04-15 12:05:33.002 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewDidLayoutSubviews
2017-04-15 12:05:33.506 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewDidAppear
2017-04-15 12:05:37.142 UIViewController和UIView生命周期加载和卸载[59883:1192231] clickButton
//pop以后
2017-04-15 12:05:42.334 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewWillDisappear
2017-04-15 12:05:42.837 UIViewController和UIView生命周期加载和卸载[59883:1192231] viewDidDisappear
2017-04-15 12:05:42.838 UIViewController和UIView生命周期加载和卸载[59883:1192231] dealloc
```

我发现一个很奇怪的现象。如果用XIB加载的控制器，并且实现了一个空loadView,那么我们在XIB设置的视图都失效了，应该是系统返回了一个默认的视图覆盖了。但是用Storyboard加载的视图，实现一个空的`loadView`则不会丢失Storyboard里面的视图，`这个是用XIB和Storyboard的一个注意点`。

```
-(void)loadView{
    [super loadView];
    NSLog(@"loadView");
}
```

### 2.2 用Storyboard加载UIViewController

初始化代码：

```
- (IBAction)laodControllerFromSB:(id)sender {
    UIStoryboard *sb = [UIStoryboard storyboardWithName:@"Second" bundle:[NSBundle mainBundle]];
    SBViewController *sbVC = [sb instantiateViewControllerWithIdentifier:@"SBViewController"];
    [self.navigationController pushViewController:sbVC animated:YES];
}
```

运行结果：

```
2017-04-15 12:26:45.364 UIViewController和UIView生命周期加载和卸载[59932:1194239] initWithCoder
2017-04-15 12:26:45.365 UIViewController和UIView生命周期加载和卸载[59932:1194239] awakeFromNib
2017-04-15 12:26:45.368 UIViewController和UIView生命周期加载和卸载[59932:1194239] loadView
2017-04-15 12:26:45.368 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLoad
2017-04-15 12:26:45.368 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillAppear
2017-04-15 12:26:45.372 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillLayoutSubviews
2017-04-15 12:26:45.373 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLayoutSubviews
2017-04-15 12:26:45.877 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidAppear
//pop以后
2017-04-15 12:26:50.669 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillDisappear
2017-04-15 12:26:51.172 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidDisappear
2017-04-15 12:26:51.172 UIViewController和UIView生命周期加载和卸载[59932:1194239] dealloc
```

对于Storyboard，使用一个空的`loadView`没有影响。

### 2.3 用NSCodeing加载UIViewController

初始化代码：

```
//这里我就不实现NSCoding协议了，直接传入一个nil参数。
- (IBAction)loadControllerFromCoder:(id)sender {
    CoderViewController *coderVC = [[CoderViewController alloc]initWithCoder:nil];
    [self.navigationController pushViewController:coderVC animated:YES];
}
```

运行结果：

```
2017-04-15 12:30:25.962 UIViewController和UIView生命周期加载和卸载[59932:1194239] initWithCoder
2017-04-15 12:30:25.963 UIViewController和UIView生命周期加载和卸载[59932:1194239] loadView
2017-04-15 12:30:25.963 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLoad
2017-04-15 12:30:25.963 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillAppear
2017-04-15 12:30:25.967 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillLayoutSubviews
2017-04-15 12:30:25.967 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLayoutSubviews
2017-04-15 12:30:25.968 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillLayoutSubviews
2017-04-15 12:30:25.968 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLayoutSubviews
2017-04-15 12:30:26.470 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidAppear
//pop以后
2017-04-15 12:30:28.034 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillDisappear
2017-04-15 12:30:28.537 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidDisappear
2017-04-15 12:30:28.537 UIViewController和UIView生命周期加载和卸载[59932:1194239] dealloc
```

### 2.4 用代码加载UIViewController

初始化代码:

```
- (IBAction)loadControllerWithNone:(id)sender {
    CodeViewController *codeVC = [[CodeViewController alloc]init];
    [self.navigationController pushViewController:codeVC animated:YES];
}
```

运行结果：

```
2017-04-15 12:31:48.785 UIViewController和UIView生命周期加载和卸载[59932:1194239] initWithNibName
2017-04-15 12:31:48.786 UIViewController和UIView生命周期加载和卸载[59932:1194239] init
2017-04-15 12:31:48.787 UIViewController和UIView生命周期加载和卸载[59932:1194239] loadView
2017-04-15 12:31:48.787 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLoad
2017-04-15 12:31:48.788 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillAppear
2017-04-15 12:31:48.792 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillLayoutSubviews
2017-04-15 12:31:48.792 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLayoutSubviews
2017-04-15 12:31:48.792 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillLayoutSubviews
2017-04-15 12:31:48.792 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidLayoutSubviews
2017-04-15 12:31:49.293 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidAppear
//pop以后
2017-04-15 12:31:55.594 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewWillDisappear
2017-04-15 12:31:56.098 UIViewController和UIView生命周期加载和卸载[59932:1194239] viewDidDisappear
2017-04-15 12:31:56.098 UIViewController和UIView生命周期加载和卸载[59932:1194239] dealloc
```

## 3 加载UIView

### 3.1 用XIB加载UIView

初始化代码:

```
- (IBAction)loadViewFromXib:(id)sender {
    XibView *xibView = [[[NSBundle mainBundle]loadNibNamed:@"XIBView" owner:self options:nil] lastObject];
    [self.view addSubview:xibView];
}
```

运行结果：

```
2017-04-15 12:33:22.194 UIViewController和UIView生命周期加载和卸载[59932:1194239] initWithCoder
2017-04-15 12:33:22.195 UIViewController和UIView生命周期加载和卸载[59932:1194239] awakeFromNib
2017-04-15 12:33:22.195 UIViewController和UIView生命周期加载和卸载[59932:1194239] willMoveToWindow
2017-04-15 12:33:22.195 UIViewController和UIView生命周期加载和卸载[59932:1194239] willMoveToSuperview
2017-04-15 12:33:22.196 UIViewController和UIView生命周期加载和卸载[59932:1194239] didMoveToWindow
2017-04-15 12:33:22.196 UIViewController和UIView生命周期加载和卸载[59932:1194239] didMoveToSuperview
2017-04-15 12:33:22.197 UIViewController和UIView生命周期加载和卸载[59932:1194239] layoutSubviews
//这里是点击移除以后
2017-04-15 12:33:25.769 UIViewController和UIView生命周期加载和卸载[59932:1194239] willMoveToSuperview
2017-04-15 12:33:25.770 UIViewController和UIView生命周期加载和卸载[59932:1194239] willMoveToWindow
2017-04-15 12:33:25.771 UIViewController和UIView生命周期加载和卸载[59932:1194239] didMoveToWindow
2017-04-15 12:33:25.771 UIViewController和UIView生命周期加载和卸载[59932:1194239] didMoveToSuperview
2017-04-15 12:33:25.771 UIViewController和UIView生命周期加载和卸载[59932:1194239] dealloc
```

### 3.2 用代码加载UIView

初始化代码：

```
- (IBAction)loadViewWithNone:(id)sender {
    CodeView *codeView = [[CodeView alloc]init];
    codeView.backgroundColor = [UIColor greenColor];
    codeView.frame = CGRectMake(0, 500, 100, 50);
    [self.view addSubview:codeView];
}
```

运行结果：

```
2017-04-15 12:38:57.562 UIViewController和UIView生命周期加载和卸载[60323:1208772] initWithFrame
2017-04-15 12:38:57.562 UIViewController和UIView生命周期加载和卸载[60323:1208772] init
2017-04-15 12:38:57.562 UIViewController和UIView生命周期加载和卸载[60323:1208772] willMoveToWindow
2017-04-15 12:38:57.563 UIViewController和UIView生命周期加载和卸载[60323:1208772] willMoveToSuperview
2017-04-15 12:38:57.563 UIViewController和UIView生命周期加载和卸载[60323:1208772] didMoveToWindow
2017-04-15 12:38:57.563 UIViewController和UIView生命周期加载和卸载[60323:1208772] didMoveToSuperview
2017-04-15 12:38:57.564 UIViewController和UIView生命周期加载和卸载[60323:1208772] layoutSubviews
//点击移除以后
2017-04-15 12:39:02.751 UIViewController和UIView生命周期加载和卸载[60323:1208772] willMoveToSuperview
2017-04-15 12:39:02.752 UIViewController和UIView生命周期加载和卸载[60323:1208772] willMoveToWindow
2017-04-15 12:39:02.752 UIViewController和UIView生命周期加载和卸载[60323:1208772] didMoveToWindow
2017-04-15 12:39:02.752 UIViewController和UIView生命周期加载和卸载[60323:1208772] didMoveToSuperview
2017-04-15 12:39:02.752 UIViewController和UIView生命周期加载和卸载[60323:1208772] 点击移除
2017-04-15 12:39:02.753 UIViewController和UIView生命周期加载和卸载[60323:1208772] dealloc
```

## 4 总结

UIViewController不同加载方式钩子函数总结：

* XIB加载方式
    * initWithNibName
    * loadView(`注意：如果实现一个空的方法，则XIB设置的属性无效，会覆盖XIB中的设置`) 
    * viewDidLoad
    * viewWillAppear
    * loadViewIfNeeded
    * viewWillLayoutSubviews
    * viewDidLayoutSubviews
    * viewDidAppear
* Storyboard加载方式
    * initWithCoder
    * awakeFromNib
    * loadView(实现是一个空方法或者不实现没有影响)
    * viewDidLoad
    * viewWillAppear
    * loadViewIfNeeded
    * viewWillLayoutSubviews
    * viewDidLayoutSubviews
    * viewDidAppear
* NSCoding加载方式
    * initWithCoder
    * loadView
    * viewDidLoad
    * viewWillAppear
    * loadViewIfNeeded
    * viewWillLayoutSubviews
    * viewDidLayoutSubviews
    * viewDidAppear
* 代码加载方式
    * initWithNibName
    * init(`这个是我初始化的时候主动调用,如果用initWithNibName传入nil参数则不会调用`)
    * loadView
    * viewDidLoad
    * viewWillAppear
    * loadViewIfNeeded
    * viewWillLayoutSubviews
    * viewDidLayoutSubviews
    * viewDidAppear
   
我们可以发现，代码加载方式和XIB加载方式一模一样，如果有XIB则加载XIB，如果没有XIB则可以代码添加视图。

UIView不同加载方式钩子函数总结：

* XIB加载方式
    * initWithCoder
    * awakeFromNib
    * willMoveToWindow
    * willMoveToSuperview
    * didMoveToWindow
    * didMoveToSuperview
    * setNeedsLayout
    * layoutSubviews
* 代码加载方式
    * initWithFrame(`设置frame。`)
    * init(`init方法调用`)
    * willMoveToWindow
    * willMoveToSuperview
    * didMoveToWindow
    * didMoveToSuperview
    * setNeedsLayout
    * layoutSubviews

我们发现，如果通过init初始化，然后手动设置Frame。则会导致上面的调用顺序。



