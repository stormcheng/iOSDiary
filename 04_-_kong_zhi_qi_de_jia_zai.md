# 05 - 控制器View加载过程

####一.`loadView`

什么时候调用:`当第一次使用控制器的view的时候就会调用`

作用:`加载控制器的view,自定义控制器的view`

`注意:`
```
1.只要重写loadView,必须自己手动创建控制器的view

2.在没给_view赋值之前,不能调用self.view会死循环

3.窗口根控制器的view可以不设置尺寸

4.不要在loadView中写[super loadView]
底层首先会判断下有没有指定storyboard或者xib,如果指定,
就会加载它们描述的控制器的view,如果没有指定,创建一个空的view.

```

####二.`loadView加载流程`
控制器View的决定权:*重写`LoadView`>`storyboard`>`nibName`>`xib`*

![](控制器view加载流程.png)

#### 三.xib加载控制器的view

`init底层会调用initWithNibName:bundle:`

```objc
 通过xib创建XMGViewController控制器的view
    1.先看nibName有没有值，有值就加载nibName对应的view
    2.如果1不符合，就找有没CHBView.xib，有就加载对应的view
    3.如果2不符合，就找有没CHBViewController.xib有就加载对应view
    4.如果3不符合，就创建一个空的view
```
```objc
#pragma mark - init的底层实现
- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    return [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
}

#pragma mark - view的底层实现
- (UIView *)view
{
    if (_view == nil) {
        [self loadView];
        [self viewDidLoad];
   }
    return _view;
}
```
####四.控制器的view延迟加载

    只有在窗口显示的时候,才会调用loadView 方法

####五.控制器view默认的是几乎透明的


