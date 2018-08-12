
<center>WebViewJavascriptBridge方案设计</center> 

[TOC]
## 一、方案分析

### 1、iOS

1）方案比较：

(1)拦截url:

(2)JavaScriptCore:

(3)WKScriptMessageHandler:

web调用原生：

<1>约定协议，如jxaction://scan表示启动二维码扫描，jxaction://location表示获取定位。 

<2>实现UIWebView代理的shouldStartLoadWithRequest:navigationType:方法，在方法中对url进行拦截，解析参数.判断是否继续加载原url。

<3>通过自定义类遵循JSExport代理，通过JSContext注入对象或Block实现调用或传值。
```
context[@"openAlbum"] = ^(id params){
    NSLog(@"js调用oc打开相册");
};
```

<4>iOS8+的WKWebView初始化WKWebView时，调用addScriptMessageHandler:name:方法，name为js中的方法名，实现WKScriptMessageHandler代理方法，当js调用name方法时会回调userContentController:didReceiveScriptMessage:

原生调用js：

<1>直接调用UIWebView的stringByEvaluatingJavaScriptFromString:方法，或者WKWebView的 evaluateJavaScript:completionHandler:方法。可以在javaScript字符串方法中拼接参数传参。

<2>通过JSContext调用evaluateScript:传值
```
JSContext *context=[_mainWebView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
NSString *jsString= [NSString stringWithFormat:@"%@('%@')",method,params];
[context evaluateScript:jsString];
```

(4)第三方框架：

WebViewJavascriptBridge：

DSBridge for IOS:

使用简单，交互方便，参数清晰，扩展性强。

JS调原生：仅需在原生中注册JS需要调用的方法，供JS调用即可，JS可通过callBack等待原生响应。

原生调JS：仅需在JS中注册原生需要调用的方法，供原生调用即可，原生可通过callBack等待JS响应。

(5)hybird App框架：


2）系统兼容性：

系统UIWebView，WKWebView根据适配系统选择。

第三方WebViewJavascriptBridge：兼容WKWebViews, UIWebViews 以及 WebViews.

hybird App：兼容性也不错。

3）优缺点：

原生中，我们可以在OC中调用javascript方法，但是反过来不能在javascript中调用OC方法，虽然后面的WebView可以通过对象注入传参。但是，无论调用还是被调用原生中写的JS代码，以及参数传递特别的复杂，不但不好管理和维护，也影响开发效率。

第三方框架WebViewJavascriptBridge：使用简单，交互方便，参数清晰，扩展性强。



### 2、Android
1）方案比较：

DSBridge:

WebViewJavascriptBridge:(有两个作者，其中一个是DSBridge作者)

2）系统兼容性：

部分框架存在Bug，已知的WebViewJavascriptBridge，h5打不开文件(例如调相册之类的)，可以改源码。

3）优缺点：

应该跟iOS差不多吧。

## 二、方案设计

### 1、事件处理
    1）点击事件

    2）公共事件

### 2、页面跳转
    1) WebView内部重定向

    2) WebView跳转到WebView

    3) WebView跳转到原生

### 3、数据交互
    1) 基本信息

    2) 鉴权信息

    3) 采集信息

## 三、实践
目前工程中采用的是URL拦截来进行js与原生调方法传值；通过evaluateJavaScript:进行原生调JS方法和传值。

## 四、参考资料
[WebViewJavascriptBridge For iOS](https://github.com/marcuswestin/WebViewJavascriptBridge)

[WebViewJavascriptBridge For Android](https://github.com/jesse01/WebViewJavascriptBridge)

[DSBridge For iOS](https://github.com/wendux/DSBridge-IOS)

[DSBridge For Android](https://github.com/wendux/DSBridge-Android)

[JsBridge Android](https://github.com/lzyzsd/JsBridge)






