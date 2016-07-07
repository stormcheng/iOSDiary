# 第六章 网络（2）


###1.NSURLConnection和Runloop（面试）
- 1.1 涉及知识点

（1）两种为NSURLConnection设置代理方式的区别

```objc
    //第一种设置方式：
    //通过该方法设置代理，会自动的发送请求
    // [[NSURLConnection alloc]initWithRequest:request delegate:self];

    //第二种设置方式：
    //设置代理，startImmediately为NO的时候，该方法不会自动发送请求
    NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
    //手动通过代码的方式来发送请求
    //注意该方法内部会自动的把connect添加到当前线程的RunLoop中在默认模式下执行
    [connect start];
 ```

（2）如何控制代理方法在哪个线程调用

```objc
    //说明：默认情况下，代理方法会在主线程中进行调用（为了方便开发者拿到数据后处理一些刷新UI的操作不需要考虑到线程间通信）
    //设置代理方法的执行队列
    [connect setDelegateQueue:[[NSOperationQueue alloc]init]];

```


（3）开子线程发送网络请求的注意点，适用于自动发送网络请求模式

```objc
     //使用GCD开启一个子线程来发送网络请求
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        //使用非自动发送网络请求模式,发送请求OK
        /*
        //创建NSURLConnection对象，设置代理，暂不发送
      NSURLConnection *connect =  [[NSURLConnection alloc]initWithRequest:request delegate:self startImmediately:NO];
        //设置代理方法的执行队列
        [connect setDelegateQueue:[[NSOperationQueue alloc]init]];

        //调用start发送网络请求
        [connect start];
        */

        //使用自动发送网络请求模式，发送请求失败（需要改造代码）
        //WHY?
        /*01 网络请求发送和数据接收是否成功，和一些因素相关，比如客户端的网速、服务器端的查询速度等等。
          02 而在子线程中创建的NSURLConnection对象是一个临时变量，当请求发送完成之后就被释放了，所以这个时候它的代理方法不会调用用。
          03 为什么使用非自动发送网络请求模式是OK的。
            因为在该模式中，调用了start来开始发送网络请求，该方法内部会自动将当前的connect作为一个Source添加到当前线程所在的Runloop中
            如果当前线程是子线程（即当前线程的runloop并未创建），那么该方法内部会默认先创建当前线程的Runloop,设置在runloop的默认模式下运行。
            此时runloop会对这个Connect对象进行强引用，保证了代理方法被调用的前提
         */
        NSURLConnection *connect = [[NSURLConnection alloc]initWithRequest:request delegate:self];
        [connect setDelegateQueue:[[NSOperationQueue alloc]init]];
        //创建当前线程的runloop，并开启runloop
        [[NSRunLoop currentRunLoop] run];
    });

    ```

---

###2.NSURLSession的基本使用
- 2.1 涉及知识点

（1）使用步骤

        使用NSURLSession创建task,然后执行task

（2）关于task

        a.NSURLSessionTask是一个抽象类，本身不能使用，只能使用它的子类
        b.NSURLSessionDataTask\NSURLSessionUploadTask\NSURLSessionDownloadTask

（3）发送get请求

