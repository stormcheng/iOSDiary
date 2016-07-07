##core data

![](images/DAC067AF-42EE-4B21-BE49-B6891E78611F.png)

- （coreData的详解网址必看->）
-
http://www.cocoachina.com/ios/20130911/6981.html

##事件的产生和传递
![](images/13DA4D04-04F7-4BA5-A938-1DC78B444147.png)

##hitTest方法的内部实现原理（递归）
![](images/4A829737-C3EB-4641-A3A4-A17CAAA26B98.png)

##证书和授权文件

 - 这意味着使用这个Provisioning Profile打包程序必须拥有相应的证书，并且是将App ID对应的程序运行到Devices中包含的设备上去。
 -
 - http://blog.sina.com.cn/s/blog_8d1bc23f0102vtzo.html

![](images/3F6ADE95-070C-4764-9898-A1402B1D0BF1.png)
![](images/4EED0035-58E0-4D79-8309-61EAD6F9EA69.png)

##富文本

- ios开发中我们避免不了要自定义一些控件,修改文字的颜色,下划线,删除线等等,所以ios提供了两个类供我们使用NSAttributedstring和NSMutableAttributedString,被叫做富文本。
- 只要是可以显示文字的控件大都有这个属性,例如：UIButton,UILabel,UITextView,UITextField等等.

```
NSFontAttributeName                设置字体属性，默认值：字体：Helvetica(Neue) 字号：12
NSForegroundColorAttributeNam      设置字体颜色，取值为 UIColor对象，默认值为黑色
NSBackgroundColorAttributeName     设置字体所在区域背景颜色，取值为 UIColor对象，默认值为nil, 透明色
NSLigatureAttributeName            设置连体属性，取值为NSNumber 对象(整数)，0 表示没有连体字符，1 表示使用默认的连体字符
NSKernAttributeName                设定字符间距，取值为 NSNumber 对象（整数），正值间距加宽，负值间距变窄
NSStrikethroughStyleAttributeName  设置删除线，取值为 NSNumber 对象（整数）
NSStrikethroughColorAttributeName  设置删除线颜色，取值为 UIColor 对象，默认值为黑色
NSUnderlineStyleAttributeName      设置下划线，取值为 NSNumber 对象（整数），枚举常量 NSUnderlineStyle中的值，与删除线类似
NSUnderlineColorAttributeName      设置下划线颜色，取值为 UIColor 对象，默认值为黑色
NSStrokeWidthAttributeName         设置笔画宽度，取值为 NSNumber 对象（整数），负值填充效果，正值中空效果
NSStrokeColorAttributeName         填充部分颜色，不是字体颜色，取值为 UIColor 对象
NSShadowAttributeName              设置阴影属性，取值为 NSShadow 对象
NSTextEffectAttributeName          设置文本特殊效果，取值为 NSString 对象，目前只有图版印刷效果可用：
NSBaselineOffsetAttributeName      设置基线偏移值，取值为 NSNumber （float）,正值上偏，负值下偏
NSObliquenessAttributeName         设置字形倾斜度，取值为 NSNumber （float）,正值右倾，负值左倾
NSExpansionAttributeName           设置文本横向拉伸属性，取值为 NSNumber （float）,正值横向拉伸文本，负值横向压缩文本
NSWritingDirectionAttributeName    设置文字书写方向，从左向右书写或者从右向左书写
NSVerticalGlyphFormAttributeName   设置文字排版方向，取值为 NSNumber 对象(整数)，0 表示横排文本，1 表示竖排文本
NSLinkAttributeName                设置链接属性，点击后调用浏览器打开指定URL地址
NSAttachmentAttributeName          设置文本附件,取值为NSTextAttachment对象,常用于文字图片混排
NSParagraphStyleAttributeName      设置文本段落排版格式，取值为 NSParagraphStyle 对象
```

##重绘(位图上下文)
```
#pragma mark 对图片尺寸进行压缩--
- (UIImage*)imageWithImage:(UIImage*)image scaledToSize:(CGSize)newSize
{
    // Create a graphics image context
    UIGraphicsBeginImageContext(newSize);

    // Tell the old image to draw in this new context, with the desired
    // new size
    [image drawInRect:CGRectMake(0,0,newSize.width,newSize.height)];

    // Get the new image from the context
    UIImage* newImage = UIGraphicsGetImageFromCurrentImageContext();

    // End the context
    UIGraphicsEndImageContext();

    // Return the new image.
    return newImage;
}
```

##添加手势后的事件链传递
 - 手势的传递级别比touch方法高，优先寻找手势，没有手势再找touch方法。

![](images/3AEC5117-691B-42E0-9BBC-3947303E2F13.png)
