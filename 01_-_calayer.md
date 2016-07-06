# 01 - CALayer
01-CALayer的基本操作.
	
 ####1.CALayer简介:
        CALayer我们又称它叫做层.
		在每个UIView内部都有一个layer这样一个属性.    
		UIView之所以能够显示,就是因为它里面有这个一个层,才具有显示的功能.     
		我们通过操作CALayer对象,可以很方便地调整UIView的一些外观属性.    
		可以给UIView设置阴影,圆角,边框等等...     
                
####2.操作layer改变UIView外观.
	
		2.1.设置阴影
		默认图层是有阴影的, 只不过,是透明的
    	_RedView.layer.shadowOpacity = 1;
    	设置阴影的圆角
    	_RedView.layer.shadowRadius  =10;
    	设置阴影的颜色,把UIKit转换成CoreGraphics框架,用.CG开头
    	_RedView.layer.shadowColor = [UIColor blueColor].CGColor;
    	
    	2.2.设置边框
    	设置图层边框,在图层中使用CoreGraphics的CGColorRef
    	_RedView.layer.borderColor = [UIColor whiteColor].CGColor;
        _RedView.layer.borderWidth = 2;

		2.3.设置圆角
    	图层的圆角半径,圆角半径为宽度的一半, 就是一个圆
    	_RedView.layer.cornerRadius = 50;
		
####3.操作layer改变UIImageView的外观.
		
    	设置图形边框
    	_imageView.layer.borderWidth = 2;
    	_imageView.layer.borderColor = [UIColor whiteColor].CGColor;
		
		设置图片的圆角半径
		_imageView.layer.cornerRadius = 50;
		裁剪,超出裁剪区域的部分全部裁剪掉
    	_imageView.layer.masksToBounds = YES;
`注意:UIImageView当中Image并不是直接添加在层上面的.这是添加在layer当中的contents里.`

        我们设置层的所有属性它只作用在层上面.对contents里面的东西并不起作用.所以我们看不到图片有圆角的效果.
    	想要让图片有圆角的效果.可以把masksToBounds这个属性设为YES,
    	当设为YES,把就会把超过根层以外的东西都给裁剪掉.
    	
####4.layer的 CATransform3D属性.
      
      只有旋转的时候才可以看出3D的效果.
      旋转
      x,y,z 分别代表x,y,z轴.
      CATransform3DMakeRotation(M_PI, 1, 0, 0);
      平移
      CATransform3DMakeTranslation(x,y,z)
      缩放
      CATransform3DMakeScale(x,y,z);
      
      可以通过KVC的方式进行设置属性.
      但是CATransform3DMakeRotation它的值,是一个结构体, 所以要把结构转成对象.
      NSValue *value = [NSValue valueWithCATransform3D:CATransform3DMakeRotation(M_PI, 1, 0, 0)];
      [_imageView.layer setValue:value forKeyPath:@"transform.scale"];
     
     
      什么时候用KVC?
      当需要做一些快速缩放,平移,二维的旋转时用KVC.
      比如: [_imageView.layer setValue:@0.5 forKeyPath:@"transform.scale"];
      快速的进行缩放.
      后面forKeyPath属性值不是乱写的.苹果文档当中给了相关的属性.
      
      
      
02-自定义CALayer
	
	1.如何自定义Layer.
	自定义CALayer的方式创建UIView的方式非常相似.
	 CALayer *layer = [CALayer layer];
     layer.frame = CGRectMake(50, 50, 100, 100);
     layer.backgroundColor = [UIColor redColor].CGColor;
     [self.view.layer addSublayer:layer];

	给layer设置图片.
	layer.contents = (id)[UIImage imageNamed:@"阿狸头像"].CGImage;
	
	2.关于CALayer的疑惑?
	  为什么要使用CGImageRef、CGColorRef?
	  为了保证可移植性，QuartzCore不能使用UIImage、UIColor，只能使用CGImageRef、CGColorRef
	  
	  UIView和CALayer都能够显示东西,该怎样选择?
	  对比CALayer，UIView多了一个事件处理的功能。也就是说，CALayer不能处理用户的触摸事件，而UIView可以
	  如果显示出来的东西需要跟用户进行交互的话，用UIView；
	  如果不需要跟用户进行交互，用UIView或者CALayer都可以
	  CALayer的性能会高一些，因为它少了事件处理的功能，更加轻量级

03-position和anchorPoint
	
	position和anchorPoint是CAlayer的两个属性.
	我们以前修改一个控件的位置都是能过Frame的方式进行修改.
	现在利用CALayer的position和anchorPoint属性也能够修改控件的位置.
	这两个属性是配合使用的.
	position:它是用来设置当前的layer在父控件当中的位置的.
			  所以它的坐标原点.以父控件的左上角为(0.0)点.
	
	anchorPoint:它是决点CALayer身上哪一个点会在position属性所指的位置
			   anchorPoint它是以当前的layer左上角为原点(0.0)
			   它的取值范围是0~1,它的默认在中间也就是(0.5,0.5)的位置.
			   anchorPoint又称锚点.就是把锚点定到position所指的位置.
			   
	两者结合使用.想要修改某个控件的位置,我们可以设置它的position点.
			   设置完毕后.layer身上的anchorPoint会自动定到position所在的位置.
			   

