# 第六章 网络（2）

###1.AFN使用技巧

```objc
1.在开发的时候可以创建一个工具类，继承自我们的AFN中的请求管理者，再控制器中真正发请求的代码使用自己封装的工具类。
2.这样做的优点是以后如果修改了底层依赖的框架，那么我们修改这个工具类就可以了，而不用再一个一个的去修改。
3.该工具类一般提供一个单例方法，在该方法中会设置一个基本的请求路径。
4.该方法通常还会提供对GET或POST请求的封装。
5.在外面的时候通过该工具类来发送请求
6.单例方法：
+ (instancetype)shareNetworkTools
{
    static XMGNetworkTools *instance;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        // 注意: BaseURL中一定要以/结尾
        instance = [[self alloc] initWithBaseURL:[NSURL URLWithString:@"http://120.25.226.186:32812/"]];
    });
    return instance;
}
```

###2.AFN文件上传
```objc
1.文件上传拼接数据的第一种方式
[formData appendPartWithFileData:data name:@"file" fileName:@"xxoo.png" mimeType:@"application/octet-stream"];
2.文件上传拼接数据的第二种方式
 [formData appendPartWithFileURL:fileUrl name:@"file" fileName:@"xx.png" mimeType:@"application/octet-stream" error:nil];
3.文件上传拼接数据的第三种方式
 [formData appendPartWithFileURL:fileUrl name:@"file" error:nil];
4.【注】在资料中已经提供了一个用于文件上传的分类。

/*文件上传相关的代码如下*/
-(void)upload
{
    //1.创建一个请求管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    //2.发送POST请求上传数据
    /*
     第一个参数：请求路径：NSString类型
     第二个参数：要上传的非文件参数
     第三个参数：block回调
        在该回调中，需要利用formData拼接即将上传的二进制数据
     第三个参数：上传成功的block回调
        task：dataTask(任务)
        responseObject:服务器返回的数据
     第四个参数：上传失败的block回调
        error：错误信息，如果上传文件失败，那么error里面包含了错误的描述信息
     */

    NSDictionary *dict = @{
                           @"username":@"wenidngding"
                           };

    [manager POST:@"http://120.25.226.186:32812/upload" parameters:dict constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {

        //把本地的图片转换为NSData类型的数据
        UIImage *image = [UIImage imageNamed:@"123"];
        NSData *data = UIImagePNGRepresentation(image);

        /*
         //拼接二进制文件数据
         第一个参数：要上传的文件的二进制数据
         第二个参数：服务器接口规定的名称
         第三个参数：这个参数上传到服务器之后用什么名字来进行保存
         第四个参数：上传文件的MIMEType类型
         */
        [formData appendPartWithFileData:data name:@"file" fileName:@"xxoo.png" mimeType:@"application/octet-stream"];

    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {
        NSLog(@"请求成功---%@",responseObject);

    } failure:^(NSURLSessionDataTask * _Nonnull task, NSError * _Nonnull error) {
        NSLog(@"请求失败--%@",error);
    }];
}

-(void)upload2
{
    NSLog(@"%s",__func__);

    //1.创建一个请求管理者
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    //2.发送POST请求上传数据
    /*
     第一个参数：请求路径：NSString类型
     第二个参数：要上传的非文件参数
     第三个参数：block回调
     在该回调中，需要利用formData拼接即将上传的二进制数据
     第三个参数：上传成功的block回调
     task：dataTask(任务)
     responseObject:服务器返回的数据
     第四个参数：上传失败的block回调
     error：错误信息，如果上传文件失败，那么error里面包含了错误的描述信息
     */

    NSDictionary *dict = @{
                           @"username":@"wenidngding"
                           };

    [manager POST:@"http://120.25.226.186:32812/upload" parameters:dict constructingBodyWithBlock:^(id<AFMultipartFormData>  _Nonnull formData) {

        //本地文件的url
        NSURL *fileUrl = [NSURL fileURLWithPath:@"/Users/文顶顶/Desktop/KF[WTI`AQ3T`A@3R(B96D89.gif"];
        /*
         //拼接二进制文件数据
         第一个参数：要上传文件的url路径
         第二个参数:服务器要求的参数名称
         第三个参数：这个文件上传到服务器之后叫什么名称
         第四个参数：文件的mimetype类型
         第五个参数：错误信息
         */
//        [formData appendPartWithFileURL:fileUrl name:@"file" fileName:@"xx.png" mimeType:@"application/octet-stream" error:nil];

        //另外一种上传文件的方式
        /*
         说明：该方法和上面的方法等价，不过该方法更加简单其内部会自动的的根据url路径确定文件保存名称，并通过内部方法获取上传文件的mimetype类型
         */
        [formData appendPartWithFileURL:fileUrl name:@"file" error:nil];


    } success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {
        NSLog(@"请求成功---%@",responseObject);

    } failure:^(NSURLSessionDataTask * _Nonnull task, NSError * _Nonnull error) {
        NSLog(@"请求失败--%@",error);
    }];
}
```