```objc
    //1.创建NSURLSession对象（可以获取单例对象）
    NSURLSession *session = [NSURLSession sharedSession];

    //2.根据NSURLSession对象创建一个Task

    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login?username=ss&pwd=ss&type=JSON"];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];

    //方法参数说明
    /*
    注意：该block是在子线程中调用的，如果拿到数据之后要做一些UI刷新操作，那么需要回到主线程刷新
    第一个参数：需要发送的请求对象
    block:当请求结束拿到服务器响应的数据时调用block
    block-NSData:该请求的响应体
    block-NSURLResponse:存放本次请求的响应信息，响应头，真实类型为NSHTTPURLResponse
    block-NSErroe:请求错误信息
     */
   NSURLSessionDataTask * dataTask =  [session dataTaskWithRequest:request completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error) {

        //拿到响应头信息
        NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;

        //4.解析拿到的响应数据
        NSLog(@"%@\n%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],res.allHeaderFields);
    }];

    //3.执行Task
    //注意：刚创建出来的task默认是挂起状态的，需要调用该方法来启动任务（执行任务）
    [dataTask resume];
```
（4）发送get请求的第二种方式
```objc
  //注意：该方法内部默认会把URL对象包装成一个NSURLRequest对象（默认是GET请求）
    //方法参数说明
    /*
    //第一个参数：发送请求的URL地址
    //block:当请求结束拿到服务器响应的数据时调用block
    //block-NSData:该请求的响应体
    //block-NSURLResponse:存放本次请求的响应信息，响应头，真实类型为NSHTTPURLResponse
    //block-NSErroe:请求错误信息
     */
- (nullable NSURLSessionDataTask *)dataTaskWithURL:(NSURL *)url completionHandler:(void (^)(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error))completionHandler;
```
（5）发送POST请求
```objc
        //1.创建NSURLSession对象（可以获取单例对象）
    NSURLSession *session = [NSURLSession sharedSession];

    //2.根据NSURLSession对象创建一个Task

    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/login"];

    //创建一个请求对象，并这是请求方法为POST，把参数放在请求体中传递
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    request.HTTPMethod = @"POST";
    request.HTTPBody = [@"username=520it&pwd=520it&type=JSON" dataUsingEncoding:NSUTF8StringEncoding];

    NSURLSessionDataTask *dataTask = [session dataTaskWithRequest:request completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error) {
        //拿到响应头信息
        NSHTTPURLResponse *res = (NSHTTPURLResponse *)response;

        //解析拿到的响应数据
        NSLog(@"%@\n%@",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],res.allHeaderFields);
    }];

    //3.执行Task
    //注意：刚创建出来的task默认是挂起状态的，需要调用该方法来启动任务（执行任务）
    [dataTask resume];
```

---

###3.NSURLSession下载文件-代理
- 3.1 涉及知识点

（1）创建NSURLSession对象，设置代理（默认配置）
```objc
 //1.创建NSURLSession,并设置代理
    /*
     第一个参数：session对象的全局配置设置，一般使用默认配置就可以
     第二个参数：谁成为session对象的代理
     第三个参数：代理方法在哪个队列中执行（在哪个线程中调用）,如果是主队列那么在主线程中执行，如果是非主队列，那么在子线程中执行
     */
    NSURLSession *session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];
```
（2）根据Session对象创建一个NSURLSessionDataTask任务（post和get选择）
```objc
//创建task
NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_01.png"];

//注意：如果要发送POST请求，那么请使用dataTaskWithRequest,设置一些请求头信息
NSURLSessionDataTask *dataTask = [session dataTaskWithURL:url];
```
（3）执行任务（其它方法，如暂停、取消等）
```objc
    //启动task
    //[dataTask resume];
    //其它方法，如取消任务，暂停任务等
    //[dataTask cancel];
    //[dataTask suspend];
```
（4）遵守代理协议，实现代理方法（3个相关的代理方法）
```objc
/*
 1.当接收到服务器响应的时候调用
     session：发送请求的session对象
     dataTask：根据NSURLSession创建的task任务
     response:服务器响应信息（响应头）
     completionHandler：通过该block回调，告诉服务器端是否接收返回的数据
 */
-(void)URLSession:(nonnull NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask didReceiveResponse:(nonnull NSURLResponse *)response completionHandler:(nonnull void (^)(NSURLSessionResponseDisposition))completionHandler

/*
 2.当接收到服务器返回的数据时调用
 该方法可能会被调用多次
 */
-(void)URLSession:(nonnull NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask didReceiveData:(nonnull NSData *)data

/*
 3.当请求完成之后调用该方法
 不论是请求成功还是请求失败都调用该方法，如果请求失败，那么error对象有值，否则那么error对象为空
 */
-(void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didCompleteWithError:(nullable NSError *)error
```
（5）当接收到服务器响应的时候，告诉服务器接收数据（调用block）
```objc
//默认情况下，当接收到服务器响应之后，服务器认为客户端不需要接收数据，所以后面的代理方法不会调用
    //如果需要继续接收服务器返回的数据，那么需要调用block,并传入对应的策略

    /*
        NSURLSessionResponseCancel = 0, 取消任务
        NSURLSessionResponseAllow = 1,  接收任务
        NSURLSessionResponseBecomeDownload = 2, 转变成下载
        NSURLSessionResponseBecomeStream NS_ENUM_AVAILABLE(10_11, 9_0) = 3, 转变成流
    */

    completionHandler(NSURLSessionResponseAllow);
```

