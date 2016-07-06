# 01-程序启动原理
一.`首先找到程序入口,执行main函数`
```objc
main -> UIApplicationMain
```

二.`UIApplicationMain底层做事情`
```objc
1.创建UIApplication对象

2.创建UIApplication的代理对象,而且给UIApplication对象代理属性赋值

3.开启主运行循环,作用接收事件,让程序一直运行

4.加载info.plist,判断下有木有指定main.storyboard,如果指定就会去加载
```
三.`函数介绍`:
```objc
//输入类名有提示,避免输入错误
NSStringFromClass: 根据一个类名生成一个类名字符串

NSClassFromString: 根据一个类名字符串生成一个类名
```

