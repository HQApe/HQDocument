<center>CTMediator问题总结</center>

## 一、Swift命名空间的问题
在Swift或OC混编的组件化过程中通过以下代码来动态获取类是行不通的，是取不到的：

OC:
```objective-C
NSString *targetClassString = [NSString stringWithFormat:@"Target_%@", targetName];
Class targetClass = NSClassFromString(targetClassString);
NSObject *target = [[targetClass alloc] init];
```
或Swift：
```Swift
let targetClass = NSClassFromString("MeasureHouseWaitVC") as! UIViewController.Type;
let target:UIViewController? = targetClass.init()
```
应该要使用以下方式获取：

OC:
```objective-C
NSString *targetClassString = [NSString stringWithFormat:@"项目空间.Target_%@", targetName];
Class targetClass = NSClassFromString(targetClassString);
NSObject *target = [[targetClass alloc] init];
```
或Swift：
```Swift
let targetClass = NSClassFromString("TMeasureHouse.MeasureHouseWaitVC") as! UIViewController.Type;
let target:UIViewController? = targetClass.init()
```

## 二、解决思路及方案
那么如何获取项目空间？即组件被封装成pod库了，如何获取组件的命名空间来动态获取类？

以下提供几个解决方案：

1、重写CTMediator中动态获取类的代码，通过传入完整命名空间来动态获取类。经验证，OC和Swift中的OC对象都不需要命名空间，swif必须要命名空间。

```objective-C
//targetClassName是完整的命名空间，例如"App.Helloworld"、"ComponentA.TargetA"，如果OC工程下没有写全也没关系，但Swift务必写全

- (id)performTargetClass:(NSString *)targetClassName action:(NSString *)actionName params:(NSDictionary *)params shouldCacheTarget:(BOOL)shouldCacheTarget
{
     
    NSString *actionString = [NSString stringWithFormat:@"Action_%@:", actionName];
    Class targetClass;
    NSObject *target = self.cachedTarget[targetClassName];
    if (target == nil) {
        targetClass = NSClassFromString(targetClassName);
        target = [[targetClass alloc] init];
    }
    //若取不到相关类，可能是OC对象。OC中不需要命名空间，因此可以再加上以下代码：
    if (target == nil) {
        NSArray *tempArr = [targetClassString componentsSeparatedByString:@"."];
        if (tempArr.count) {
            targetClass = NSClassFromString(tempArr.lastObject);
            target = [[targetClass alloc] init];
        }
    }

    .....
}
```
以及
```objective-C
- (void)releaseCachedTargetWithTargetClass:(NSString *)targetClassName
{
    [self.cachedTarget removeObjectForKey:targetClassName];
}

```

本工程案例已对CTMediator做优化处理。
