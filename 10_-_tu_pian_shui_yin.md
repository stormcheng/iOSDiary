# 10 - 图片水印


```objc
	 给图片水印的目的:
	 告诉别人图片的来源.
	 防止别人盗用图片.打广告.

	 添加水印它最终是生成了一个新的图片.
	 生成图片要用到了图片上下文.不需要再去自定义View,
	 之前一直在自定义View,是因为要拿跟View相关联的上下文.
	 跟View相关联的上下文是系统自动帮我们创建的,所以不需要我们自己手动创建,
	 但是图片上下文需要我们自己去手动创建.还需要我们自己手动去关闭.

	 实现水印效果的思路:
	 开启一个和原始图片一样的图片上下文.
	 把原始图片先绘制到图片上下文.
	 再把要添加的水印(文字,logo)等绘制到图片上下文.
	 最后从上下文中取出一张图片.
	 关闭图片上下文.

	 1.如何开启一个图片上下文?
	     size:开启多大的上文
	     opaque:不透明度
	     scale:缩放上下文.
	     UIGraphicsBeginImageContextWithOptions(image.size, YES, 0);

     2.如何从图片上下文当中生成一张图片?
     	UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();

     3.如何关闭上下文?
     	UIGraphicsEndImageContext();
```