---

###4.NSURLSessionDownloadTask实现大文件下载

- 4.1 涉及知识点

（1）使用NSURLSession和NSURLSessionDownload可以很方便的实现文件下载操作
```objc
 /*
     第一个参数：要下载文件的url路径
     第二个参数：当接收完服务器返回的数据之后调用该block
     location:下载的文件的保存地址（默认是存储在沙盒中tmp文件夹下面，随时会被删除）
     response：服务器响应信息，响应头
     error：该请求的错误信息
     */
    //说明：downloadTaskWithURL方法已经实现了在下载文件数据的过程中边下载文件数据，边写入到沙盒文件的操作
    NSURLSessionDownloadTask *downloadTask = [session downloadTaskWithURL:url completionHandler:^(NSURL * __nullable location, NSURLResponse * __nullable response, NSError * __nullable error)
```
（2）downloadTaskWithURL内部默认已经实现了变下载边写入操作，所以不用开发人员担心内存问题

（3）文件下载后默认保存在tmp文件目录，需要开发人员手动的剪切到合适的沙盒目录

（4）缺点：没有办法监控下载进度

---

###5.使用NSURLSessionDownloadTask实现大文件下载-监听下载进度
- 5.1 涉及知识点

（1）创建NSURLSession并设置代理，通过NSURLSessionDownloadTask并以代理的方式来完成大文件的下载
```objc
     //1.创建NSULRSession,设置代理
    self.session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration] delegate:self delegateQueue:[NSOperationQueue mainQueue]];

    //2.创建task
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];
    self.downloadTask = [self.session downloadTaskWithURL:url];

    //3.执行task
    [self.downloadTask resume];
```
（2）常用代理方法的说明
```objc
    /*
 1.当接收到下载数据的时候调用,可以在该方法中监听文件下载的进度
 该方法会被调用多次
 totalBytesWritten:已经写入到文件中的数据大小
 totalBytesExpectedToWrite:目前文件的总大小
 bytesWritten:本次下载的文件数据大小
 */
-(void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
/*
 2.恢复下载的时候调用该方法
 fileOffset:恢复之后，要从文件的什么地方开发下载
 expectedTotalBytes：该文件数据的总大小
 */
-(void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
/*
 3.下载完成之后调用该方法
 */
-(void)URLSession:(nonnull NSURLSession *)session downloadTask:(nonnull NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(nonnull NSURL *)location
/*
 4.请求完成之后调用
 如果请求失败，那么error有值
 */
-(void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didCompleteWithError:(nullable NSError *)error
```
（3）实现断点下载相关代码
```objc
    //如果任务，取消了那么以后就不能恢复了
    //    [self.downloadTask cancel];

    //如果采取这种方式来取消任务，那么该方法会通过resumeData保存当前文件的下载信息
    //只要有了这份信息，以后就可以通过这些信息来恢复下载
    [self.downloadTask cancelByProducingResumeData:^(NSData * __nullable resumeData) {
        self.resumeData = resumeData;
    }];

    -----------
    //继续下载
    //首先通过之前保存的resumeData信息，创建一个下载任务
    self.downloadTask = [self.session downloadTaskWithResumeData:self.resumeData];

     [self.downloadTask resume];
```
（4）计算当前下载进度

