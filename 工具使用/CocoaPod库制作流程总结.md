# <center>CocoaPod库制作流程总结</center>
[TOC]
## 一、流程：
### 1．安装cocoapods
这个步骤参考网上的安装教程，很多，很详细了。

### 2．注册trunk（cocoaPod公开库发布需要）
注册cocoapods的trunk，在终端运行一下指令：

```
pod trunk register XXX@XXX.com '注册的名字' --verbose
```
或者

```
pod trunk register XXX@XXX.com 名字 --verbose
```
注册成功之后你会看到如下确认邮件的提示：

```
[!] Please verify the session by clicking the link in the verification email that has been sent to xxx@xxx.com
```

### 3．创建远程pod仓库

在自己的远程Git上创建pod仓库，本文以HQSpecs为例，在公司的私有Git仓库中创建HQSpecs仓库。具体操作略过。(公开库不需要)

### 4．添加关联本地pod仓库
在终端直接执行或进入```~/.cocoapods/repos```目录执行以下语句：
```
pod repo add HQSpecs http://repo.we.com/ape.zhang/hqspecs.git
```
会把github上的HQSpecs仓库克隆到本地，在repos下会生成一个HQSpecs仓库，这个跟cocoapods的master仓库是类似的。

### 5．创建远程项目仓库
在自己的远程Git上创建工程项目仓库，本文以HQFoundation为例，在公司的私有Git仓库中创建HQFoundation工程仓库。具体操作略过。

### 6．创建本地pod项目，关联远程项目仓库
cd到期望存放本地工程的目录，执行以下语句：

```
pod lib create HQFoundation
```
根据提示做相应选择如下所示：

```
What platform do you want to use?? [ iOS / macOS ]
 > iOS
What language do you want to use?? [ Swift / ObjC ]
 > ObjC
Would you like to include a demo application with your library? [ Yes / No ]
 > Yes
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > NOne
Would you like to do view based testing? [ Yes / No ]
 > NO
What is your class prefix?
 > HQ
```

会在你指定的目录下，根据cocoapods的pods模板生成一个pods工程，包含pods以及example的demo工程。
### 7．编辑pod项目，修改.podspec文件
编辑pod工程项目，在pod目录下的classes文件加下将replaceMe.m删除，添加自己要封装的类，可以有很多。本问以HQRootObject.h、HQRootObject.m为例。
编辑.podspec文件，本文如下：

```
Pod::Spec.new do |s|
  s.name             = 'HQFoundation'    #pod库名称
  s.version          = '0.1.0'            #pod库版本号
  s.summary          = 'HQFoundation.'  #简单介绍，不超过140字
  s.description      = <<-DESC           #详细描述
  这是我的Foundation库.
					   DESC
  #项目仓库首页
  s.homepage         = 'http://repo.we.com/ape.zhang/hqfoundation' 
  #协议，一般默认MIT
  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  #作者
  s.author           = { 'HQApe' => 'ape.zhang@corp.to8to.com' }
  #项目Git路劲，可以指明标签，分支等
  s.source           = { :git => 'http://repo.we.com/ape.zhang/hqfoundation.git', :tag => s.version.to_s }
  #在import时，引用的名字
  s.module_name      = 'HQFoundation'
  #系统版本
  s.ios.deployment_target = '8.0'
  #pod库需要包含的类和文件
  s.source_files = 'HQFoundation/Classes/*.{h,m}'
  #添加第三方依赖，去掉注释即可依赖第三方
  #s.dependency 'AFNetworking'
end
```

cd到Example目录下，执行pod install，就可得到以下工程结构：

```
-HQFoundation
-Pod
 |_Podfile
 |_Development Pods
 |  |_HQFoundation
 |  | |_ReplaceMe.m
 |  |_Pod
 |  |_Support Files
 |_Frameworks
 |....

```

### 8．检查.podspec文件
执行以下语句检查podspec文件：

```
pod lib lint
```
如有无关紧要的警告而未通过检查，则输入以下命令：```--allow-warnings```

### 9．Push提交本地pod项目到远程
提交本地pod项目工程到远程，在HQFoundation下执行以下语句：

