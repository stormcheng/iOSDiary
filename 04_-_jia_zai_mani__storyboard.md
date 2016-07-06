# 04 - 加载Mani.storyboard



###一. `什么时候创建window`
1.加载`info.plist`,判断有没有指定main.storyboard,指定了main.storyboard,就会去加载main.storyboard,执行main.storyboard的时候创建.

-----
###二.`加载main.storyboard的步骤`

```
    1. 创建窗口

    2. 加载控制器

    3. 设置窗口的根控制器,显示窗口
```
---
###三.`手动加载,模拟过程`

在加载info.plist文件之后,程序启动才完成。启动完成之后,就要显示窗口,因此在程序启动完成的时候模拟加载过程.

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // 窗口显示的注意点:
    // 1.一定要强引用
    // 2.控件要想显示出来,必须要有尺寸

    // 1.创建窗口
    self.window = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

    // 2.加载main.storyboard来加载stroyboard
    // name:storyboard名称不需要后缀
    UIStoryboard *stroyboard = [UIStoryboard storyboardWithName:@"Main" bundle:nil];

    // 3.加载sotryboard描述的控制器
    //   加载箭头指向的控制器
    ViewController *vc = [stroyboard instantiateInitialViewController];

    // 4.设置窗口的根控制器,底层会自动把根控制器的view添加到窗口上,并且让控制器的view有旋转功能
    self.window.rootViewController = vc;

    // 5.显示窗口
    // makeKeyAndVisible:让窗口成为应用程序的主窗口,并且显示窗口
    [self.window makeKeyAndVisible];

    return YES;
}

```
---
###四.`窗口补充`


> 
1. 状态栏、键盘都是窗口
2. 窗口层级关系

`设置窗口的层级,层级谁大就显示在最外面`
```
UIWindowLevelAlert > UIWindowLevelStatusBar > UIWindowLevelNormal
```

> 3.希望一个控件显示在最前面，直接把它加到`window`上

###五.控制器补充`

####1.`其他创建方法`
```objc
    //1、加载storyboard中指定标识的controller
    ViewController *vc1 = [story instantiateViewControllerWithIdentifier:@"VC"];
    //2、加载Xib中的controller
    ViewController *vc = [[ViewController alloc] initWithNibName:@"VC" bundle:nil];
    //3、直接创建，生成的view是几乎透明的，重写loadView方法创建view
    ViewController *vc = [[ViewController alloc] init];
```
####2.`注意：`

> 2.1 通过xib创建控制器原因:

```
想通过Xib描述控制器的view
```

> 2.2 如何通过xib创建控制器

```objc
 1.设置xib的文件拥有者是控制器,这时候xib就描述这个控制器

 2.连线,告诉控制器是哪个view在描述
```