###3.使用AFN进行序列化处理
```objc
/*
1.AFN它内部默认把服务器响应的数据当做json来进行解析，所以如果服务器返回给我的不是JSON数据那么请求报错，这个时候需要设置AFN对响应信息的解析方式。AFN提供了三种解析响应信息的方式，分别是：
1）AFXMLParserResponseSerializer----XML
2) AFHTTPResponseSerializer---------默认二进制响应数据
3）AFJSONResponseSerializer---------JSON

2.还有一种情况就是服务器返回给我们的数据格式不太一致（开发者工具Content-Type:text/xml）,那么这种情况也有可能请求不成功。解决方法:
1） 直接在源代码中修改，添加相应的Content-Type
2） 拿到这个属性，添加到它的集合中

3.相关代码
-(void)srializer
{
    //1.创建请求管理者，内部基于NSURLSession
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    /* 知识点1：设置AFN采用什么样的方式来解析服务器返回的数据*/

    //如果返回的是XML，那么告诉AFN，响应的时候使用XML的方式解析
    manager.responseSerializer = [AFXMLParserResponseSerializer serializer];

    //如果返回的就是二进制数据，那么采用默认二进制的方式来解析数据
    //manager.responseSerializer = [AFHTTPResponseSerializer serializer];

    //采用JSON的方式来解析数据
    //manager.responseSerializer = [AFJSONResponseSerializer serializer];


    /*知识点2： 告诉AFN服务器返回的数据内容是什么类型的 Content-Type*/
    manager.responseSerializer.acceptableContentTypes = [NSSet setWithObject:@"text/xml"];

    //2.把所有的请求参数通过字典的方式来装载，GET方法内部会自动把所有的键值对取出以&符号拼接并最后用？符号连接在请求路径后面
    NSDictionary *dict = @{
                           @"username":@"223",
                           @"pwd":@"ewr",
                           @"type":@"XML"
                           };

    //3.发送GET请求
    [manager GET:@"http://120.25.226.186:32812/login" parameters:dict success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {

        //4.请求成功的回调block
        NSLog(@"%@",[responseObject class]);
    } failure:^(NSURLSessionDataTask * _Nonnull task, NSError * _Nonnull error) {

        //5.请求失败的回调，可以打印error的值查看错误信息
        NSLog(@"%@",error);
    }];
}
```

###4.使用AFN来检测网络状态