```
git add .
git commit –m”添加HQFoundation类文件”
git push origin master
```
将本地工程推送到远程pod工程项目仓库

### 10．给pod项目打标签tag，并push到远程
执行以下语句，给项目打标签，注意标签要与pod版本一致：

```
git tag 0.1.0
git push --tags
```
推送成功后会显示：

```
Total 0 (delta 0), reused 0 (delta 0)
To http://repo.we.com/ape.zhang/hqfoundation.git
 * [new tag]         1.0.1 -> 1.0.1
```

### 11．检查远程.podspec
执行以下语句对远程podspec检查(特别注意，必须要先push工程，再打标签，顺序错了会通不过验证):

```
pod spec lint 
```

有警告的话加上 ```--allow-warnings```,验证通过的话，就基本可以完成了。

### 12．将本地pod项目的.podspec上传自己的本地和远程仓库
我们自己的spec仓库里，而不是cocoapods的仓库，我这里是HQSpecs仓库

公开库：

```
pod trunk push HQFoundation.podspec
```

私有库：

```
pod repo push HQSpecs HQFoundation.podspec
```

有警告的话加上 ```--allow-warnings```,cocoapods会把podspec文件上传到HQSpecs仓库中。

### 13．制作完成，使用自己的pod库
现在我们可以通过pod search HQFOundation 搜索我们的pod库
在podfile中指定自己的仓库路劲 
```
source 'git@repo.we.com:ape.zhang/hqspecs.git‘ 私有仓库路径
pod 'HQFoundation'
```
如果找不到，就执行以下pod repo update；

## 二、进阶

### 1.依赖自己的私有库
在podspec文件中添加

```
s.dependency 'HQUIKit', '~> 0.2.0'
```
验证的时候记得加上如下代码:

```
pod lib lint --sources=master,HQSpecs
```

HQSpecs是自己的私有库，master是GitHub公共仓库，如果有无关警告加上```--allow-warnings```
### 2．添加第三方依赖、系统Framework、libraries和静态.a文件
在podspec中做以下修改 

```
......
#第三方.a文件:  添加.a静态库的依赖
s.source_files='HQFoundation/Classes/*.{h,m}','HQFoundation/Classes/ThirdParty/*.{h}' 
#第三方.a文件:  添加.a静态库的依赖
s.vendored_libraries  = 'HQFoundation/Classes/ThirdParty/*.{a}' 
# 该pod依赖的系统framework:  添加.a依赖的系统framework
s.frameworks='SystemConfiguration','CoreGraphics','CoreTelephony','Security','CoreLocation','JavaScriptCore'
# 该pod依赖的系统library:  添加.a依赖的系统library
s.libraries  = 'iconv','sqlite3','stdc++','z'
# 添加第三方依赖
s.dependency 'AFNetworking'
......
```

Framework的依赖见高德的集成。

### 3．制作subspec
subspec就是如下所示：
```
SDWebImage (4.4.1)
   ......
   - Subspecs:
     - SDWebImage/Core (4.4.1)
     - SDWebImage/MapKit (4.4.1)
     - SDWebImage/GIF (4.4.1)
     - SDWebImage/WebP (4.4.1)
```
在podspec中做以下修改

```
#制作subspec
s.subspec 'HQSub' do |hs|
  hs.source_files = 'HQFoundation/Classes/HQSub/*.{h,m}'
end
```
建议，每一个subspec都是一个能够独立运行的项目
到podfile目录执行
```
pod install
```
### 4．升级pods的版本
将podspec的version 改为0.2.0
然后验证执行

```
pod lib lint
```

有警告的话加上 ```--allow-warnings```
将HQFoudation push至远程仓库并新建0.2.0的tag
远程验证通过，将podspec push至私有的spec仓库中搜索 ```pod search HQFoudation```

### 5．对高德等含第三方.framework静态库封装
这类操作主要在于.podspec文件的配置上，以及在pod使用的时候要注意到相关问题（在下面的问题中有说道）。下面附上对高德地图的配置：
#### 5.1使用手动集成的高德Framework：

