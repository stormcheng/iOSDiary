# 第九章 多线程（3）


####补充

    1-1 关于GCD中的创建和释放
   
    在iOS6.0之前，在GCD中每当使用带creat单词的函数创建对象之后，都应该对其进行一次release操作。
    在iOS6.0之后，GCD被纳入到了ARC的内存管理机制中，在使用GCD的时候我们就像对待普通OC对象一样对待GCD,
    因此不再需要我们调用release方法。
   
    1-2 GCD中设置队列的优先级
        01 使用create函数创建出来的队列不论是串行队列还是并发队列，，其执行任务线程的优先级都是默认优先级。
        02 可以通过set_target_queue来变更队列的优先级。第一个参数传通过creat创建出来的队列，
        后面一个参数传指定了优先级的全局并发队列。第一个参数如果传主队列或者全局并发队列的话，那么执行结果是未知的。

    1-3 暂停和恢复。
        GCD中的队列也是可以暂停和恢复的，直接把相应的队列作为参数就传递就可以。
        使用 dispatch_resume(queue1);和dispatch_suspend(queue1);

    1-4 GCD中可以不使用block而使用函数。

    1-5 在NSOperation中关于main方法的调用问题。
        先调用start方法，在start方法内部会调用main方法。可以通过代码来进行验证。

    参考资料：https://developer.apple.com/library/ios/documentation/Performance/Reference/GCD_libdispatch_Ref/index.html#//apple_ref/c/func/dispatch_queue_create

####1.Runloop基础知识
- 1.1 字面意思

		a 运行循环
		b 跑圈

- 1.2 基本作用（作用重大）

		a 保持程序的持续运行(ios程序为什么能一直活着不会死)
		b 处理app中的各种事件（比如触摸事件、定时器事件【NSTimer】、selector事件【选择器·performSelector···】）
		c 节省CPU资源，提高程序性能，有事情就做事情，没事情就休息

- 1.3 重要说明

        （1）如果没有Runloop,那么程序一启动就会退出，什么事情都做不了。
        （2）如果有了Runloop，那么相当于在内部有一个死循环，能够保证程序的持续运行
        （2）main函数中的Runloop
        		a 在UIApplication函数内部就启动了一个Runloop
        			该函数返回一个int类型的值
        		b 这个默认启动的Runloop是跟主线程相关联的

- 1.4 Runloop对象

        （1）在iOS开发中有两套api来访问Runloop
            a.foundation框架【NSRunloop】
            b.core foundation框架【CFRunloopRef】
        （2）NSRunLoop和CFRunLoopRef都代表着RunLoop对象,它们是等价的，可以互相转换
        （3）NSRunLoop是基于CFRunLoopRef的一层OC包装，所以要了解RunLoop内部结构，需要多研究CFRunLoopRef层面的API（Core Foundation层面）


- 1.5 Runloop参考资料

```objc
（1）苹果官方文档
https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html

（2）CFRunLoopRef开源代码下载地址：
http://opensource.apple.com/source/CF/CF-1151.16/

```

- 1.6 Runloop与线程

		1.Runloop和线程的关系：一个Runloop对应着一条唯一的线程
    		问题：如何让子线程不死
    		回答：给这条子线程开启一个Runloop
		2.Runloop的创建：主线程Runloop已经创建好了，子线程的runloop需要手动创建
		3.Runloop的生命周期：在第一次获取时创建，在线程结束时销毁

- 1.7 获得Runloop对象

```objc
1.获得当前Runloop对象
    //01 NSRunloop
     NSRunLoop * runloop1 = [NSRunLoop currentRunLoop];
    //02 CFRunLoopRef
    CFRunLoopRef runloop2 =   CFRunLoopGetCurrent();

2.拿到当前应用程序的主Runloop（主线程对应的Runloop）
    //01 NSRunloop
     NSRunLoop * runloop1 = [NSRunLoop mainRunLoop];
    //02 CFRunLoopRef
     CFRunLoopRef runloop2 =   CFRunLoopGetMain();

3.注意点：开一个子线程创建runloop,不是通过alloc init方法创建，而是直接通过调用currentRunLoop方法来创建，它本身是一个懒加载的。
4.在子线程中，如果不主动获取Runloop的话，那么子线程内部是不会创建Runloop的。可以下载CFRunloopRef的源码，搜索_CFRunloopGet0,查看代码。
5.Runloop对象是利用字典来进行存储，而且key是对应的线程Value为该线程对应的Runloop。

```
- 1.8 Runloop相关类

