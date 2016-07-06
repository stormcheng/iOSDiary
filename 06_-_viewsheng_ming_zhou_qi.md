# 06 - View生命周期

控制器view的生命周期方法 
> ARC:
viewDidLoad -> viewWillAppear -> viewWillLayoutSubviews -> viewDidLayoutSubviews -> viewDidAppear -> viewWillDisappear -> viewDidDisappear

```objc
// 非ARC:
// 控制器的view即将销毁
- (void)viewWillUnload
{
}
// 控制器的view即将销毁
- (void)viewDidUnload
{
    // 清空没有必要的数据
    self.datas = nil;
}
```
![](images/内存警告处理.png)
-----
```objc
// 什么是控制器view的生命周期方法,一般以view开头的方法,都是view的生命周期

// 控制器的view即将显示的时候调用
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
}
// 控制器的view完全显示的时候调用
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
}
// 控制器的view即将消失的时候调用
- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
}

// 控制器的view完全消失的时候调用
- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
}

// 控制器的view即将布局子控件的时候调用
- (void)viewWillLayoutSubviews
{
    [super viewWillLayoutSubviews];
}

// 控制器的view布局子控件完成的时候调用
- (void)viewDidLayoutSubviews
{
    [super viewDidLayoutSubviews];
}

// 控制器的view加载完成的时候调用
- (void)viewDidLoad {
    [super viewDidLoad];
}

//- (void)setDatas:(NSArray *)datas
//{
//    if (datas != _datas) {
//        [_datas release];
//        _datas = [datas retain];
//    }
//    return _datas;
//}

// 当前控制器接收到内存警告的时候调用
- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // 清空一些缓存,一般清空图片,一些没有用的数据
}
```
![](生命周期方法.png)