```objc
/*
说明：可以使用AFN框架中的AFNetworkReachabilityManager来监听网络状态的改变，也可以利用苹果提供的Reachability来监听。建议在开发中直接使用AFN框架处理。
 */
//使用AFN框架来检测网络状态的改变
-(void)AFNReachability
{
    //1.创建网络监听管理者
    AFNetworkReachabilityManager *manager = [AFNetworkReachabilityManager sharedManager];

    //2.监听网络状态的改变
    /*
     AFNetworkReachabilityStatusUnknown          = 未知
     AFNetworkReachabilityStatusNotReachable     = 没有网络
     AFNetworkReachabilityStatusReachableViaWWAN = 3G
     AFNetworkReachabilityStatusReachableViaWiFi = WIFI
     */
    [manager setReachabilityStatusChangeBlock:^(AFNetworkReachabilityStatus status) {
        switch (status) {
            case AFNetworkReachabilityStatusUnknown:
                NSLog(@"未知");
                break;
            case AFNetworkReachabilityStatusNotReachable:
                NSLog(@"没有网络");
                break;
            case AFNetworkReachabilityStatusReachableViaWWAN:
                NSLog(@"3G");
                break;
            case AFNetworkReachabilityStatusReachableViaWiFi:
                NSLog(@"WIFI");
                break;

            default:
                break;
        }
    }];

    //3.开始监听
    [manager startMonitoring];
}

------------------------------------------------------------
//使用苹果提供的Reachability来检测网络状态，如果要持续监听网络状态的概念，需要结合通知一起使用。
//提供下载地址：https://developer.apple.com/library/ios/samplecode/Reachability/Reachability.zip

-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event
{
    //1.注册一个通知
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(networkChange) name:kReachabilityChangedNotification object:nil];

    //2.拿到一个对象，然后调用开始监听方法
    Reachability *r = [Reachability reachabilityForInternetConnection];
    [r startNotifier];

    //持有该对象，不要让该对象释放掉
    self.r = r;
}

//当控制器释放的时候，移除通知的监听
-(void)dealloc
{
    [[NSNotificationCenter defaultCenter] removeObserver:self];
}

-(void)networkChange
{
    //获取当前网络的状态
    if([Reachability reachabilityForLocalWiFi].currentReachabilityStatus != NotReachable)
    {
        NSLog(@"当前网络为WIFI");
    }else if ([Reachability reachabilityForInternetConnection].currentReachabilityStatus != NotReachable)
    {
        NSLog(@"当前网络为手机自带网络");
    }else
    {
        NSLog(@"当前没有网络");
    }
}
```

###5.数据安全
```objc
01 攻城利器：Charles（公司中一般都使用该工具来抓包，并做网络测试）
注意：Charles在使用中的乱码问题，可以显示包内容，然后打开info.plist文件，找到java目录下面的VMOptions，在后面添加一项：-Dfile.encoding=UTF-8
02 MD5消息摘要算法是不可逆的。
03 数据加密的方式和规范一般公司会有具体的规定，不必多花时间。
```
###6.HTTPS的基本使用
```objc
1.https简单说明
    HTTPS（全称：Hyper Text Transfer Protocol over Secure Socket Layer），是以安全为目标的HTTP通道，简单讲是HTTP的安全版。
    即HTTP下加入SSL层，HTTPS的安全基础是SSL，因此加密的详细内容就需要SSL。 它是一个URI scheme（抽象标识符体系），句法类同http:体系。用于安全的HTTP数据传输。
    https:URL表明它使用了HTTP，但HTTPS存在不同于HTTP的默认端口及一个加密/身份验证层（在HTTP与TCP之间）。

2.HTTPS和HTTP的区别主要为以下四点：
        一、https协议需要到ca申请证书，一般免费证书很少，需要交费。
        二、http是超文本传输协议，信息是明文传输，https 则是具有安全性的ssl加密传输协议。
        三、http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
        四、http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

3.对开发的影响。
3.1 如果是自己使用NSURLSession来封装网络请求，涉及代码如下。
- (void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event
{
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];

    NSURLSessionDataTask *task =  [session dataTaskWithURL:[NSURL URLWithString:@"https://www.apple.com"] completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
        NSLog(@"%@", [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding]);
    }];
    [task resume];
}

/*
 只要请求的地址是HTTPS的, 就会调用这个代理方法
 我们需要在该方法中告诉系统, 是否信任服务器返回的证书
 Challenge: 挑战 质问 (包含了受保护的区域)
 protectionSpace : 受保护区域
 NSURLAuthenticationMethodServerTrust : 证书的类型是 服务器信任
 了解：证书有好几种，一种是被认证过的一种是没有被认证过的，如果证书是被认证过的，那么只需要信任一次就可以了，如果是没有被认证过的，那么每次都需要重新认证
 */
- (void)URLSession:(NSURLSession *)session didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition, NSURLCredential *))completionHandler
{
    //    NSLog(@"didReceiveChallenge %@", challenge.protectionSpace);
    NSLog(@"调用了最外层");
    // 1.判断服务器返回的证书类型, 是否是服务器信任
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        NSLog(@"调用了里面这一层是服务器信任的证书");
        /*
         NSURLSessionAuthChallengeUseCredential = 0,                     使用证书
         NSURLSessionAuthChallengePerformDefaultHandling = 1,            忽略证书(默认的处理方式)
         NSURLSessionAuthChallengeCancelAuthenticationChallenge = 2,     忽略书证, 并取消这次请求
         NSURLSessionAuthChallengeRejectProtectionSpace = 3,            忽略当前这一次, 下一次再询问
         */
//        NSURLCredential *credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];

        NSURLCredential *card = [[NSURLCredential alloc]initWithTrust:challenge.protectionSpace.serverTrust];

        completionHandler(NSURLSessionAuthChallengeUseCredential , card);
    }
}

3.2 如果是使用AFN框架，那么我们不需要做任何额外的操作，AFN内部已经做了处理。
```

