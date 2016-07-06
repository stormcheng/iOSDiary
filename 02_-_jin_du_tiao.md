# 02 - 进度条

```objc
	1.搭建界面.

	2.拖动滑块的时候让他里面的能够跟着我的拖动,数字在改变.
	  数字改变时有一个注意点, 就是要显示%,它是一个特殊的符号,要用两个%%代表一个%

	3.拖动滑块的时候就是在上面画弧.
	 从最上面,按顺时针画,所以,它的起始角度是-90度.结束角度也是-90度
     也是从起始角度开始画,
     起始角度-90度, 看你下载进度是多少
     假如说你下载进度是100,就是1 * 360度
     也就是说这个进度占你360度多少分之一

     CGContextRef ctx = UIGraphicsGetCurrentContext();
     CGPoint center = CGPointMake(50, 50);
     CGFloat radius = rect.size.width * 0.5;
     CGFloat startA = -M_PI_2;
     CGFloat endA = -M_PI_2 + M_PI * 2 * progress;

     UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
     											          radius:radius
     											          startAngle:startA
     											          endAngle:endA
     											          clockwise:YES];
```
----
###`问题`


> 1. 要获得Progress的值,这个进度值没有, 所以要传进来才能画.弄一个成员变量,要在值改变的时候就要传进来.
    要拿到ProgressView才能够传进来,所以要拖线,拿到ProgressView
    
> 2. 所有都做好的, 发现没有画圆孤?为什么?
    问题:drawRect方法总共调用多少次?
        总共就调用一次, 第一次Progress为0,以后都不会执行了
    解决:每次传的时候,就要画一次,重写Progress方法
```objc
    -(void)setProgress:(CGFloat)progress{
        _progress = progress;
        手动调用drawRect方法, 让它重新绘制
        [self drawRect:self.bounds];
     }
```
     运行发现还是不画,为什么?
        原因:drawRect方法是不能手动调用,因为在drawRect方法中才能获取跟View相关联的上下文
        系统在调用DrawRect方法时,会自动帮你创建一个跟View相关联的上下文,并且传递给它.
        自己调用的,没有给drawRect方法传递上下文.所以在draw方法中拿不到上下文.
    解决办法:
        想要重绘,调用[self setNeedsDisplay];
        告诉系统重新绘制View,系统就会自动帮你调用drawRect方法,
        系统在调用drawRect方法,它会帮你创建上下文