```objc
    //获取文件下载进度
    self.progress.progress = 1.0 * totalBytesWritten/totalBytesExpectedToWrite;
 ```

（5）局限性

    01 如果用户点击暂停之后退出程序，那么需要把恢复下载的数据写一份到沙盒，代码复杂度更
    02 如果用户在下载中途未保存恢复下载数据即退出程序，则不具备可操作性

---

###6.使用NSURLSessionDataTask实现大文件离线断点下载（完整）
- 6.1 涉及知识点

（1）关于NSOutputStream的使用
```objc
    //1. 创建一个输入流,数据追加到文件的屁股上
    //把数据写入到指定的文件地址，如果当前文件不存在，则会自动创建
    NSOutputStream *stream = [[NSOutputStream alloc]initWithURL:[NSURL fileURLWithPath:[self fullPath]] append:YES];

    //2. 打开流
    [stream open];

    //3. 写入流数据
    [stream write:data.bytes maxLength:data.length];

    //4.当不需要的时候应该关闭流
    [stream close];
```
（2）关于网络请求请求头的设置（可以设置请求下载文件的某一部分）
```objc
    //1. 设置请求对象
    //1.1 创建请求路径
    NSURL *url = [NSURL URLWithString:@"http://120.25.226.186:32812/resources/videos/minion_01.mp4"];

    //1.2 创建可变请求对象
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];

    //1.3 拿到当前文件的残留数据大小
    self.currentContentLength = [self FileSize];

    //1.4 告诉服务器从哪个地方开始下载文件数据
    NSString *range = [NSString stringWithFormat:@"bytes=%zd-",self.currentContentLength];
    NSLog(@"%@",range);

    //1.5 设置请求头
    [request setValue:range forHTTPHeaderField:@"Range"];
```
（3）NSURLSession对象的释放
```objc
-(void)dealloc
{
    //在最后的时候应该把session释放，以免造成内存泄露
    //    NSURLSession设置过代理后，需要在最后（比如控制器销毁的时候）调用session的invalidateAndCancel或者resetWithCompletionHandler，才不会有内存泄露
    //    [self.session invalidateAndCancel];
    [self.session resetWithCompletionHandler:^{

        NSLog(@"释放---");
    }];
}
```
（4）优化部分

        01 关于文件下载进度的实时更新
        02 方法的独立与抽取

---

###7.NSURLSession实现文件上传
- 7.1 涉及知识点

（1）实现文件上传的方法
```objc
      /*
     第一个参数：请求对象
     第二个参数：请求体（要上传的文件数据）
     block回调：
     NSData:响应体
     NSURLResponse：响应头
     NSError：请求的错误信息
     */
    NSURLSessionUploadTask *uploadTask =  [session uploadTaskWithRequest:request fromData:data completionHandler:^(NSData * __nullable data, NSURLResponse * __nullable response, NSError * __nullable error)
```
（2）设置代理，在代理方法中监听文件上传进度
```objc
/*
 调用该方法上传文件数据
 如果文件数据很大，那么该方法会被调用多次
 参数说明：
     totalBytesSent：已经上传的文件数据的大小
     totalBytesExpectedToSend：文件的总大小
 */
-(void)URLSession:(nonnull NSURLSession *)session task:(nonnull NSURLSessionTask *)task didSendBodyData:(int64_t)bytesSent totalBytesSent:(int64_t)totalBytesSent totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend
{
    NSLog(@"%.2f",1.0 * totalBytesSent/totalBytesExpectedToSend);
}
```
（3）关于NSURLSessionConfiguration相关

    01 作用：可以统一配置NSURLSession,如请求超时等
    02 创建的方式和使用

