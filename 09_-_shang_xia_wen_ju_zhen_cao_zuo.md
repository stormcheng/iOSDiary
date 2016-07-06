# 09 - 上下文矩阵操作


>  上下文的矩阵操作其实就是修改上下文的形变    

```objc
	  //主要有以下几种
	  1.平移
	  CGContextTranslateCTM(ctx, 100, 100);
	  2.旋转
	  CGContextRotateCTM(ctx, M_2_PI);
	  3.缩放
	  CGContextScaleCTM(ctx, 0.5, 0.5);
	  #注意:上下文操作必须得要在添加路径之前去设置
```
