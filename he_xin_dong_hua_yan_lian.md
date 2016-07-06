# 核心动画演练

01-转盘

	1.搭建界面
		把转盘View给封装起来. 由于界面是固定不变的,可以弄一个Xib展示界面.
		外界使用时直接来一个类方法直接调用.
	2.让转盘进行旋转.
		在封装的View内部提供一个开始旋转的方法和结束旋转的方法,供外界直接调用.
		在View内部实现方法.
		开始旋转:
			添加核心动画.动画要添加到里面的背景图片上.不能够直接添加到View.
	3.添加按钮.
		1.分析按钮的位置.每一个都是倾斜的.想要办到这种效果,可以先把所有的按钮都给添加到圆盘的最上方
		然后让每一个按钮绕着圆盘的中心点进行旋转.
		想要让按钮绕着圆盘中心点进行旋转,要选把每个按钮的锚点设置到按钮底部中心位置(0.5,1)
		再让按钮显示到中心的位置,设置按钮的position为圆盘的中心即可.
		
		2.添加按钮时,让每一个按钮进行旋转,要计算出每一个按钮旋转的角度.总共有12个按钮,正好占圆的一圈,
		  那每一个按钮占30.那一个按钮在上一个按钮的基础上加上30.即可算出每一个按钮旋转的角度.
		  
	4.给按钮添加点击事件.让按钮成为选中状态,取消按钮的高亮状态.
		1.在添加按钮时, 给每一个按钮添加选中状态.
		  添加按钮事件的时候要注意.我们每一个按钮都是添加到UIImageView上的.UIImageView默认是不接收事件的.
		  所以需要手动把UIImageView的设置为可接收事件.
		  
		2.让按钮成为选中状态.
		  在事件处理方法当中,我们要先定义一个成员属性记录上一个按钮的选中状态.
		  先把上一个按钮的选中状态取消选中.
		  让当前点击的按钮成为选中状态.
		  把当前的按钮赋值上一个按钮.
		  
	5.设置按钮每一个按钮的图片
			
		1.查看美工提供的资料文件.发现所有的按钮都在一张图片上面.
		  这个时候我们就得需要把这一张图片做一个裁剪.
		  首先要确定裁剪的每一张图片的宽,高
		  每一张图片的高度, 为原始图片的高度
		  每一张图片的宽度, 为整个图片的宽度除以总共要裁剪多少张图片.这里我们总共有12张图片, 所以总宽度除以12.
		  用下面这个方法做裁剪.
		  参数说明:
		  image:为要裁剪的图片,既原始图片.它的类型为CGImageRef,类型,所以要转成CGImage
		  rect:为裁剪的范围.这时需要确定每一个X的位置.
		  每一个x为它当前的角标 * 要裁剪的宽度.
		  CGImageCreateWithImageInRect(image, rect);
		  
		  这个方法它会返回一个图片.图片类型为CGImageRef imgR.
		  所要我们在设置背景图片的时候,要把这张图片给转成UIImage.
		  转的方法为:
		  UIImage *image = [UIImage imageWithCGImage:imgR];
		  
	   2.运行时只能看见图片的一部分
	   	  运行的时候只能看到图片的一部分.这个是因为我们加载图片,加载的是@2X的图片.
	   	  而我们CGImageCreateWithImageInRect这个方法它是C语言的方法.它裁剪的区域是一个像素点.
	   	  所以导致我们看到的只有这一部分内容
	   	  解决办法为把裁剪的宽高都乘上一个像素比例.
	   	  获得像素比例
	   	  [UIScreen mainScreen].scale
	   	  这样裁剪出来的图片就是整个图片了.
		 
	   3.但是运行的时候还是发现裁剪的图片尺寸比较大.
	   	  这个时候我们要修改按钮内部图片的尺寸大小.
	   	  在按钮当中实现一个方法.在这个方法当中重新计算图片的尺寸
	   	  -(CGRect)imageRectForContentRect:(CGRect)contentRect{
    
		    CGFloat btnW = 40;
    		CGFloat btnH = 47;
    		CGFloat btnY = 20;
		    CGFloat btnX = (contentRect.size.width - btnW) * 0.5;
		    return CGRectMake(btnX, btnY, btnW, btnH);

		  }	
		  
		  
	6.在按钮旋转的时候,还要点击按钮所以这个时候就不能够用核心动画.
		这个时候我们就要用UIView动画.
		
		方法为:添加一个定时器, 每次在原来基础上添加一个旋转角度.
		定时器我们只需要添加一次, 我们就采用懒加载的方法.我们采用CADisplayLink的方法.
		因为这个定时器可以让它很方便的暂停,开始.
		
		-(CADisplayLink *)link{
    
    	if (_link == nil) {
        _link = [CADisplayLink displayLinkWithTarget:self selector:@selector(rotationChange)];
        [_link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
    	}
    		return _link;
    	}
    	
      在定时器方法当中,做一个旋转,每次让它在原来的基础上旋转一个角度.
      
      
    7.实现中间点击按钮.
      中间点击按钮为逻辑为,快速的让圆盘旋转几圈,动画完成时,让选择的按钮指上最上方.
      快速旋转几圈, 这个时候不需要与用户进行交互,所以我们可以使用核心动画.
      在动画开始的时候,停止定时器.
      得要监听动画完成.在动画完成的时候,来做事情.
      想要坚听动画的完成.我们可以设置核心动画的代理.
      
      核心动画代理方法比较特殊.它不需要遵守协议,它是NSObject的一个分类.也称它是非常式协议.
      直接实现它的方法就可以了.
      
      
      在这个方法当中要实现的思路为.
      看当前选中的按钮之前是旋转多少度旋转到当前位置的.
      然后再让转盘倒着旋转回去就可以了.
      
      
      可以通过当前按钮的transform求出当前按钮的旋转角度
      方法为:
      CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);
      
      -(void)animationDidStop:(nonnull CAAnimation *)anim finished:(BOOL)flag{
      
		 CGAffineTransform transform = self.selectBtn.transform;
		 CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);
		 self.innerCircle.transform = CGAffineTransformMakeRotation(-angel);
		 
	  }
	  