###7 WebView的基本使用

```objc
1 概念性知识
    01 webView是有缺点的，会导致内存泄露，而且这个问题是它系统本身的问题。
    02 手机上面的safai其实就是用webView来实现的
    03 现在的开发并不完全是原生的开发，而更加倾向于原生+Html5的方式
    04 webView是OC代码和html代码之间进行交互的桥梁

2 代码相关
/*A*网页操控相关方法**/
    [self.webView goBack];      回退
    [self.webView goForward];   前进
    [self.webView reload];      刷新

    //设置是否能够前进和回退
    self.goBackBtn.enabled = webView.canGoBack;
    self.fowardBtn.enabled = webView.canGoForward;

/*B*常用的属性设置**/
    self.webView.scalesPageToFit = YES; 设置网页自动适应
    self.webView.dataDetectorTypes = UIDataDetectorTypeAll; 设置检测网页中的格式类型，all表示检测所有类型包括超链接、电话号码、地址等。
    self.webView.scrollView.contentInset = UIEdgeInsetsMake(50, 0, 0, 0);

/*C*相关代理方法**/
    //每当将加载请求的时候调用该方法，返回YES 表示加载该请求，返回NO 表示不加载该请求
    //可以在该方法中拦截请求
    -(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
    {
        return ![request.URL.absoluteString containsString:@"dushu"];
    }

    //开始加载网页，不仅监听我们指定的请求，还会监听内部发送的请求
    -(void)webViewDidStartLoad:(UIWebView *)webView

    //网页加载完毕之后会调用该方法
    -(void)webViewDidFinishLoad:(UIWebView *)webView

    //网页加载失败调用该方法
    -(void)webView:(UIWebView *)webView didFailLoadWithError:(NSError *)error

/*D*其它知识点-加载本地资源**/
    NSURL *url = [[NSBundle mainBundle] URLForResource:@"text.html" withExtension:nil];
    [self.webView loadRequest:[NSURLRequest requestWithURL:url]];
```