（1）Runloop运行原理图

![](images/2.png)

（2）五个相关的类

	a.CFRunloopRef
	b.CFRunloopModeRef【Runloop的运行模式】
	c.CFRunloopSourceRef【Runloop要处理的事件源】
	d.CFRunloopTimerRef【Timer事件】
	e.CFRunloopObserverRef【Runloop的观察者（监听者）】

（3）Runloop和相关类之间的关系图

 ![PNG](images/11.png)

（4）Runloop要想跑起来，它的内部必须要有一个mode,这个mode里面必须有source\observer\timer，至少要有其中的一个。

- CFRunloopModeRef

    	1.CFRunloopModeRef代表着Runloop的运行模式
    	2.一个Runloop中可以有多个mode,一个mode里面又可以有多个source\observer\timer等等
    	3.每次runloop启动的时候，只能指定一个mode,这个mode被称为该Runloop的当前mode
    	4.如果需要切换mode,只能先退出当前Runloop,再重新指定一个mode进入
    	5.这样做主要是为了分割不同组的定时器等，让他们相互之间不受影响
    	6.系统默认注册了5个mode
    	    a.kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
            b.UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
            c.UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用
            d.GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
            e.kCFRunLoopCommonModes: 这是一个占位用的Mode，不是一种真正的Mode


- CFRunloopTimerRef

（1）NSTimer相关代码
```objc
/*
	说明：
	（1）runloop一启动就会选中一种模式，当选中了一种模式之后其它的模式就都不鸟。一个mode里面可以添加多个NSTimer,也就是说以后当创建NSTimer的时候，可以指定它是在什么模式下运行的。
	（2）它是基于时间的触发器，说直白点那就是时间到了我就触发一个事件，触发一个操作。基本上说的就是NSTimer
	（3）相关代码
*/
- (void)timer2
{
    //NSTimer 调用了scheduledTimer方法，那么会自动添加到当前的runloop里面去，而且runloop的运行模式kCFRunLoopDefaultMode

    NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];

    //更改模式
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

}

- (void)timer1
{
    //    [NSTimer scheduledTimerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];

    NSTimer *timer = [NSTimer timerWithTimeInterval:2.0 target:self selector:@selector(run) userInfo:nil repeats:YES];

    //定时器添加到UITrackingRunLoopMode模式，一旦runloop切换模式，那么定时器就不工作
    //    [[NSRunLoop currentRunLoop] addTimer:timer forMode:UITrackingRunLoopMode];

    //定时器添加到NSDefaultRunLoopMode模式，一旦runloop切换模式，那么定时器就不工作
    //    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];

    //占位模式：common modes标记
    //被标记为common modes的模式 kCFRunLoopDefaultMode  UITrackingRunLoopMode
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

    //    NSLog(@"%@",[NSRunLoop currentRunLoop]);
}

- (void)run
{
    NSLog(@"---run---%@",[NSRunLoop currentRunLoop].currentMode);
}

- (IBAction)btnClick {

    NSLog(@"---btnClick---");
}

```

（2）GCD中的定时器
```objc
//0.创建一个队列
    dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

    //1.创建一个GCD的定时器
    /*
     第一个参数：说明这是一个定时器
     第四个参数：GCD的回调任务添加到那个队列中执行，如果是主队列则在主线程执行
     */
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);

    //2.设置定时器的开始时间，间隔时间以及精准度

    //设置开始时间，三秒钟之后调用
    dispatch_time_t start = dispatch_time(DISPATCH_TIME_NOW,3.0 *NSEC_PER_SEC);
    //设置定时器工作的间隔时间
    uint64_t intevel = 1.0 * NSEC_PER_SEC;

    /*
     第一个参数：要给哪个定时器设置
     第二个参数：定时器的开始时间DISPATCH_TIME_NOW表示从当前开始
     第三个参数：定时器调用方法的间隔时间
     第四个参数：定时器的精准度，如果传0则表示采用最精准的方式计算，如果传大于0的数值，则表示该定时切换i可以接收该值范围内的误差，通常传0
     该参数的意义：可以适当的提高程序的性能
     注意点：GCD定时器中的时间以纳秒为单位（面试）
     */

    dispatch_source_set_timer(timer, start, intevel, 0 * NSEC_PER_SEC);

    //3.设置定时器开启后回调的方法
    /*
     第一个参数：要给哪个定时器设置
     第二个参数：回调block
     */
    dispatch_source_set_event_handler(timer, ^{
        NSLog(@"------%@",[NSThread currentThread]);
    });

    //4.执行定时器
    dispatch_resume(timer);

    //注意：dispatch_source_t本质上是OC类，在这里是个局部变量，需要强引用
    self.timer = timer;

```

