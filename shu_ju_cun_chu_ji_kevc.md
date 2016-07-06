#数据存储及KVC简介
## 1.plist存储
1.plist存储,生成一个plist文件.

2.plist不是数组就是字典,plist存储就是用来存储字典或者数组.

`注意:Plist不能存储自定义对象`

3.获取应用沙盒中Caches文件路径
```objc

    // directory:获取哪个文件夹
    // domainMask:在哪个范围内搜索,NSUserDomainMask:表示在用户的手机上查找
    // expandTilde:是否展开全路径 YES:表示展开全路径 NO:不会展开全路径,会把应用沙盒的路径用波浪号(~)代替

    // 获取到Caches文件夹路径
    NSString *cachePath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];
```
4.读取plist,之前是什么类型存储的,读取也是什么

## 2.偏好设置存储
```objc
    以字典的形式进行偏好设置,用法跟字典.
    偏好设置好处:  1.不需要关心文件名
                  2.快速进行键值对存储
                  3.直接存储基本数据类型

    //1. 获取NSUserDefaults单例对象
    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

    //2. 设置值
    [defaults setObject:@"123" forKey:@"num"];
    [defaults setBool:YES forKey:@"isOn"];
    //立即存储
    [defaults synchronize];

    //3. 获取值
    NSString *num = [[NSUserDefaults standardUserDefaults] objectForKey:@"num"];
    BOOL ison =  [[NSUserDefaults standardUserDefaults] boolForKey:@"isOn"];
```

## 3.归档
1.NSKeyedArchiver专门用来做自定义对象归档

```objc
// 什么时候调用:当一个对象要归档的时候就会调用这个方法归档
// 作用:告诉苹果当前对象中哪些属性需要归档
- (void)encodeWithCoder:(NSCoder *)aCoder
{
    [aCoder encodeObject:_name forKey:@"name"];
    [aCoder encodeInt:_age forKey:@"age"];
}

// 什么时候调用:当一个对象要解档的时候就会调用这个方法解档
// 作用:告诉苹果当前对象中哪些属性需要解档
// initWithCoder什么时候调用:只要解析一个文件的时候就会调用
- (id)initWithCoder:(NSCoder *)aDecoder
{
    #warning  [super initWithCoder]
    #warning  父类遵守<NSCoding>协议，解档父类属性
    if (self = [super init]) {
        // 解档
        // 注意一定要记得给成员属性赋值
      _name = [aDecoder decodeObjectForKey:@"name"];
      _age = [aDecoder decodeIntForKey:@"age"];
    }
    return self;
}
```
```objc
// 创建自定义对象
    Person *p = [[Person alloc] init];
    p.age = 18;
    p.name= @"chan";

    // 获取caches文件夹
    NSString *cachesPath = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES)[0];

    // 拼接文件名
    NSString *filePath = [cachesPath stringByAppendingPathComponent:@"person.data"];

    //归档
    [NSKeyedArchiver archiveRootObject:p toFile:filePath];

```
# KVC
```objc
// setValuesForKeysWithDictionary底层实现

//  利用KVC字典转模型,
    [flag setValuesForKeysWithDictionary:dict];

    // 1.遍历字典中的所有key
    [dict enumerateKeysAndObjectsUsingBlock:^(id key, id obj, BOOL *stop) {
        // 2.给模型的属性赋值,利用KVC,把字典中的key当做模型的属性名使用,字典中的值传递给模型的属性.
        [flag setValue:obj forKey:key];
        // name -> icon
        // KeyPath:模型中的属性名
        // 属性的值

//        [flag setValue:dict[@"name"] forKey:@"name"];
//        [flag setValue:dict[@"icon"] forKey:@"icon"];
    }];



// setValue:forKey:底层实现
// 给模型中的icon属性赋值
// [flag setValue:dict[@"icon"] forKey:@"icon"];

// 1.首先去寻找模型中有木有setIcon:方法,直接调用setIcon:方法,[flag setIcon:dict[@"icon"]]
// 2.接着寻找模型中有没有icon的属性名,如果有,就直接赋值 icon = dict[@"icon"]

// 3.接着寻找模型中有没有_icon的属性名,如果有,就直接赋值 _icon = dict[@"icon"]

// 4.找不到,直接报错,setValue:forUndefinedKey:

```