04-隐式动画.
	
	什么是隐式动画?
	了解什么是隐式动画前,要先了解是什么根层和非根层.
	根层:UIView内部自动关联着的那个layer我们称它是根层.
	非根层:自己手动创建的层,称为非根层.
	
	隐式动画就是当对非根层的部分属性进行修改时, 它会自动的产生一些动画的效果.
	我们称这个默认产生的动画为隐式动画.
	
	如何取消隐式动画?
	首先要了解动画底层是怎么做的.动画的底层是包装成一个事务来进行的.
	什么是事务?
	很多操作绑定在一起,当这些操作执行完毕后,才去执行下一个操作.
	
	开启事务
	[CATransaction begin];
	设置事务没有动画
	[CATransaction setDisableActions:YES];
	设置动画执行的时长
	[CATransaction setAnimationDuration:2];
	
	
	提交事务
    [CATransaction commit];
	

	
	
05-时钟效果
	
	1.搭建界面.
		分析界面.
		界面上时针,分针,秒针不需要与用户进行交互.所以都可以使用layer方式来做.
		做之前要观察时针在做什么效果.
		是根据当前的时间,绕着表盘的中心点进行旋转.
		要了解一个非常重要的知识点.无论是旋转,缩放它都是绕着锚点.进行的.
		
		要想让时针,分针,称针显示的中间,还要绕着中心点进行旋转.
		那就要设置它的position和anchorPoint两个属性.
		
		
		创建秒针
	    CALayer *layer = [CALayer layer];
	     _secLayer = layer;
	    layer.bounds = CGRectMake(0, 0, 1, 80);
	    layer.anchorPoint = CGPointMake(0.5, 1);
	    layer.position = CGPointMake(_clockView.bounds.size.width * 0.5, 	_clockView.bounds.size.height * 0.5);
	    layer.backgroundColor = [UIColor redColor].CGColor;
	    [_clockView.layer addSublayer:layer];
	    
	    
	 2.让秒针开始旋转.
	 	
	 	让秒针旋转.所以要计算当前的旋转度是多少?
	 	当前的旋转角度为:当前的时间 * 每秒旋转多少度.
	 	
	 	计算每一秒旋转多少度.
	 	60秒转一圈360度
	 	360 除以60就是每一秒转多少度.每秒转6度.
	 	
	 	获取当前的时间
	 	创建日历类
        NSCalendar *calendar = [NSCalendar currentCalendar];
        把日历类转换成一个日期组件
        日期组件(年,月,日,时,分,秒)
        component:日期组件有哪些东西组成,他是一个枚举,里面有年月日时分秒
        fromDate:当前的日期
        NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond
         									   fromDate:[NSDate date]];
         									   
        我们的秒就是保存在日期组件里面,它里面提供了很多get方法.
        NSInteger second = cmp.second;
         
        那么当前秒针旋转的角度就是
        当前的秒数乘以每秒转多少度.
        second * perSecA 
        还得要把角度转换成弧度.
        
        因为下面分针,时针也得要用到, 就把它抽出一个速参数的宏.
        #define angle2Rad(angle) ((angle) / 180.0 * M_PI)
        
        让它每隔一秒旋转一次.所以添加一个定时器.
        每个一秒就调用,旋转秒针
         - (void)timeChange{
         获取当前的秒数
         创建日历类
         NSCalendar *calendar = [NSCalendar currentCalendar];
         把日历类转换成一个日期组件
         日期组件(年,月,日,时,分,秒)
         component:日期组件有哪些东西组成,他是一个枚举,里面有年月日时分秒
         fromDate:当前的日期
         NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond 
         								  	  fromDate:[NSDate date]];
         我们的秒就是保存在日期组件里面,它里面提供了很多get方法.
         NSInteger second = cmp.second;
         秒针旋转多少度.
         CGFloat angel = angle2Rad(second * perSecA);
         旋转秒针
         self.secondL.transform = CATransform3DMakeRotation(angel, 0, 0, 1);
         }
        运行发现他会一下只就调到某一个时间才开始旋转
        一开始的时候就要来到这个方法,获取当前的秒数把它定位好.
        要在添加定时器之后就调用一次timeChange方法.
        
	 	
	 	3.添加分针
	 	
	 	快速拷贝一下,然后添加一个分针成员属性.
        修改宽度,修改颜色
        也得要让它旋转,
        要算出每分钟转多少度
        转60分钟刚好是一圈
        所以每一分钟也是转6度.
        
        获取当前多少分?
        同样是在日期组件里面获得
        里面有左移符号,右移符号.他就可以用一个并运算
        现在同时让他支持秒数和分 后面直接加上一个 |
         NSDateComponents *cmp = [calendar components:NSCalendarUnitSecond | 
         									NSCalendarUnitMinute 
         									fromDate:[NSDate date]];
         									
        CGFloat minueteAngel = angle2Rad(minute * perMinuteA);
        self.minueL.transform = CATransform3DMakeRotation(minueteAngel, 0, 0, 1);
	 		
	 	4.添加时针
	 		
	 	 同样复制之前的,添加一个小时属性
         小时转多少度
         当前是多少小时,再计算先每一小时转多少度.
         12个小时转一圈. 360除以12,每小时转30度
         时针旋转多少度
         CGFloat hourAngel = angle2Rad(hour * perHourA);
         旋转时针
         self.hourL.transform = CATransform3DMakeRotation(hourAngel, 0, 0, 1);
            
        直接这样写会有问题
        就是没转一分钟,小时也会移动一点点
        接下来要算出,每一分钟,小时要转多少度
        60分钟一小时.一小时转30度.
        30 除以60,就是每一分钟,时针转多少度.0.5
 
        时针旋转多少度
        CGFloat hourAngel = angle2Rad(hour * perHourA + minute * perMinuteHourA);
        旋转时针
        self.hourL.transform = CATransform3DMakeRotation(hourAngel, 0, 0, 1);