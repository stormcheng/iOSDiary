#01-iOS9新特性之关键字
* iOS9新出的关键字:用来修饰属性,或者方法的参数,方法的返回值
* 好处:

    * 1.迎合swift

    * 2.提高我们开发人员开发规范,减少程序员之间交流

* 注意:
    * iOS9新出关键字nonnull,nullable,null_resettable,_Null_unspecified只能`修饰对象,不能修饰基本数据类型`.
    *
* `nullable`作用:表示可以为空

```objc
 nullable书写规范:
 // 方式一:
 @property (nonatomic, strong, nullable) NSString *name;
 // 方式二:
 @property (nonatomic, strong) NSString *_Nullable name;
 // 方式三:
 @property (nonatomic, strong) NSString *__nullable name;

```
* `nonnull`作用:不能为空

```objc
nonnull: non:非 null:空

书写格式:
 @property (nonatomic, strong, nonnull) NSString *icon;

 @property (nonatomic, strong) NSString * _Nonnull icon;

 @property (nonatomic, strong) NSString * __nonnull icon;


```

* 在`NS_ASSUME_NONNULL_BEGIN`和`NS_ASSUME_NONNULL_END`之间,定义的所有对象属性和方法默认都是nonnull

* `null_resettable`作用: get:不能返回为空, set可以为空

```
// 书写方式:
@property (nonatomic, strong, null_resettable) NSString *name;

 // 注意;如果使用null_resettable,必须 重写get方法或者set方法,处理传递的值为空的情况


```

* `_Null_unspecified`:不确定是否为空

```objc
书写方式只有这种
    方式一
    @property (nonatomic, strong) NSString *_Null_unspecified name;
    方式二
    @property (nonatomic, strong) NSString *__null_unspecified name;
```

