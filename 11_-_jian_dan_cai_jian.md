# 11 - 简单裁剪

```objc
	裁剪图片思路.
    开启一个图片上下文.
    上下文的大小和原始图片保持一样.以免图片被拉伸缩放.
    在上下文的上面添加一个圆形裁剪区域.圆形裁剪区域的半径大小和图片的宽度一样大.
    把要裁剪的图片绘制到图片上下文当中.
    从上下文当中取出图片.
    关闭上下文.

    1.如何设置圆形路径?
    	 UIBezierPath *path = [UIBezierPath bezierPathWithOvalInRect:
    	 					CGRectMake(0, 0, image.size.width, image.size.width)];


    2.如何把一个路径设为裁剪区域?
    	[path addClip];
```