- CFRunloopSourceRef

    	1.是事件源也就是输入源，有两种分类模式；
    	  一种是按照苹果官方文档进行划分的
    	  另一种是基于函数的调用栈来进行划分的（source0和source1）。
        2.具体的分类情况
            （1）以前的分法
                Port-Based Sources
                Custom Input Sources
                Cocoa Perform Selector Sources

            （2）现在的分法
                Source0：非基于Port的
                Source1：基于Port的
        3.可以通过打断点的方式查看一个方法的函数调用栈

- CFRunLoopObserverRef

（1）CFRunLoopObserverRef是观察者，能够监听RunLoop的状态改变

（2）如何监听
```objc
 //创建一个runloop监听者
    CFRunLoopObserverRef observer = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(),kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {

        NSLog(@"监听runloop状态改变---%zd",activity);
    });

    //为runloop添加一个监听者
    CFRunLoopAddObserver(CFRunLoopGetCurrent(), observer, kCFRunLoopDefaultMode);

    CFRelease(observer);
```
（3）监听的状态
```objc
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),   //即将进入Runloop
    kCFRunLoopBeforeTimers = (1UL << 1),    //即将处理NSTimer
    kCFRunLoopBeforeSources = (1UL << 2),   //即将处理Sources
    kCFRunLoopBeforeWaiting = (1UL << 5),   //即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),    //刚从休眠中唤醒
    kCFRunLoopExit = (1UL << 7),            //即将退出runloop
    kCFRunLoopAllActivities = 0x0FFFFFFFU   //所有状态改变
};
```

- 1.9 Runloop运行逻辑
-
![](images/3.png)
--------------------
![](images/4.png)

####2.Runloop应用

    NSTimer
    ImageView显示
    PerformSelector
    常驻线程
    自动释放池



###3.网络基础
- 3.1 网络基础

		001 问题：为什么要学习网络编程？
		    回答：（1）网络编程是一种实时更新应用数据的常用手段
                 （2）网络编程是开发优秀网络应用的前提和基础

		002 网络基本概念
			2-1 客户端（就是手机或者ipad等手持设备上面的APP）
			2-2 服务器（远程服务器-本地服务器）
			2-3 请求（客户端索要数据的方式）
			2-4 响应（需要客户端解析数据）
			2-5 数据库（服务器的数据从哪里来）

