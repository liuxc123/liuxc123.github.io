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
2018-04-24 17:26:08.799985+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToSuperview
2018-04-24 17:26:08.800237+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToWindow
2018-04-24 17:26:08.800931+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToWindow
2018-04-24 17:26:08.801165+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToSuperview
2018-04-24 17:26:08.801499+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
2018-04-24 17:30:25.472638+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithNibName
2018-04-24 17:30:25.474095+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadView
2018-04-24 17:30:25.474368+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLoad
2018-04-24 17:30:25.474598+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillAppear
2018-04-24 17:30:25.474752+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadViewIfNeeded
2018-04-24 17:30:25.484713+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillLayoutSubviews
2018-04-24 17:30:25.484896+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLayoutSubviews
2018-04-24 17:30:25.987213+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidAppear
2018-04-24 17:30:25.987213+0800 UIViewController和UIView生命周期加载和卸载[59883:1192231] clickButton
//pop以后
2018-04-24 17:31:24.384741+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillDisappear
2018-04-24 17:31:24.888131+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidDisappear
2018-04-24 17:31:24.888403+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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
2018-04-24 17:31:49.174073+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithCoder
2018-04-24 17:31:49.174343+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] awakeFromNib
2018-04-24 17:31:49.178285+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadView
2018-04-24 17:31:49.178582+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLoad
2018-04-24 17:31:49.178791+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillAppear
2018-04-24 17:31:49.178980+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadViewIfNeeded
2018-04-24 17:31:49.188183+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillLayoutSubviews
2018-04-24 17:31:49.188405+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLayoutSubviews
2018-04-24 17:31:49.691875+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidAppear
//pop以后
2018-04-24 17:32:04.990075+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillDisappear
2018-04-24 17:32:05.493917+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidDisappear
2018-04-24 17:32:05.494169+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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
2018-04-24 17:32:24.710172+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithCoder
2018-04-24 17:32:24.711632+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadView
2018-04-24 17:32:24.711926+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLoad
2018-04-24 17:32:24.712247+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillAppear
2018-04-24 17:32:24.712434+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadViewIfNeeded
2018-04-24 17:32:24.722093+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillLayoutSubviews
2018-04-24 17:32:24.722220+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLayoutSubviews
2018-04-24 17:32:25.223966+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidAppear
//pop以后
2018-04-24 17:32:39.395215+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillDisappear
2018-04-24 17:32:39.898003+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidDisappear
2018-04-24 17:32:39.898269+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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
2018-04-24 17:32:56.444591+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithNibName
2018-04-24 17:32:56.446164+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadView
2018-04-24 17:32:56.446451+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLoad
2018-04-24 17:32:56.446689+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillAppear
2018-04-24 17:32:56.446863+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] loadViewIfNeeded
2018-04-24 17:32:56.456733+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillLayoutSubviews
2018-04-24 17:32:56.456898+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidLayoutSubviews
2018-04-24 17:32:56.958470+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidAppear
//pop以后
2018-04-24 17:33:06.696999+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewWillDisappear
2018-04-24 17:33:07.200255+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] viewDidDisappear
2018-04-24 17:33:07.200511+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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
2018-04-24 17:33:20.803998+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithCoder
2018-04-24 17:33:20.804249+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] awakeFromNib
2018-04-24 17:33:20.804455+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToWindow
2018-04-24 17:33:20.804610+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToSuperview
2018-04-24 17:33:20.804996+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToWindow
2018-04-24 17:33:20.805238+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToSuperview
2018-04-24 17:33:20.805362+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] setNeedsLayout
2018-04-24 17:33:20.805966+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] layoutSubviews
//这里是点击移除以后
2018-04-24 17:33:32.628030+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToSuperview
2018-04-24 17:33:32.628204+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToWindow
2018-04-24 17:33:32.628591+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToWindow
2018-04-24 17:33:32.628709+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToSuperview
2018-04-24 17:33:32.628892+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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
2018-04-24 17:33:47.292690+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] initWithFrame
2018-04-24 17:33:47.292908+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] init
2018-04-24 17:33:47.293115+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToWindow
2018-04-24 17:33:47.293290+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToSuperview
2018-04-24 17:33:47.293542+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToWindow
2018-04-24 17:33:47.293773+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToSuperview
2018-04-24 17:33:47.294269+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] layoutSubviews
//点击移除以后
2018-04-24 17:34:03.861096+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToSuperview
2018-04-24 17:34:03.861271+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] willMoveToWindow
2018-04-24 17:34:03.861589+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToWindow
2018-04-24 17:34:03.861691+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] didMoveToSuperview
2018-04-24 17:34:03.861847+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] 点击移除
2018-04-24 17:34:03.862060+0800 UIViewController和UIView生命周期加载和卸载[42121:1546481] dealloc
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



