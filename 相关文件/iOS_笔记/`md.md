# 02 - 基本线条绘制

```objc
	1.DrawRect方法作用?什么时候调用.
		DrawRect作用:在这个方法当中绘图.只有在这个方法当中才能取得跟View相关联的上下文.
		DrawRect是系统自己调用的, 它是当View显示的时候自动调用.

	2.画线(基本步骤描述)
		2.1获取跟View相关联的上下文
		 CGContextRef ctx = UIGraphicsGetCurrentContext();

		2.2绘制路径
		UIBezierPath *path = [UIBezierPath bezierPath];

		2.2.1设置起点
	    [path moveToPoint:CGPointMake(10, 125)];

	    2.2.2添加一根线到某个点
	    [path addLineToPointa:CGPointMake(200, 125)];

		2.3把路径添加到上下文
		 CGContextAddPath(ctx,path.CGPath);

		2.4把上下文的内容渲染到View上面.
		 CGContextStrokePath(ctx);

	 3. 想要再添加一根线怎么办?
	 	第一种方法:重新设置起点,添加一根线到某个点,一个UIBezierPath路径上面可以有多条线.
	 	第二种方法:直接在原来的基础上添加线.把上一条的终点当做下一条线的起点.添加一根线到某个点
	 			  直接在下面addLineToPoint:

	 4.怎么样设置线的宽度,颜色,样式?
	 	设置这些样式,我们称为是修改图形上下文的状态.
	 	设置线宽:CGContextSetLineWidth(ctx, 20);
	 	设置线段的连接样式: CGContextSetLineJoin(ctx, kCGLineJoinRound);
	 	添加顶角样式:CGContextSetLineCap(ctx, kCGLineCapRound);
	 	设置线的颜色: [[UIColor redColor] setStroke];

	 5.如何画曲线?

	 	画曲线方法比较特殊需要一个控制点来决定曲线的弯曲程度.画曲线方法为:
	 	先设置一个曲线的起点
	 	[path moveToPoint:CGPointMake(10, 125)];
	 	再添加到个点到曲线的终点.同时还须要一个controlPoint(控件点决定曲线弯曲的方法程序)
        [path addQuadCurveToPoint:CGPointMake(240, 125) controlPoint:CGPointMake(125, 10)];

     6.如何画矩形,圆角矩形?

     	画矩形直接利用UIBezierPath给我们封装好的路径方法
     	(x,y)点决定了矩形左上角的点在哪个位置
     	(width,height)是矩形的宽度高度
     	bezierPathWithOvalInRect:CGRectMake(x, y, width, height)

     	圆角矩形的画法多了一个参数,cornerRadius
     	cornerRadius它是矩形的圆角半径.
     	通过圆角矩形可以画一个圆.当矩形是正方形的时候,把圆角半径设为宽度的一半,就是一个圆.
     	bezierPathWithRoundedRect:CGRectMake(10, 100, 50, 50) cornerRadius:10

     7.如果画椭圆,圆?

     	画椭圆的方法为:
     	前两个参数分别代码圆的圆心,后面两个参数分别代表圆的宽度,与高度.
     	宽高都相等时,画的是一个正圆, 不相等时画的是一个椭圆
     	bezierPathWithOvalInRect:CGRectMake(10, 100, 50, 50)

	 8.如何利用UIKit封装的上下文进行画图?
	 	直接来个:[path stroke]就可以了.
	 	它底层的实现,就是获取上下文,拼接路径,把路径添加到上下文,渲染到View

	 9.如何画圆弧?

	 	 首先要确定圆才能确定圆弧,圆孤它就圆上的一个角度嘛

	 	Center:圆心
         radius:圆的半径
         startAngle:起始角度
         endAngle:终点角度
         clockwise:Yes顺时针,No逆时针

         注意:startAngle角度的位置是从圆的最右侧为0度.

	 	 UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:CGPointMake(125, 125)
	 	 									                  radius:100
	 	 									                  startAngle:0
	 	 									                  endAngle:M_PI * 2
	 	 									                  clockwise:YES];


	 10.如果画扇形.
	 	画扇形的方法为:先画一个圆孤再添加一个一根线到圆心,然后关闭路径.
	 				 关闭路径就会自动从路径的终点到路径的起点封闭起下
	 	用填充的话,它会默认做一个封闭路径,从路径的终点到起点.
	 	[path fill];
```