- 3.2 Http

		001 URL
			1-1 如何找到服务器（通过一个唯一的URL）
			1-2 URL介绍
				a. 统一资源定位符
				b. url格式（协议\主机地址\路径）
				    协议：不同的协议，代表着不同的资源查找方式、资源传输方式
                    主机地址：存放资源的主机（服务器）的IP地址（域名）
                    路径：资源在主机（服务器）中的具体位置

			1-3 请求协议
				【file】访问的是本地计算机上的资源，格式是file://（不用加主机地址）
				【ftp】访问的是共享主机的文件资源，格式是ftp://
				【mailto】访问的是电子邮件地址，格式是mailto:
				【http】超文本传输协议，访问的是远程的网络资源，格式是http://（网络请求中最常用的协议）

		002 http协议
			2-1 http协议简单介绍
			    a.超文本传输协议
			    b.规定客户端和服务器之间的数据传输格式
                c.让客户端和服务器能有效地进行数据沟通

			2-2 http协议优缺点
				a.简单快速（协议简单，服务器端程序规模小，通信速度快）
				b.灵活（允许传输各种数据）
				c.非持续性连接(1.1之前版本是非持续的，即限制每次连接只处理一个请求，服务器对客户端的请求做出响应后，马上断开连接，这种方式可以节省传输时间)
			2-3 基本通信过程
                a.请求：客户端向服务器索要数据
                b.响应：服务器返回客户端相应的数据

		003 GET和POST请求
			3-1 http里面发送请求的方法
			GET（常用）、POST（常用）、OPTIONS、HEAD、PUT、DELETE、TRACE、CONNECT、PATCH

			3-2 GET和POST请求的对比【区别在于参数如何传递】
                GET
                在请求URL后面以?的形式跟上发给服务器的参数，多个参数之间用&隔开，比如
                http://ww.test.com/login?username=123&pwd=234&type=JSON
                由于浏览器和服务器对URL长度有限制，因此在URL后面附带的参数是有限制的，通常不能超过1KB

                POST
                发给服务器的参数全部放在请求体中
                理论上，POST传递的数据量没有限制（具体还得看服务器的处理能力）

			3-3 如何选择【除简单数据查询外，其它的一律使用POST请求】
    			a.如果要传递大量数据，比如文件上传，只能用POST请求
                b.GET的安全性比POST要差些，如果包含机密\敏感信息，建议用POST
                c.如果仅仅是索取数据（数据查询），建议使用GET
                d.如果是增加、修改、删除数据，建议使用POST


		004 iOS中发送http请求的方案
			4-1 苹果原生
				NSURLConnection 03年推出的古老技术
				NSURLSession 	13年推出iOS7之后，以取代NSURLConnection【重点】
				CFNetwork		底层技术、C语言的

			4-2 第三方框架
				ASIHttpRequest
				AFNetworking 		【重点】
				MKNetworkKit

		005 http请求通信过程
			5-1 请求
				【包括请求头+请求体·非必选】
			5-2 响应
				【响应头+响应体】
			5-3 通信过程
				a.发送请求的时候把请求头和请求体（请求体是非必须的）包装成一个请求对象
				b.服务器端对请求进行响应，在响应信息中包含响应头和响应体，响应信息是对服务器端的描述，具体的信息放在响应体中传递给客户端
			5-4 状态码
				【200】：请求成功
				【400】：客户端请求的语法错误，服务器无法解析
				【404】：无法找到资源
				【500】：服务器内部错误，无法完成请求

####4.NSURLConnection使用
- 4.1 NSURLConnection同步请求（GET）

（1）步骤

        01 设置请求路径
        02 创建请求对象（默认是GET请求，且已经默认包含了请求头）
        03 使用NSURLSession sendsync方法发送网络请求
        04 接收到服务器的响应后，解析响应体

（2）相关代码
```objc
//1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it&type=XML"];
//    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/video?type=XML"];

    //2.创建一个请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //3.把请求发送给服务器
    //sendSynchronousRequest  阻塞式的方法，会卡住线程

    NSHTTPURLResponse *response = nil;
    NSError *error = nil;

    /*
     第一个参数：请求对象
     第二个参数：响应头信息，当该方法执行完毕之后，该参数被赋值
     第三个参数：错误信息，如果请求失败，则error有值
     */
     //该方法是阻塞式的，会卡住线程
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];

    //4.解析服务器返回的数据
    NSString *str = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];

```
- 4.2 NSURLConnection异步请求（GET-SendAsync）

（1）相关说明

    01 该方法不会卡住当前线程，网络请求任务是异步执行的

（2）相关代码
```objc
//1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=520it&pwd=520it"];

    //2.创建一个请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //3.把请求发送给服务器,发送一个异步请求
    /*
     第一个参数：请求对象
     第二个参数：回调方法在哪个线程中执行，如果是主队列则block在主线程中执行，非主队列则在子线程中执行
     第三个参数：completionHandlerBlock块：接受到响应的时候执行该block中的代码
        response：响应头信息
        data：响应体
        connectionError：错误信息，如果请求失败，那么该参数有值
     */
    [NSURLConnection sendAsynchronousRequest:request queue:[[NSOperationQueue alloc]init] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {

        //4.解析服务器返回的数据
        NSString *str = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        //转换并打印响应头信息
        NSHTTPURLResponse *r = (NSHTTPURLResponse *)response;
        NSLog(@"--%zd---%@--",r.statusCode,r.allHeaderFields);
    }];

```