```
s.source_files = 'TAMapLib/Classes/*.{h,m}'
#关联高德地图Framework
s.vendored_frameworks=['HQAMapLib/Classes/AMap_iOS_Lib/AMapFoundationKit.framework','HQAMapLib/Classes/AMap_iOS_Lib/AMapLocationKit.framework','HQAMapLib/Classes/AMap_iOS_Lib/AMapSearchKit.framework','HQAMapLib/Classes/AMap_iOS_Lib/MAMapKit.framework']
#系统的Framework
s.frameworks=['GLKit','OpenGLES','CoreGraphics','QuartzCore','CoreLocation','CoreTelephony','SystemConfiguration','Security','AdSupport','JavaScriptCore','AMapFoundationKit', 'AMapSearchKit', 'MAMapKit', 'AMapLocationKit']
    #系统的Libraries
s.libraries = ['z','stdc++','c++']
```

使用了第三方.a或.framework的，验证时需要加上：

```
--use-libraries 
```
#### 5.2依赖高德地图自动集成的pod
```
s.source_files = 'TAMapLib/Classes/*.{h,m}'
#系统的Framework
s.frameworks=['GLKit','OpenGLES','CoreGraphics','QuartzCore','CoreLocation','CoreTelephony','SystemConfiguration','Security','AdSupport','JavaScriptCore','AMapFoundationKit', 'AMapSearchKit', 'MAMapKit', 'AMapLocationKit']
#系统的Libraries
s.libraries = ['z','stdc++','c++']
s.dependency 'AMap3DMap'
s.dependency 'AMapSearch'
s.dependency 'AMapLocation'
```
注意：如果制作的这个高德地图库，被其他库所依赖，其他库也因该实现这些配置，否则，用到某些类会报Undefined symbols for architecture x86_64等错误。后面问题也有说道。目前只在OC中顺利使用，swift遇到的问题见下面的问题。

### 6．引用图片、xib、storyboard、plist等资源文件

将资源拖到pod的HQFoundation的Assert目录下，可以分文件，在podspec中配置如下：

```
s.resources = ['HQFoundation/Assets/*/*.png','HQFoundation/Classes/**/*.xib']
```
或

```
s.resource_bundles = {
   'HQFoundation' => ['HQFoundation/Assets/*/*.png','HQFoundation/Classes/**/*.xib']
}
```
利用 resources 属性时，这些资源文件build时会被直接拷贝到client target的mainBundle里。但是，这就带来了一个问题，那就是 client target 的资源和各种 pod 所带来的资源都在同一 bundle 的同一层目录下，很容易产生命名冲突。为了解决这一问题，CocoaPods 在 0.23.0 加入了一个新属性 resource_bundles。当然前者也有解决方案。资源的访问，self为pod中的任意类:
前者：

```
+ (UIImage *)imageInBoundleWithName:(NSString *)name {
    NSBundle *boundle = [NSBundle bundleForClass:[self class]];
    UIImage *image = [UIImage imageNamed:name inBundle:boundle compatibleWithTraitCollection:nil];
    return image;
}
```
后者：

```
+ (UIImage *)imageInBoundleWithName:(NSString *)name {
    NSBundle *boundle = [NSBundle bundleForClass:[self class]];
    NSBundle *currentBoundle = [NSBundle bundleWithPath:[boundle pathForResource:@"HQFoundation" ofType:@"bundle"]];
    UIImage *image = [UIImage imageNamed:name inBundle:currentBoundle compatibleWithTraitCollection:nil];
    return image;
}
```
加载xib：
前者：

```
NSBundle *xibBundle = [NSBundle bundleForClass:[self class]];
HQView *view = [xibBundle loadNibNamed:@"HQView" owner:nil options:nil].lastObject;
```
后者：
    
```
NSBundle *xibBundle = [NSBundle bundleForClass:[self class]];
NSBundle *currentBundle = [NSBundle bundleWithPath:[xibBundle pathForResource:@"HQFoundation" ofType:@"bundle"]];
HQView *view = [currentBundle loadNibNamed:@"HQView" owner:nil options:nil].lastObject;
```

