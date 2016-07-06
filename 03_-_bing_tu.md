# 03 - 饼图

# 04-画饼图

```objc
        //第一步, 获取上下文
        //第二步,拼接路径 ,绘制第一个扇形
         //获取上下文
         CGContextRef ctx =  UIGraphicsGetCurrentContext();
         CGPoint center = CGPointMake(125, 125);
         CGFloat radius = 100;
         CGFloat startA = 0;
         CGFloat endA = 0;
         CGFloat angle = 25 / 100.0 * M_PI * 2;
         endA = startA + angle;
         //拼接路径
         UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
         										              radius:radius
         										              startAngle:startA
         										              endAngle:endA
         										              clockwise:YES];
         //添加一根线到圆心
         [path addLineToPoint:center];
         //把路径添加到上下文
         CGContextAddPath(ctx, path.CGPath);
         //把上下文渲染到View
         CGContextFillPath(ctx);

        #warning: CGFloat angle = 25 / 100.0 * M_PI * 2;
        //100.0一定要加一个.0

        //绘制第二个扇形
        //一样的, 把路径描述第二个扇形就好了
        //直接来个path =
        //让Path指针重新指向一个新的对象.也就是把指针重复利用了
        //之前的那个对象已经用完了,已经添加到了上下文当中了.

        //第二个扇形
        startA = endA;
        angle = 25 / 100.0 * M_PI * 2;
        endA = startA + angle;
        path = [UIBezierPath bezierPathWithArcCenter:center
         										 radius:radius
         										 startAngle:startA
         										 endAngle:endA
         										 clockwise:YES];
         [path addLineToPoint:center];
         //把第二个路径添加到上下文
         CGContextAddPath(ctx, path.CGPath);
         //把上下文渲染到View
         CGContextFillPath(ctx);


        //添加第二个扇形之后, 发现它们的颜色都一样,想要修改它的颜色
        //在下面再写一个
        [[UIColor greenColor] set];
        //下面的一个颜色把之前的东西给覆盖了.
        //解决办法, 让它渲染两次

        //第三个扇形,把第二个拷贝一下就好了

```
-----
```objc

        //抽取代码:
            //假设他给一组数据
            NSArray datas =  @[@25,@25,@50];
            把数组便利出来


         NSArray *datas =  @[@25,@25,@50];

         CGPoint center = CGPointMake(125, 125);
         CGFloat radius = 100;
         CGFloat startA = 0;
         CGFloat angle = 0;
         CGFloat endA = 0;

         for (NSNumber *number in datas) {

         startA = endA;
         angle = number.intValue / 100.0 * M_PI * 2;
         endA = startA + angle;

         描述路径
         UIBezierPath *path = [UIBezierPath bezierPathWithArcCenter:center
         									  radius:radius
         									  startAngle:startA
         									  endAngle:endA
         									  clockwise:YES];

         [path addLineToPoint:center];
         [[self randomColor] set];
         [path fill];

         }

         - (UIColor *)randomColor{
         CGFloat r = arc4random_uniform(256)/ 255.0;
         CGFloat g = arc4random_uniform(256)/ 255.0;
         CGFloat b = arc4random_uniform(256)/ 255.0;
         return [UIColor colorWithRed:r green:g blue:b alpha:1];

         }
```
