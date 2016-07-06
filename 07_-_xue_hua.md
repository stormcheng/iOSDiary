# 07 - 雪花

```objc
	1.定时器雪花整体思路:
		先在控制器View面绘制一个雪花.
		在View加载完毕后,添加一个定时器.
		在定时器方法当中调用得绘方法.
		在绘图方法当不段的去修改雪花的Y值.
		当雪花的Y值超过屏幕的高度时,让雪花的Y值重新设为0.从最顶部开始.

	2.添加定时器实现方案
		第一种采用NSTime
		第二种采用CADisplayLink
		最终采用CADisplayLink方案.

		2.1为什么采用CADisplayLink方案不用NSTime?

		   首先要了解setNeedsDisplay
		   setNeedsDisplay底层会调用DrawRect方法重绘.
		   但是它不是立马就进行重绘.它仅仅是设置了一个重绘标志,等到下一次屏幕刷新的时候才会调用DrawRect方法.

		   如果使用NSTime的话,假设是0.01调用一次重绘.假设屏幕0.02秒的时候它才刷新一次.中间就会等0.01秒.
		   也就是每次都会等0.01秒这样累加上去.让变的越来越卡顿.

		   使用CADisplayLink时,它的定时器方法就是屏幕每次刷新的时候就会调用(通常屏幕一秒钟刷新60次)
		   它和setNeedsDisplay调用DrawRect方法的时机正好吻合,不会出间等待间隔.不会出现屏幕卡顿现象.

	    2.2如何使用CADisplayLink添加定时器?
	    	Target:哪个对象要监听方法.
	    	selector:监听的方法名称.
	    	CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self
	    							selector:@selector(setNeedsDisplay)];
	    	#想要让CADisplayLink工作,必须得要把它添加到主运行循环.
	    	只要添加到主运行循环, 跟模式没有关系
	    	[link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];

	  3.具体实现代码如下:

			 -(void)awakeFromNib{
			    CADisplayLink *link = [CADisplayLink displayLinkWithTarget:self
			    					    selector:@selector(setNeedsDisplay)];
			    [link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];

			  }


			 - (void)drawRect:(CGRect)rect {
			    if(_snowY > rect.size.height){
			        _snowY = 0;
			    }
			    UIImage *image = [UIImage imageNamed:@"雪花"];
			    [image drawAtPoint:CGPointMake(0, _snowY)];
			    _snowY += 10;
			 }
```