```objc
//创建配置的三种方式
+ (NSURLSessionConfiguration *)defaultSessionConfiguration;
+ (NSURLSessionConfiguration *)ephemeralSessionConfiguration;
+ (NSURLSessionConfiguration *)backgroundSessionConfigurationWithIdentifier:(NSString *)identifier NS_AVAILABLE(10_10, 8_0);

//统一配置NSURLSession
-(NSURLSession *)session
{
    if (_session == nil) {

        //创建NSURLSessionConfiguration
        NSURLSessionConfiguration *config = [NSURLSessionConfiguration defaultSessionConfiguration];

        //设置请求超时为10秒钟
        config.timeoutIntervalForRequest = 10;

        //在蜂窝网络情况下是否继续请求（上传或下载）
        config.allowsCellularAccess = NO;

        _session = [NSURLSession sessionWithConfiguration:config delegate:self delegateQueue:[NSOperationQueue mainQueue]];
    }
    return _session;
}

```

---
###8.AFN框架基本使用
- 8.1 AFN内部结构

```objc
AFN结构体
    - NSURLConnection
        + AFURLConnectionOperation
        + AFHTTPRequestOperation
        + AFHTTPRequestOperationManager(封装了常用的 HTTP 方法)
            * 属性
                * baseURL :AFN建议开发者针对 AFHTTPRequestOperationManager 自定义个一个单例子类，设置 baseURL, 所有的网络访问，都只使用相对路径即可
                * requestSerializer :请求数据格式/默认是二进制的 HTTP
                * responseSerializer :响应的数据格式/默认是 JSON 格式
                * operationQueue
                * reachabilityManager :网络连接管理器
            * 方法
                * manager :方便创建管理器的类方法
                * HTTPRequestOperationWithRequest :在访问服务器时，如果要告诉服务器一些附加信息，都需要在 Request 中设置
                * GET
                * POST

    - NSURLSession
        + AFURLSessionManager
        + AFHTTPSessionManager(封装了常用的 HTTP 方法)
            * GET
            * POST
            * UIKit + AFNetworking 分类
            * NSProgress :利用KVO

    - 半自动的序列化&反序列化的功能
        + AFURLRequestSerialization :请求的数据格式/默认是二进制的
        + AFURLResponseSerialization :响应的数据格式/默认是JSON格式
    - 附加功能
        + 安全策略
            * HTTPS
            * AFSecurityPolicy
        + 网络检测
            * 对苹果的网络连接检测做了一个封装
            * AFNetworkReachabilityManager

建议:
可以学习下AFN对 UIKit 做了一些分类, 对自己能力提升是非常有帮助的
```
- 8.2 AFN的基本使用

（1）发送GET请求的两种方式（POST同）
```objc
-(void)get1
{
    //1.创建AFHTTPRequestOperationManager管理者
    //AFHTTPRequestOperationManager内部是基于NSURLConnection实现的
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];

    //2.发送请求
    /*
     http://120.25.226.186:32812/login?username=ee&pwd=ee&type=JSON
     第一个参数：NSString类型的请求路径，AFN内部会自动将该路径包装为一个url并创建请求对象
     第二个参数：请求参数，以字典的方式传递，AFN内部会判断当前是POST请求还是GET请求，以选择直接拼接还是转换为NSData放到请求体中传递
     第三个参数：请求成功之后回调Block
     第四个参数：请求失败回调Block
     */

    NSDictionary *param = @{
                            @"username":@"520it",
                            @"pwd":@"520it"
                            };

    //注意：字符串中不能包含空格
    [manager GET:@"http://120.25.226.186:32812/login" parameters:param success:^(AFHTTPRequestOperation * _Nonnull operation, id  _Nonnull responseObject) {

        NSLog(@"请求成功---%@",responseObject);

    } failure:^(AFHTTPRequestOperation * _Nonnull operation, NSError * _Nonnull error) {
            NSLog(@"失败---%@",error);
    }];
}

-(void)get2
{
    //1.创建AFHTTPSessionManager管理者
    //AFHTTPSessionManager内部是基于NSURLSession实现的
    AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];

    //2.发送请求
    NSDictionary *param = @{
                            @"username":@"520it",
                            @"pwd":@"520it"
                            };

    //注意：responseObject:请求成功返回的响应结果（AFN内部已经把响应体转换为OC对象，通常是字典或数组）
    [manager GET:@"http://120.25.226.186:32812/login" parameters:param success:^(NSURLSessionDataTask * _Nonnull task, id  _Nonnull responseObject) {
            NSLog(@"请求成功---%@",[responseObject class]);

    } failure:^(NSURLSessionDataTask * _Nonnull task, NSError * _Nonnull error) {
        NSLog(@"失败---%@",error);
    }];
}
```