- 4.3 NSURLConnection异步请求（GET-代理）

（1）步骤

    01 确定请求路径
    02 创建请求对象
    03 创建NSURLConnection对象并设置代理
    04 遵守NSURLConnectionDataDelegate协议，并实现相应的代理方法
    05 在代理方法中监听网络请求的响应

（2）设置代理的几种方法

```objc
/*
     设置代理的第一种方式：自动发送网络请求
     [[NSURLConnection alloc]initWithRequest:request delegate:self];
     */

    /*
     设置代理的第二种方式：
     第一个参数：请求对象
     第二个参数：谁成为NSURLConnetion对象的代理
     第三个参数：是否马上发送网络请求，如果该值为YES则立刻发送，如果为NO则不会发送网路请求
     NSURLConnection *conn = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];

     //调用该方法控制网络请求的发送
     [conn start];
     */

    //设置代理的第三种方式：使用类方法设置代理，会自动发送网络请求
    NSURLConnection *conn = [NSURLConnection connectionWithRequest:request delegate:self];
    //取消网络请求
    //[conn cancel];

```
（3）相关的代理方法

```objc
/*
 1.当接收到服务器响应的时候调用
 第一个参数connection：监听的是哪个NSURLConnection对象
 第二个参数response：接收到的服务器返回的响应头信息
 */
- (void)connection:(nonnull NSURLConnection *)connection didReceiveResponse:(nonnull NSURLResponse *)response

/*
 2.当接收到数据的时候调用，该方法会被调用多次
 第一个参数connection：监听的是哪个NSURLConnection对象
 第二个参数data：本次接收到的服务端返回的二进制数据（可能是片段）
 */
- (void)connection:(nonnull NSURLConnection *)connection didReceiveData:(nonnull NSData *)data
/*

 3.当服务端返回的数据接收完毕之后会调用
 通常在该方法中解析服务器返回的数据
 */
-(void)connectionDidFinishLoading:(nonnull NSURLConnection *)connection

/*4.当请求错误的时候调用（比如请求超时）
 第一个参数connection：NSURLConnection对象
 第二个参数：网络请求的错误信息，如果请求失败，则error有值
 */
- (void)connection:(nonnull NSURLConnection *)connection didFailWithError:(nonnull NSError *)error
```
（4）其它知识点

```objc
    01 关于消息弹窗第三方框架的使用
        SVProgressHUD
    02 字符串截取相关方法
    - (NSRange)rangeOfString:(NSString *)searchString;
    - (NSString *)substringWithRange:(NSRange)range;
```

- 4.4 NSURLConnection发送POST请求

（1）发送POST请求步骤

	a.确定URL路径
	b.创建请求对象（可变对象）
	c.修改请求对象的方法为POST，设置请求体（Data）
	d.发送一个异步请求
	e.补充：设置请求超时，处理错误信息，设置请求头（如获取客户端的版本等等,请求头是可设置可不设置的）

（2）相关代码
```objc
 //1.确定请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];

    //2.创建请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

    //2.1更改请求方法
    request.HTTPMethod = @"POST";

    //2.2设置请求体
    request.HTTPBody = [@"username=520it&pwd=520it" dataUsingEncoding:NSUTF8StringEncoding];

    //2.3请求超时
    request.timeoutInterval = 5;

    //2.4设置请求头
    [request setValue:@"ios 9.0" forHTTPHeaderField:@"User-Agent"];


    //3.发送请求
    [NSURLConnection sendAsynchronousRequest:request queue:[NSOperationQueue mainQueue] completionHandler:^(NSURLResponse * __nullable response, NSData * __nullable data, NSError * __nullable connectionError) {

        //4.解析服务器返回的数据
        if (connectionError) {
            NSLog(@"--请求失败-");
        }else
        {
            NSLog(@"%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding]);
        }

    }];
```

- 4.5 URL中文转码问题
```objc
   //1.确定请求路径

    NSString *urlStr = @"http://120.25.226.186:32812/login2?username=小码哥&pwd=520it";
    NSLog(@"%@",urlStr);
    //中文转码操作
    urlStr =  [urlStr stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
    NSLog(@"%@",urlStr);

    NSURL *url = [NSURL URLWithString:urlStr];
```
