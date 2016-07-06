# 03 - UIApplicationDelegate
```objc
// AppDelegate:监听应用程序的生命周期
// 以下方法就是应用程序的生命周期方法

// 应用程序启动完成的时候就会调用AppDelegate的方法
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    return YES;
}

// 当应用程序失去焦点的时候调用
- (void)applicationWillResignActive:(UIApplication *)application {
}

// 当应用程序进入后台的时候调用
- (void)applicationDidEnterBackground:(UIApplication *)application {
}

// 当应用程序进入前台的时候调用
- (void)applicationWillEnterForeground:(UIApplication *)application {
}

// 当应用程序完全获取焦点的时候调用
// 只有当应用程序完全获取焦点的时候,才能够与用户交互
- (void)applicationDidBecomeActive:(UIApplication *)application {
}

// 当应用程序关闭的时候
- (void)applicationWillTerminate:(UIApplication *)application {
}
```