02-图片折叠
	
	1.分析界面效果
		当鼠标在图片上拖动的时候,图片有一个折叠的效果.
		这种折叠效果其实就是图片的上半部分绕着X轴做一个旋转的操作.
		我们图片的旋转都是绕着锚点进行旋转的.所以如果是一张图片的,办不到只上图片的上半部分进行一个旋转.
		其实是两张图片, 把两张图片合成一张图片的方法, 
		实现方案.弄上下两张图片,上上部图片只显示上半部分, 下部图片只显示下半部分.
		
	 2.如果让一张图片只显示上半部分或者下半部分?
		利用CALayer的一个属性contentsRect = CGRectMake(0, 0, 1, 0.5);
		contentsRect就是要显示的范围.它是取值范围是(0~1);
		想让上部图片只显示上半部分contentsRect设置CGRectMake(0, 0, 1, 0.5); 
		让下部图片只显示下半部分contentsRect设置为CGRectMake(0, 0.5, 1, 0.5)
		
	 3.让上部图片绕着锚点进行旋转,但是图片的默认锚点在中心,所以要把上部图片的锚点设在最底部.
	 	修改上部分的锚点为anchorPoint = CGPointMake(0.5, 1)
	 	但是修过锚点之后, 会出现一个问题,就是上部分的图片,会往上走.导致两个图片中间有一个空隙.
	 	解决办法为.把两张图片放到一起.设置上部分图片的锚点后.上部分图片会上走一半的距离.
	 	然后再设置下部图片的锚点.设置上图片的最上面设置下部图片锚点值为anchorPoint = CGPointMake(0.5, 0);
		这样就能够办到两张图片合成一张的效果.
		
	 4. 给上部图片添加手势.根据手指向下拖动的距离.来计算旋转的角度.
	 	拖动的时候,发现它的拖动范围为整个图片.所以添加手势的时候,不能只添加上部分或着下部分.
	 	可以弄一个跟View相同大小的的View,把手势添加给这个UIView.
	 	添加完手势时, 在手势方法当中去旋转上部分图片.
	 	要來计算旋转的角度,上半部分旋转的角度是根据手指向上拖动的Y值来决定.当手指到最下部时正好旋转180度.
	 	假设手指移动到最下部为200.那旋转角度应该为 angel =  transP.Y * M_PI / 200.0;
	 	
	 	
	5. 拖动的时候让它有一个立体产效果
		立体的效果就是有一种近大远小的感觉.	
		想要设置立体效果.要修改它的TransForm当中的一个M34值,设置方式为
		弄一个空的TransFrom3D
		CATransform3D transform = CATransform3DIdentity; 
		200.0可以理解为，人的眼睛离手机屏幕的垂直距离，近大远小效果越明显
		transform.m34 = - 1 / 200.0;	
	 	transform = CATransform3DRotate(transform, angle, 1, 0, 0);
	 	相对上一次改了m34的形变,再去旋转
	 	为什么不用Make,make会上m34清空,这个地方不能让m34清空
	 	
	6. 给下部分图片添加阴影的效果.阴影是有渐变的效果.是从透明到黑色的一个渐变.
		我们可以通过CAGradientLayer这个层来创建渐变.这个层我们就称它是一个渐变层.
		渐变层也是需要添加到一个层上面才能够显示.
		
		渐变层里面有一个colors属性.这个属性就是设置要渐变的颜色.它是一个数组.
		数组当中要求我们传入都是CGColorRef类型,所以我们要把颜色转成CGColor.
		但是转成CGColor后,数组就认识它是一个对象了,就要通过在前面加上一个(id)来告诉编译器是一个对象.
		
		可以设置渐变的方向:
		通过startPoint和endPoint这两个属性来设置渐变的方向.它的取值范围也是(0~1)
		
		 默认方向为上下渐变为:
         	gradientL.startPoint = CGPointMake(0, 0);
         	gradientL.endPoint = CGPointMake(0, 1);
         设置左右渐变
          	gradientL.startPoint = CGPointMake(0, 0);
          	gradientL.endPoint = CGPointMake(1, 0);
		
	 	可以设置渐变从一个颜色到下一个颜色的位置
	 	设置属性为locations = @[@0.3,@0.7]
	 	
	 	渐变层同时还有一个opacity属性.这个属性是调协渐变层的不透明度.它的取值范围同样也是0-1,
	 	当为0时代表透明, 当为1明,代码不透明.
	 	
	 	
	 	所以我们可以给下部分图片添加一个渐变层,渐变层的颜色为从透明到黑色.
	 	gradientL.colors = 
	 	@[(id)[UIColor clearColor].CGColor,(id)[UIColor blackColor].CGColor];
	 	
	 	开始时没有渐变,所以可以把渐变层设为透明状态.
	 	之后随着手指向下拖动的时,渐变层的透明度跟着改变.
	 	
	 	当手指拖到最下面的时候,渐变层的透明度正好为1.所以要中间可以有一个比例.
	 	 CGFloat opacity  =  1 * transP.y / 200.0;
	 	在手指拖动的时候,设置它的不透度
	 	
	 	
	7. 在手指拖动过程当中,松开手指时,有一个动画返弹回去的效果.
		所以我们要坚听手指的状态.当手势状态为结束时
		把渐变层的透明度设为透明
		把上部图片的旋转设为0,也就是清空之前的形变.
		
		同时加上一个返弹动画的效果.
		返弹动画添加方法为
		
			
			
		 Duration:动画的执行时长
		 delay:动画延时时长.
		 Damping:动画的弹性系数,越小,弹簧效果越明显
		 initialSpringVelocity:弹簧初始化速度
		 [UIView animateWithDuration:0.8 delay:0 usingSpringWithDamping:0.1        												   initialSpringVelocity:0 
		 										options:UIViewAnimationOptionCurveLinear 
		 										animations:^{
        
        						动画执行代码    
        
      	  } completion:^(BOOL finished) {
            	动画完成时调用.
        }];
		
		
		

