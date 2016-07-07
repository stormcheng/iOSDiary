# 第六章 网络（1）


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
