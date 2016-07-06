# Quarz2D演练
```objc
01-带有边框的图片裁剪

	具体实现思路:
	1.假设边框宽度为BorderW
	2.开启的图片上下文的尺寸就应该是原始图片的宽高分别加上两倍的BorderW,这样开启的目的是为了不让原始图片变形.
	3.在上下文上面添加一个圆形填充路径.位置从0,0点开始,宽高和上下文尺寸一样大.设置颜色为要设置的边框颜色.
	4.继续在上下文上面添加一个圆形路径,这个路径为裁剪路径.
	  它的x,y分别从BorderW这个点开始.宽度和高度分别和原始图片的宽高一样大.
	  将绘制的这个路径设为裁剪区域.
	5.把原始路径绘制到上下文当中.绘制的位置和是裁剪区域的位置相同,x,y分别从border开始绘制.
	6.从上下文状态当中取出图片.
	7.关闭上下文状态.

```
----
```objc
02-截屏

	截屏效果实现具体思路为:把UIView的东西绘制图片上下文当中,生成一张新的图片.
	注意:UIView上的东西是不能直接画到上下文当中的.
		UIView之所以能够显示是因为内部的一个层(layer),所以我要把层上的东西渲染到UIView上面的.
		怎样把图层当中的内容渲染到上下文当中?

		直接调用layer的renderInContext:方法
		renderInContext:带有一个参数, 就是要把图层上的内容渲染到哪个上下文.

		截屏具体实现代码为:
		    1.开启一个图片上下文
    		UIGraphicsBeginImageContextWithOptions(self.view.bounds.size, NO, 0);
		    获取当前的上下文.
		    CGContextRef ctx = UIGraphicsGetCurrentContext();
		    2.把控制器View的内容绘制上下文当中.
		    [self.view.layer renderInContext:ctx];
		    3.从上下文当中取出图片
		    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
		    4.关闭上下文.
		    UIGraphicsEndImageContext();

```
-------
```objc
03-图片擦除

	图片擦除思路.
	弄两个不同的图片.上面一张, 下面一张.
	添加手势,手指在上面移动,擦除图片.
	擦除前要先确定好擦除区域.
	假设擦除区域的宽高分别为30.
	那点当前的擦除范围应该是通过当前的手指所在的点来确定擦除的范围,位置.
	那么当前擦除区域的x应该是等于当前手指的x减去擦除范围的一半,同样,y也是当前手指的y减去高度的一半.

	有了擦除区域,要让图片办到擦除的效果,首先要把图片绘制到图片上下文当中, 在图片上下文当中进行擦除.
	之后再生成一张新的图片,把新生成的这一张图片设置为上部的图片.那么就可以通过透明的效果,看到下部的图片了.

	第一个参数, 要擦除哪一个上下文
	第二人参数,要擦除的区域.
	CGContextClearRect(ctx, rect);

	具体实现代码为:

	确定擦除的范围
    CGFloat rectWH = 30;
    获取手指的当前点.curP
    CGPoint curP = [pan locationInView:pan.view];
    CGFloat x = curP.x - rectWH * 0.5;
    CGFloat y = curP.y - rectWH * 0.5;
    CGRect rect = CGRectMake(x, y,rectWH, rectWH);

    先把图片绘制到上下文.
    UIGraphicsBeginImageContextWithOptions(self.imageView.bounds.size, NO, 0);
    获取当前的上下文
    CGContextRef ctx = UIGraphicsGetCurrentContext();
    把上面一张图片绘制到上下文.
    [self.imageView.layer renderInContext:ctx];
    再绘上下文当中图片进行擦除.
    CGContextClearRect(ctx, rect);
    生成一张新图片
    UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
    再把新的图片给重新负值
    self.imageView.image = newImage;
    关闭上下文.
    UIGraphicsEndImageContext();
```
-----
```objc
04-图片截屏

	图片截屏实现思路.
	手指在屏幕上移动的时
	添加一个半透明的UIView,
	然后开启一个上下文把UIView的frame设置成裁剪区域.把图片显示的图片绘制到上下文当中,生成一张新的图片
	再把生成的图片再赋值给原来的UImageView.

	具体实现步骤:
	1.给图片添加一个手势,监听手指在图片上的拖动,添加手势时要注意,UIImageView默认是不接事件的.
	  要把它设置成能够接收事件
	2.监听手指的移动.手指移动的时候添加一个UIView，
	  x,y就是起始点,也就是当前手指开始的点.
	  width即是x轴的偏移量,
	  高度即是Y轴的偏移量.
	  UIView的尺寸位置为CGrect(x,y,witdth,height);

	  计算代码为:
	  CGFloat offSetX = curP.x - self.beginP.x;
      CGFloat offsetY = curP.y - self.beginP.y;
      CGRect rect = CGRectMake(self.beginP.x, self.beginP.y, offSetX, offsetY);

      UIView之需要添加一次,所以给UIView设置成懒加载的形式,
      保证之有一个.每次移动的时候,只是修改UIView的frame.

    3.开启一个图片上下文,图片上下文的大小为原始图片的尺寸大小.使得整个屏幕都能够截屏.
      利用UIBezierPath设置一个矩形的裁剪区域.
      然后把这个路径设置为裁剪区域.
      把路径设为裁剪区域的方法为:
      [path addClip];

    4.把图片绘制到图片上下文当中
      由于是一个UIImageView上面的图片,所以也得需要渲染到上下文当中.
      要先获取当前的上下文,
      把UIImageView的layer渲染到当前的上下文当中.
      CGContextRef ctx = UIGraphicsGetCurrentContext();
      [self.imageV.layer renderInContext:ctx];

    5.取出新的图片,重新赋值图片.
      UIImage *newImage = UIGraphicsGetImageFromCurrentImageContext();
      self.imageV.image = newImage;

    6.关闭上下文,移除上面半透明的UIView
      UIGraphicsEndImageContext();
      [self.coverView removeFromSuperview];

```
-----
```objc
05-手势解锁

	分析界面,当手指在上面移动时,当移动到一个按钮范围内当中, 它会把按钮给成为选中的状态.
	并且把第一个选中的按钮当做一个线的起点,当手指移动到某个按钮上时,就会添加一根线到选中的那妞上.
	当手指松开时,所有按钮取消选中.所有的线都清空.

	实现思路:
		先判断点前手指在不在当前的按钮上.如果在按钮上,就把当前按钮成为选中状态.
		并且把当前选中的按钮添加到一个数组当中.如果当前按钮已经是选中状态,就不需要再添加到数组中了.
		每次移动时,都让它进行重绘.
		在绘图当中,遍历出所有的选中的按钮,
		判断数组当中的第一个无素,如果是第一个,那么就把它设为路径的起点.其它都在添加一根线到按钮的圆心.
		如果当前点不在按钮上.那么就记录住当前手指所在的点.直接从起点添加一根线到当前手指所在的点.


	实现步骤:
	1.搭建界面
	    界面是一个九宫格的布局.九宫格实现思路.
		先确定有多少列  cloum = 3;
		计算出每列之间的距离
		计算为: CGFloat margin = (当前View的宽度 - 列数 * 按钮的宽度) / 总列数 + 1
		每一列的X的值与它当前所在的行有关
		当前所在的列为:curColum = i % cloum
		每一行的Y的值与它当前所在的行有关.
		当前所在的行为:curRow = i / cloum

		每一个按钮的X值为, margin + 当前所在的列 * (按钮的宽度+ 每个按钮之间的间距)
		每一个按钮的Y值为 当前所在的行 * (按钮的宽度 + 每个按钮之间的距离)

		具体代码为:
		总列娄
    	int colum = 3;
    	每个按钮的宽高
    	CGFloat btnWH = 74;
    	每个按钮之间的距离
    	CGFloat margin = (self.bounds.size.width - colum * btnWH) / (colum + 1);
	    for(int i = 0; i < self.subviews.count; i++ ){
			当前所在的列
	        int curColum = i % colum;
	        当前所在的行
	        int curRow = i / colum;
	        CGFloat x = margin + (btnWH + margin) * curColum;
	        CGFloat y = (btnWH + margin) * curRow;
	        取出所有的子控件
	        UIButton *btn = self.subviews[i];
	        btn.frame = CGRectMake(x, y, btnWH, btnWH);
	    }

	 2.监听手指在上面的点击,移动,松开都需要做操作.

	 	2.1在手指开始点击屏幕时,如果当前手指所在的点在按钮上, 那就让按钮成为选中状态.
			所以要遍历出所有的按钮,判断当前手指所在的点在不在按钮上,
			如何判断当前点在不在按钮上?
			当前方法就是判断一个点在不在某一个区域,如果在的话会返回Yes,不在的话,返回NO.
			CGRectContainsPoint(btn.frame, point)

			在手指点击屏幕的时候,要做的事分别有
			1.获取当前手指所在的点.
				UITouch *touch = [touches anyObject];
				CGPoint curP =  [touch locationInView:self];
			2.判断当前点在不在按钮上.
				 for (UIButton *btn in self.subviews) {
        			if (CGRectContainsPoint(btn.frame, point)) {
          				  return btn;
        			}
			     }
	   		3.如果当前点在按钮上,并且当前按钮不是选中的状态.
	   		  那么把当前的按钮成为选中状态.
	   		  并且把当前的按钮添加到数组当中.


	   	2.2 当手指在移动的时也需要判断.
			  判断当前点在按钮上,并且当前按钮不是选中的状态.
	   		  那么把当前的按钮成为选中状态.
	   		  并且把当前的按钮添加到数组当中.
			 在移动的时候做重绘的工作.

	    2.3 当手指离开屏幕时.
	        取出所有的选中按钮,把所有选中按钮取消选中状态.
	        清空选中按钮的数组.
	        绘重绘的工作.


	 3. 在绘图方法当中.
	 	创建路径
	 	遍历出有的选中按钮.如果是第一个按钮,把第一个按钮的中心点当做是路径的起点.
	 	其它按钮都直接添加一条线,到该按钮的中心.

	 	遍历完所有的选中按钮后.
	 	最后添加一条线到当前手指所在的点.
```
```objc
06-画板

	画板界面分析.
	顶部是一个工具栏.有清屏,撤销,橡皮擦,照片功能.最右部是一个保存按钮
	中间部分为画板区域
	最下部拖动滑竿能够改变画笔的粗线.可以选颜色.

	1.界面搭建
		最上部为一个ToolBar,往ToolBar拖些item,使用ToolBar的好处.里面按钮的位置不需要我们再去管理.
		给最上部的工具栏做自动布局.离父控件左,上,右都为0,保存工具条的高度不度

		拖一个UIView当前下部的View.在下部的View当中拖累三个按钮,设置每一个按钮的背景颜色.
		点击每一按钮时办到设置画笔的颜色.
		其中三个按钮只间的间距始终保存等,每一个按钮的宽度和高度都相等.通过自动布局的方式办到.

		先把这个UIView的自动布局设好, 让其左,右,下都是0,高度固定.

		自动布局设置为:第一个按钮高度固定,与左,右,下都保存20个间距.
		第二个按钮与第一个按钮,高度,宽度,centerY都相等.与右边有20个间距.
		第三个按钮也是第一个按钮的高度,宽度,centerY都相等.与右边有20个间距,最右边也保存20个间距.

		最后是中间的画板区域,画板区域只需要上距离上下左右都为0即可.

	2.实现画板功能.

	  当手指移动的时候,在中间的画板区域当中进行绘制.由于每一个路径都有不同的状态.所以我们就不能用一条路径来做.
	  所以要弄一个数组记录住每一条路径.
	  实现画板功能.
	  1.监听手指在屏幕上的状态.在开始点击屏幕的时候,创建一个路径.并把手指当前的点设为路径的起点.

	    弄一个成员属性记录当前绘制的路径.并把当前的路径添加到路径数组当中.

	  2.当手指在移动的时候,用当前的路径添加一根线到当前手指所在的点.然后进行重绘.
	  3.在绘图方法当中.取出所有的路径.把所有的路径给绘制出来.

	3.设置路径属性.
		提供属性方法.

		清屏功能:删除所有路径进行重绘

		撤销功能:删除最后一条路径,进行重绘

		设置线宽:
				由于每一条线宽度都不样.所以要在开始创建路径的时,就要添加一个成员属性,设置一个默认值.
				在把当前路径添加到路径数组之前,设置好线的宽度.然后重写线宽属性方法.
				下一次再创建路径时,线的宽度就是当前设置的宽度.
		设置线的颜色:
				同样,由于每一条线的颜色也不一样.也需要记录住每一条路径的颜色.
				由于UIBezierPath没有给我们直接提供设置颜色的属性.我们可以自定义一个UIBezierPath.
				创建一个MyBezierPath类,继承UIBezierPath,在该类中添加一个颜色的属性.
				在创建路径的时候,直接使用自己定义的路径.设置路径默认的一个颜色.方法给设置线宽一样.
				在绘图过程中, 取出来的都是MyBezierPath,把MyBezierPath的颜色设置路径的颜色.

		橡皮擦功能:橡皮擦功能其实就是把路径的颜色设为白色.


	4.保存绘制的图片到相册.
		保存相册的思路:就是把绘制的在View上的内容生成一张图片,保存到系统相册当中.
		具体步骤:
		开启一个跟View相同大小的图片上下文.
		把View的layer上面内容渲染到上下文当中.
		生成一张图片,把图片保存到上下文.
		关闭上下文.
		如何把一张图片保存到上下文?
		调用方法:
		参数说明:
		第一个参数:要写入到相册的图片.
		第二个参数:哪个对象坚听写入完成时的状态.
		第三个参数:图片保存完成时调用的方法
		UIImageWriteToSavedPhotosAlbum(newImage,
		 									 self,
		 @selector(image:didFinishSavingWithError: contextInfo:),nil);
		注意:图片保存完成时调用的方法必须得是image:didFinishSavingWithError: contextInfo:


	 5.选择图片.
	 	点击图片时弹出系统的相册.
	 	如果弹出系统的相册?
	 	使用UIImagePickerController控件器Modal出它来.
	 	UIImagePickerController *pick = [[UIImagePickerController alloc] init];

	 	设置照片的来源
	  	pick.sourceType =  UIImagePickerControllerSourceTypeSavedPhotosAlbum;

	  	设置代码,监听选择图片,UIImagePickerController比较特殊,它需要遵守两个协议
	  	<UINavigationControllerDelegate,UIImagePickerControllerDelegate>
	  	pick.delegate = self;

	  	modal出控件器
	  	[self presentViewController:pick animated:YES completion:nil];

	  	注意没有实现代码方法时,选择一张照片会自动的dismiss掉相册控制器.但是设置代码后,就得要自己去dismiss了


	  	实现代理方法.
	  	选择的照片就在这个方法第二个参数当中, 它是一个字典
	  	-(void)imagePickerController:(nonnull UIImagePickerController *)picker
	  	 didFinishPickingMediaWithInfo:(nonnull NSDictionary<NSString *,id> *)info{

			获取当前选中的图片.通过UIImagePickerControllerOriginalImage就能获取.
			UIImage *image = info[UIImagePickerControllerOriginalImage];

		}

		获取完图片后.图片是能够缩放,平移的,因此获取完图片后,是在画板板View上面添加了一个UIView.
		只有UIView才能做平移,缩放,旋转等操作.
		因此做法为.在图片选择完毕后,添加一个和画板View相同大小的UIView,这个UIView内部有一个UIImageView.
		对这个UIImageView进行一些手势操作.操作完成时.长按图片,把View的内容截屏,生成一张图片.
		把新生成的图片绘制到画板上面.

	6.绘制图片.
		在画板View当中提供一个UImage属性,供外界传递.重写属性的set方法,每次传递图片时进行重绘
		画图片也有有序的,所以要把图片也添加到路径数组当中.
		在绘图片过过程当中,如果发现取出来的是一个图片类型.那就直接图片绘制到上下文当中.

		具体实现代码为:
		-(void)drawRect:(CGRect)rect{

		    for (DrawPath *path in self.pathArray) {

        		if ([path isKindOfClass:[UIImage class]]) {

         				UIImage *image = (UIImage *)path;
            			[image drawInRect:rect];

        			}else{
            			[path.lineColor set];
            			[path stroke];
        			}

    			}
    		}
```