03-音乐震动条

	分析震动条界面
	每一个条都在做一个上下缩放的动画.而且不需要与用户交互.所以每一个震动条可以用CALayer来做.
	发现每个都非常相似.所以先搞定一个,然后其它的直接复制就可以了.
	
	1.演示界面
    分析,改它的缩放, 然后再还原
    
    2. 搭建界面
    宽度200,高度:200,拖线
    
    3.添加CALayer,振动条,设置尺寸,大小,
     CALayer *layer = [CALayer layer];
     设置背景色
     layer.backgroundColor = [UIColor redColor].CGColor;
     设置尺寸
     CGFloat barH  = 120;
     CGFloat barW = 30;
     CGFloat X = 0;
     CGFloat Y = _contentsView.bounds.size.height - barH;
     layer.frame = CGRectMake(X, Y, barW, barH);
     [_contentsView.layer addSublayer:layer];
     
     
     4.添加动画
    	4.1添加高度缩小后,马上还原
        	为什么选用核心动画?
        	给图层做动画用核心动画,不需要与用户做交互.
 
    	4.2采用哪一种核心动画?
 
        把它的缩放改成某个值就好了.选用CABasicAnimation
         CABasicAnimation *anim = [CABasicAnimation animation];
         动画时长
         anim.duration = 2;
         形变缩放动画
         anim.keyPath = @"transform.scale";
         改为0,让它缩小
         anim.toValue = @0;
         让动画一直执行
         anim.repeatCount = MAXFLOAT;
         反转回去
         anim.autoreverses = YES;
         [layer addAnimation:anim forKey:nil];
 
        运行发现:不是我们想要的上下进行缩放?它是绕着中心点去缩放?
          所以要设置锚点. 设置锚点就不能设置frame了, 要设置Position
 
         设置锚点
         layer.anchorPoint = CGPointMake(0, 1);
         设置position
         layer.position = CGPointMake(0, _contentsView.bounds.size.height);
          不能够再用frame,设置它的bounds
         layer.bounds = CGRectMake(0, 0, barW, barW);
         
        运行发现:是不是 x轴 和 Y轴一起缩放了
          不需要X轴缩放,所以更改动画的KeyPath
          anim.keyPath = @"transfrom.scale.y"
 
    5. 使用复制层
         复制层可以把它里面的所有子层给复制.
        
         添加复制层,首先先要让这个层显示出来.
         复制层必须加到一个层里面才能复制它的子层.
         不需要设置它的尺寸, 需要设置它的颜色.子层超过父层也能够显示,所以不用设置尺寸.
 
         CAReplicatorLayer *replicator = [CAReplicatorLayer  layer];
         将复制层添加到_contententView.layer
         [_contentsView.layer addSublayer:replicator];
 
         instanceCount:表示原来层中的所有子层复制的份数
         replicator.instanceCount = 2;

         在复制层中添加子层
         [replicator addSublayer:layer];
         
         运行发现没有两个,为什么?
         其实已经有两个了,两个的位置,尺寸都是一样的,所以看着只有一个.
         
         解决办法:让子层有偏移位置
         instanceTransform:复制出来的层,相对上一个子层的形变
         replicator.instanceTransform = CATransform3DMakeTranslation(45, 0, 0);
 
 
         现在复制出来的层,执行的动画,都一样, 怎么样才能有错乱的感觉?
 
         相对于上一个层的动画延时
         replicator.instanceDelay = 0.3;
 
         使用它,应该把原始层的颜色设置为白色
         replicator.instanceColor = [UIColor greenColor].CGColor;
	  	