####8 HTML
```objc
1.Html决定网页的内容，css决定网页的样式，js决定网页的事件
2.html学习网站：http://www.w3school.com.cn

```
####9 OC和JS代码的互调
```objc
01 OC调用JS的代码
    NSString *str = [self.webView stringByEvaluatingJavaScriptFromString:@"sum()"];

02 JS怎么调用OC的说明
    新的需求：点击按钮的时候拨打电话
    但是我在点击按钮的时候，用户是不知道的，我们怎么能够知道用户点击了网页上面的一个按钮，只能通过一个技巧，那就是自己搞一个特定的协议头比如说xmg://,当我拦截到你的网络请求的时候，只需要判断一下当前的协议头是不是这个就能判断你现在是否是JS调用。
    OC里面有通过字符串生成SEL类型的方法，所以当拿到数据之后做下面的事情
    1）截取方法的名称
    2）将截取出来的字符串转换为SEL
    3）利用performSelect方法来调用SEL

03 涉及到的相关方法
    [@"abc" hasPrefix:@"A"] //判断字符串是否以一个固定的字符开头，这里为A
    //截串操作
    - (NSString *)substringFromIndex:(NSUInteger)from;
    //切割字符串，返回一个数组
    - (NSArray<NSString *> *)componentsSeparatedByString:(NSString *)separator;
    //替换操作
    - (NSString *)stringByReplacingOccurrencesOfString:(NSString *)target withString:(NSString *)replacement
    //把string包装成SEL

    SEL selector = NSSelectorFromString(sel);

04 如何屏蔽警告
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Warc-performSelector-leaks"
            //-Warc-performSelector-leaks为唯一的警告标识
            [self performSelector:selector withObject:nil];
    #pragma clang diagnostic pop
```

####9 NSInvocation的基本使用
```objc
//封装invacation可以调用多个参数的方法
-(void)invacation
{
    //1.创建一个MethodSignature，签名中保存了方法的名称，参数和返回值
    //这个方法属于谁，那么就用谁来进行创建
    //注意：签名一般是用来设置参数和获得返回值的，和方法的调用没有太大的关系
    NSMethodSignature *signature = [ViewController instanceMethodSignatureForSelector:@selector(callWithNumber:andContext:withStatus:)];

    /*注意不要写错了方法名称
     //    NSMethodSignature *signature = [ViewController methodSignatureForSelector:@selector(call)];
     */

    //2.通过MethodSignature来创建一个NSInvocation
    //NSInvocation中保存了方法所属于的对象|方法名称|参数|返回值等等
    NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:signature];

    /*2.1 设置invocation，来调用方法*/

    invocation.target = self;
    //    invocation.selector = @selector(call);
    //    invocation.selector = @selector(callWithNumber:);
    //    invocation.selector = @selector(callWithNumber:andContext:);
    invocation.selector = @selector(callWithNumber:andContext:withStatus:);

    NSString *number = @"10086";
    NSString *context = @"下课了";
    NSString *status = @"睡觉的时候";

    //注意：
    //1.自定义的参数索引从2开始，0和1已经被self and _cmd占用了
    //2.方法签名中保存的方法名称必须和调用的名称一致
    [invocation setArgument:&number atIndex:2];
    [invocation setArgument:&context atIndex:3];
    [invocation setArgument:&status atIndex:4];

    /*3.调用invok方法来执行*/
    [invocation invoke];
}
```
####10 异常处理
```objc
01 一般处理方式：
    a.app异常闪退，那么捕获crash信息，并记录在本地沙盒中。
    b.当下次用户重新打开app的时候，检查沙盒中是否保存有上次捕获到的crash信息。
    c.如果有那么利用专门的接口发送给服务器，以求在后期版本中修复。

02 如何抛出异常

    //抛出异常的两种方式
        // @throw  [NSException exceptionWithName:@"好大一个bug" reason:@"异常原因：我也不知道" userInfo:nil];

        //方式二
        NSString *info = [NSString stringWithFormat:@"%@方法找不到",NSStringFromSelector(aSelector)];
        //下面这种方法是自动抛出的
        [NSException raise:@"这是一个异常" format:info,nil];

03 如何捕获异常
    NSSetUncaughtExceptionHandler (&UncaughtExceptionHandler);

    void UncaughtExceptionHandler(NSException *exception) {
    NSArray *arr = [exception callStackSymbols];//得到当前调用栈信息
    NSString *reason = [exception reason];//非常重要，就是崩溃的原因
    NSString *name = [exception name];//异常类型

    NSString *errorMsg = [NSString stringWithFormat:@"当前调用栈的信息：%@\nCrash的原因：%@\n异常类型：%@\n",arr,reason,name];
    //把该信息保存到本地沙盒，下次回传给服务器。
}

```
####补充
    关于JS相关的学习框架：WebViewJavascriptBridge