### 7．创建分支pod库
创建分支pod库流程与上述流程不一样，具体如下：

1. 像以上pod库制作一样，通过

```ruby
$ pod lib create BranchRepoDemo
```
创建pod工程文件，并按照要求编辑podspec文件，podspec文件中的source要如下设置：

```ruby
......

s.source = { :git => 'https://github.com/xxx/BranchRepoDemo.git', :branch => "dev" }

......
```
同样要保证通过```$ pod lib lint```的校验, 不需要对工程打tag。

2. 关联远程origin/master分支

3. 在远程项目仓库中创建分支dev

4. 在本地项目仓库中有以下操作：

终端中Git分支相关命令

```ruby
$ git checkout -b dev //新建并切换到本地dev分支

$ git pull origin dev //本地分支与远程分支相关联

$ git push origin dev //本地分支推送到远程分支
```
成功推送后，即可使用，使用方法如下：

```ruby
pod 'pod库名', :git => 'https://github.com/xxx/BranchRepoDemo.git', :branch => 'dev'
```

## 三、遇到的问题

### 1、远程验证不通过
本地的私有仓库验证通过，但是远程仓库上的私有仓库验证不通过

1) 路经不对

```
Error[iOS] file patterns: The `source_files` pattern did not match any
```

解决办法：重新打开xxx.podspec文件编辑一下，确定共享文件路径没有错误，然后再上传到github上验证。

2) 上传xxx.podspec到github和给xxx.podspec打tag顺序搞反了,远程验证不通过
解决办法：必须先将本地文件夹所有的文件上传到github上

```
git add .
git commit -m"add all"
git push origin master
```

然后再给xxx.podspec打上tag，并上传

```
git tag -m "注释" 1.0.0
git push --tags
```
### 2、pod库依赖包含.framework静态库问题
当自己的pod库依赖了包含.Framework的pod库时，在使用时执行这个pod install会报错，使用这个库的时候也会报这个错。

es：制作pod库直接依赖高德的pod库时，包含s.source_files配置，也出现这个错误；不包含s.source_files没有这个错误

```
[!] The 'Pods-Test' target has transitive dependencies that include static 
binaries: (/Users/admin/Desktop/Test/Pods/AMap3DMap/MAMapKit.framework, 
  /Users/admin/Desktop/Test/Pods/AMapLocation/AMapLocationKit.framework, 
  and /Users/admin/Desktop/Test/Pods/AMapSearch/AMapSearchKit.framework)
```
解决办法：这个得在工程的podfile文件里加上下面这句话：

```
target 'HQSwiftLib_Example' do
	......
	pre_install do |installer| 
Pod::Installer::Xcode::TargetValidator.send(:define_method, :verify_no_static_framework_transitive_dependencies) {}
	end
end
```

### 3、Swift库的相关类、属性、方法无法访问
在创建swift的pod库时，需要公开的类、属性、方法，需要用public修饰，否则在工程中无法访问。又是一大坑。

### 4、在制作高德地图的pod库时，找不到框架：
pod中不包含任何类文件的时候，即podspec中设置了s.source_files，在工程中pod install后可以直接```#import <MAMapKit/MAMapKit.h>```等框架；但是单pod中包含其他类文件时，在工程中pod install后直接```#import <MAMapKit/MAMapKit.h>```等框架会报找不到```（'MAMapKit/MAMapKit.h' file not found）```。

通过比较发现，是Framework Search Paths 路径不对。前者的路劲是```"${PODS_ROOT}/../../MyRepo/Classes/AMap_iOS_Lib"```，后者的路劲是```"${PODS_CONFIGURATION_BUILD_DIR}/MyRepo"```。

解决办法：把后者改为```"${PODS_ROOT}/../../MyRepo/Classes/AMap_iOS_Lib"```就可以正常#import了。因此，可以探索如何通过pod自动配置为```"${PODS_ROOT}/../../MyRepo/Classes/AMap_iOS_Lib"```。

后期使用，遇到了问题5，又坑了一把.