04-倒影
	
	 1.演示界面,分析界面
        将一张图片绕着X轴旋转180度
        两个是一模一样的,可以用复制层
        图片加到控制器上面, 让控制器成为复制层
 
    2. 搭建界面, 拖入图片
       复制控制器的View里面的子层, 因此控制器的View它的层必须要是复制层.
       打印出控制器的层,是CALayer,CALayer不是复制层, 不能复制.
       如何让控制器的View是一个复制层?
       想要修改系统的东西,必须得要自定义View
 
      2.1在自定义的View中修改根层的类型
 
         修改这个View的图层类型为复制层
         返回的就是这个View根层的类型
         + (nonnull Class)layerClass{
            return [CAReplicatorLayer class];
         }
     2.2 在ViewDidLoad中添加复制层
 
         CAReplicatorLayer *repL = (CAReplicatorLayer *)self.view.layer;
         repL.instanceTransform = CATransform3DMakeRotation(M_PI, 1, 0, 0);
         repL.instanceCount = 2;
        运行发现中间有一个间隙
        原因:它是绕着父层的锚点旋转,
        验证:在StroyBoard中,把图片拖到View的中间,运行,看到两个图片正好重合
        解决办法:在StoryBoard中,用约束让它水平,垂直剧中,两个正好重合
                想要让它有倒影效果, 让它往上移动一段距离, 往上面移一半的高度
 
    3.让底部图片有倒影效果
        
        更改颜色通道
        每一个颜色通道减等于0.1
 
         更改每一个颜色通道
         repL.instanceRedOffset -= 0.1;
         repL.instanceBlueOffset -= 0.1;
         repL.instanceGreenOffset -= 0.1;
         repL.instanceAlphaOffset -= 0.1;


