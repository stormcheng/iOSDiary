# 约束


## 使用代码实现Autolayout的方法1
- 创建约束

```objc
* view1 ：要约束的控件
* attr1 ：约束的类型（做怎样的约束）
* relation ：与参照控件之间的关系
* view2 ：参照的控件
* attr2 ：约束的类型（做怎样的约束）
* multiplier ：乘数
* c ：常量
+(id)constraintWithItem:(id)view1 attribute:(NSLayoutAttribute)attr1 relatedBy:(NSLayoutRelation)relation 
toItem:(id)view2 attribute:(NSLayoutAttribute)attr2 multiplier:(CGFloat)multiplier constant:(CGFloat)c;

```

- 添加约束

```objc
- (void)addConstraint:(NSLayoutConstraint *)constraint;
- (void)addConstraints:(NSArray *)constraints;
```

- 注意
    - 一定要在拥有父控件之后再添加约束
    - 关闭Autoresizing功能
    ```objc
    view.translatesAutoresizingMaskIntoConstraints = NO;
    ```

## 使用代码实现Autolayout的方法2 - VFL
- 使用VFL创建约束数组
- H:[cancelButton(72)]-12-[acceptButton(50)]
- V:[redBox][yellowBox(==redBox)]
- H:|-10-[Find]-[FindNext]-[FindField(>=20)]-|


```objc
+ (NSArray *)constraintsWithVisualFormat:(NSString *)format
options:(NSLayoutFormatOptions)opts
metrics:(NSDictionary *)metrics
views:(NSDictionary *)views;
* format ：VFL语句
* opts ：约束类型
* metrics ：VFL语句中用到的具体数值
* views ：VFL语句中用到的控件
```

- 使用下面的宏来自动生成views和metrics参数

```objc
NSDictionaryOfVariableBindings(...)
```

## 使用代码实现Autolayout的方法3 - Masonry(第三方类库)

Masonry的执行流程
```objc

        mas_makeConstraints执行流程:
        1.创建约束制造者MASConstraintMaker,绑定控件,生成了一个保存所有约束的数组
        2.执行mas_makeConstraints传入进行的block
        3.让约束制造者安装约束
            *   1.清空之前的所有约束
            *   2.遍历约束数组,一个一个安装


```
```
    // 设置约束,一定要先把view添加上去,才能设置约束
    [redView mas_makeConstraints:^(MASConstraintMaker *make) {
        // 链式编程思想特点:方法返回值必须要方法调用者
        // block:把需要操作的值当做block参数,block也需要返回值,就是方法调用者
       // 设置约束
        // 给make添加left,top约束,调用equalTo给这两个约束赋值
        make.left.top.equalTo(@10);
        make.right.bottom.equalTo(@-10);

    }];

```

- 使用步骤
    - 添加Masonry文件夹的所有源代码到项目中
    - 添加2个宏、导入主头文件
    ```objc
    // 只要添加了这个宏，就不用带mas_前缀
    #define MAS_SHORTHAND
// 只要添加了这个宏，equalTo就等价于mas_equalTo
#define MAS_SHORTHAND_GLOBALS
// 这个头文件一定要放在上面两个宏的后面
#import "Masonry.h"
    ```

- 添加约束的方法

```objc
// 这个方法只会添加新的约束
 [view makeConstraints:^(MASConstraintMaker *make) {

 }];

// 这个方法会将以前的所有约束删掉，添加新的约束
 [view remakeConstraints:^(MASConstraintMaker *make) {

 }];

 // 这个方法将会覆盖以前的某些特定的约束
 [view updateConstraints:^(MASConstraintMaker *make) {

 }];
```

- 约束的类型
```objc
1.尺寸：width\height\size
2.边界：left\leading\right\trailing\top\bottom
3.中心点：center\centerX\centerY
4.边界：edges
```