### 5、工程中能引用但不能调用
制作pod库依赖自己制作的高德地图.a静态库或依赖微信.a静态库时，podspec文件里没有把高德地图需要的系统库和他本身的Framework都引用，在该pod或工程里可以import，却不可使用，报错：

```
Undefined symbols for architecture x86_64:
  "_OBJC_CLASS_$_AMapLocationManager", referenced from:
      objc-class-ref in HQMapViewController.o
ld: symbol(s) not found for architecture x86_64
```
原因：在制作的高德微信pod库中，s.source_files中如果没有使用相关类，在依赖者或工程中都是无法使用的会报：

```
Undefined symbols for architecture x86_64/armv7/...:
"_OBJC_CLASS_$_MAMapView", referenced from:
      objc-class-ref in HQViewController.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
```

解决方法：可以将功能实现封装在pod中，例如微信支付、分享等。有个奇怪的问题是，在pod内用到的类，在工程中使用就不会报以上错误。

### 6、部分高德地图分模块无法引用
使用直接依赖高德pod库，封装好自己的高德pod库，在swift的pod库中无法使用，import AMapLocationKit提示No such module 'AMapLocationKit'，还有'AMap2DMap'。提示是缺少module，因此建议高德地图组件单独OC封装或混编封装，已向高德提工单，看看什么情况。

### 7、高德地图提示apiKey没有注册
在Appdelegate里注册了高德地图的apiKey依然提示没有注册

```
[AMapServices sharedServices].apiKey = @"your apiKey";
```
[AMapFoundationKit][Info] : 错误信息：apiKey为空，请正确设置AMapServices.apiKey。请在 AppDelegate.m 文件的 didFinishLaunchingWithOptions 方法中配置高德 Key。

将注册代码放到封装的pod里执行即可。我这里是有个配置文件config：

```
TMHConfig.config(withUrl: "url",
                     uid: uid,
                   token: "tiket",
                 appName: "name",
               gd_apiKey: "your apiKey")
```

配置文件建议放到使用这个库的时候在配置。

### 8、pod库仓库与工程仓库共用search不到
本地和远程验证都通过，也成功push了，但是```pod search```却找不到。

解决办法：在制作私有pod库时，要将仓库和工程分开建立，否则会出现验证通过，却找不到库的问题，本文制作的私有库就是将pod仓库和项目仓库分开，便于管理。

### 9、使用封装好的pod库，报类在两个地方实现
使用直接依赖高德pod库，封装好自己的高德pod库，在工程中使用时会报

```
Class AMapInitRequestReformer is implemented in both /Users/admin/Library/Developer/CoreSimulator/Devices/3D47A858-C1C7-40D2-8D92-139E3E403BE0/data/Containers/Bundle/Application/2C044E8E-6927-4EAD-B6D3-8BCEBDFE69DA/podMap_Example.app/Frameworks/podMap.framework/podMap (0x1035eae10) and /Users/admin/Library/Developer/CoreSimulator/Devices/3D47A858-C1C7-40D2-8D92-139E3E403BE0/data/Containers/Bundle/Application/2C044E8E-6927-4EAD-B6D3-8BCEBDFE69DA/podMap_Example.app/podMap_Example (0x1014dd7a0). One of the two will be used. Which one is undefined.

```

### 10、当遇到swift里面无法实现的功能时，可以考虑OC和swift混编实现。

### 11、删除本地和远程的tag。
```ruby
git tag　　//查看tag
...
git tag -d test_tag　　　　　　　　//本地删除tag
git push origin :refs/tags/test_tag　　　　//本地tag删除了，再执行该句，删除线上tag
```


## 参考资料

[给 Pod 添加资源文件](http://blog.xianqu.org/2015/08/pod-resources/)

[在Pod库中使用xcasset的拷贝陷阱](https://www.jianshu.com/p/63a4e2920f22)

[关于制作私有pod库包含framework和.a文件时遇到的一些问题](https://blog.csdn.net/w_shuiping/article/details/80606277)

[使用私有Cocoapods仓库 中高级用法](https://www.jianshu.com/p/d6a592d6fced)