（2）使用AFN下载文件
```objc
-(void)download
{
    //1.创建一个管理者
    AFHTTPSessionManager *manage  = [AFHTTPSessionManager manager];

    //2.下载文件
    /*
     第一个参数：请求对象
     第二个参数：下载进度
     第三个参数：block回调，需要返回一个url地址，用来告诉AFN下载文件的目标地址
         targetPath：AFN内部下载文件存储的地址，tmp文件夹下
         response：请求的响应头
         返回值：文件应该剪切到什么地方
     第四个参数：block回调，当文件下载完成之后调用
        response：响应头
        filePath：文件存储在沙盒的地址 == 第三个参数中block的返回值
        error：错误信息
     */

    //2.1 创建请求对象
    NSURLRequest *request = [NSURLRequest requestWithURL:[NSURL URLWithString:@"http://120.25.226.186:32812/resources/images/minion_02.png"]];

    //2.2 创建下载进度，并监听
    NSProgress *progress = nil;

    NSURLSessionDownloadTask *downloadTask = [manage downloadTaskWithRequest:request progress:&progress destination:^NSURL * _Nonnull(NSURL * _Nonnull targetPath, NSURLResponse * _Nonnull response) {

        NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];

        //拼接文件全路径
        NSString *fullpath = [caches stringByAppendingPathComponent:response.suggestedFilename];
        NSURL *filePathUrl = [NSURL fileURLWithPath:fullpath];
        return filePathUrl;

    } completionHandler:^(NSURLResponse * _Nonnull response, NSURL * _Nonnull filePath, NSError * _Nonnull error) {

        NSLog(@"文件下载完毕---%@",filePath);
    }];

    //2.3 使用KVO监听下载进度
    [progress addObserver:self forKeyPath:@"completedUnitCount" options:NSKeyValueObservingOptionNew context:nil];

    //3.启动任务
    [downloadTask resume];
}

//获取并计算当前文件的下载进度
-(void)observeValueForKeyPath:(NSString *)keyPath ofObject:(NSProgress *)progress change:(NSDictionary<NSString *,id> *)change context:(void *)context
{
    NSLog(@"%zd--%zd--%f",progress.completedUnitCount,progress.totalUnitCount,1.0 * progress.completedUnitCount/progress.totalUnitCount);
}
```
###9.Cocoapods的安装
```objc
1.先升级Gem
    sudo gem update --system
2.切换cocoapods的数据源
    【先删除，再添加，查看】
    gem sources --remove https://rubygems.org/
    gem sources -a http://ruby.taobao.org/
    gem sources -l
3.安装cocoapods
    sudo gem install cocoapods
4.将Podspec文件托管地址从github切换到国内的oschina
    【先删除，再添加，再更新】
    pod repo remove master
    pod repo add master http://git.oschina.net/akuandev/Specs.git
    pod repo add master https://gitcafe.com/akuandev/Specs.git
    pod repo update
5.设置pod仓库
    pod setup
6.测试
    【如果有版本号，则说明已经安装成功】
    pod --version
7.利用cocoapods来安装第三方框架
    01 进入要安装框架的项目的.xcodeproj同级文件夹
    02 在该文件夹中新建一个文件Podfile
    03 在文件中告诉cocoapods需要安装的框架信息
        a.该框架支持的平台
        b.适用的iOS版本
        c.框架的名称
        d.框架的版本
8.安装
pod install --no-repo-update
pod update --no-repo-update

```
