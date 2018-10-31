
## 混合开发HybridAPP

> 小杂种APP，混合APP

app: application (应用程序)，在这里泛指移动设备上使用的应用程序

##### 对比现在流行的三种APP：WebAPP，NativeAPP,HybridAPP

WebAPP: （移动端网站）

* 不需要下载安装，通过移动端浏览器来访问

* HTML5+CSS3+JS；Web前端开发人员

* 开发成本低，更新维护迭代成本也很低，使用成本低，跨平台

* 较为依赖网络，流畅度较低，吃性能

* 不能调用设备的原生功能


NativeAPP: ( apk ipa)

* 需要下载安装，打开访问 (app store)

* IOS开发人员(oC,swift)，Android开发人员(java) xml+Objective-C/xml+swift(ios),xml+java

* 开发成本高，更新维护迭代成本也很高，使用成本高，不能跨平台

* 基本不太依赖网络，流畅度较高，性能好

* 可以调用设备的原生功能

HybridAPP：

目前流行的开发模式有两种：

1. 原生主导开发（最广泛，最简单）

	大部分功能还是由native开发人员来开发，部分界面嵌入H5页面来实现，这样就可以将nativeApp和webapp的优点集合到一起了
	
	稳定性、兼容性都会比较好
	
	怎么去判断一个APP是nativeAPP还是HybridAPP：
	
	* 长按文字，看是否能选中
	
	* 打开手机的开发者模式

	其实开发HybridAPP内嵌的H5页面和开发纯WebAPP的区别在于：需要和原生Native进行交互，这些方法都很简单。还有一个知识就是在某些情况下需要判断ios还是Android，原理：利用window.navigator.userAgent
	
	[判断ios、android](http://www.jb51.net/article/117472.htm)
	
	附录：Native与JS交互

2. H5主导开发
	
	前端开发者利用一些工具来进行HybridAPP的开发，内容界面都是H5页面，在外面套上native的壳子，打包成APP
	
	使用的工具：
	
	* DCLOUD：Hbuilder+mui+（h5+runtime） (mui,h5+ )
	* weex、ApiCloud
	* phonegap+cordova 需要在电脑上配置java andrord jdk...，mac xcode
	* ionic

优点：

* 开发成本低，更新维护迭代成本也很低，使用成本低，跨平台
* 基本不太依赖网络，流畅度较高，性能好
* 可以调用设备的原生功能


### Hbuilder编辑器

1. 进行真机调试

	* 打开要调试的项目
	* 手机连接电脑
	* 在Hbuilder基座上运行项目，hbuilder就会为手机安装一个app：hbuilder
	* 在手机基座中访问该项目
	
2. 云端打包

	* 配置manifest.json打包配置文件
	* 发行为原生安装包。注意，打包ios的话需要使用苹果开放者证书，没有的话只能打包越狱包
	* 会发送到云端（dcloud服务器）去打包，成功后下载下来安装包
	
	


# Native与JS交互方式

## 前言

我们知道混合开发的模式现在主要分为两种，H5工程师利用某些工具如DCLOUD产品、codorva+phonegap等等来开发一个外嵌native壳子的混合app

还有就是应用比较广泛的，有native开发工程师和H5工程师一起写作开发的应用，在native的webview里嵌入H5页面，当然只是部分界面这么做，这样做的好处就是效率高，开发成本和维护成本都比较低，较为轻量，但是有一个问题不可避免的会出现，就是js和native的交互

native与js交互部分等详细内容请移步这里：

[简书资源](http://www.jianshu.com/p/d19689e0ed83)

[掘金资源](https://juejin.im/post/599a58f6f265da247b4e756b)

---

### Native（Objective-C或Swift）调用Javascript方法

#### 1.Native调用Javascript语言，是通过UIWebView组件的stringByEvaluatingJavaScriptFromString方法来实现的，该方法返回js脚本的执行结果。


```
// Swift
webview.stringByEvaluatingJavaScriptFromString("Math.random()")  
// OC
[webView stringByEvaluatingJavaScriptFromString:@"JSBridge.init(1)"];
```

从上面代码可以看出它其实就是调用了window下的一个对象，如果我们要让native来调用我们js写的方法，那这个方法就要在window下能访问到。但从全局考虑，我们只要暴露一个对象如JSBridge让native调用就好了，所以在这里可以对native的代码做一个简单的封装：


```
//下面为伪代码
webview.setDataToJs(somedata);
webview.setDataToJs = function(data) {
 webview.stringByEvaluatingJavaScriptFromString("JSBridge.trigger(event, data)")
}
```

另外：==在android中，native与js的通讯方式与ios类似==

#### 2.在iOS 7之后，apple添加了一个新的库JavaScriptCore


```
JSContext *context = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
NSString *textJS = @"showAlert('这里是JS中alert弹出的message')";
[context evaluateScript:textJS];
```

---
## Javascript -> OC/Swift 

Javascript调用Native，并没有现成的API可以直接拿来用，而是需要间接地通过一些方法来实现。UIWebView有个特性：在UIWebView内发起的所有网络请求，都可以通过delegate函数在Native层得到通知。这样，我们就可以在UIWebView内发起一个自定义的网络请求，通常是这样的格式：

```
jsbridge://methodName?param100000=value1&param2=value2
```

发起这样一个网络请求有两种方式：

1. 通过localtion.href；
2. 通过iframe方式；

通过location.href有个问题，就是如果我们连续多次修改window.location.href的值，在Native层只能接收到最后一次请求，前面的请求都会被忽略掉。

使用iframe方式，以唤起Native APP的分享组件为例，简单的封闭如下：


```
var url = 'jsbridge://doAction?title=分享标题&desc=分享描述&link=http%3A%2F%2Fwww.baidu.com';
var iframe = document.createElement('iframe');
iframe.style.width = '1px';
iframe.style.height = '1px';
iframe.style.display = 'none';
iframe.src = url;
document.body.appendChild(iframe);
setTimeout(function() {
    iframe.remove();
}, 100);
```


#### 2.还有一种方式就是使用JavaScriptCore

定义好JS需要调用的方法，例如JS要调用share方法：

则可以在UIWebView加载url完成后，在其代理方法中添加要调用的share方法

这样的话web页面中就可以直接使用到这个方法：


```
function secondClick() {
    share('分享的标题','分享的内容','图片地址');
}
...
<button type="button" onclick="secondClick()">Click Me!</button>
```

---

### javascript调用native Android方式

目前在android中有三种调用native的方式：

#### 1.通过schema方式，使用shouldOverrideUrlLoading方法对url协议进行解析。这种js的调用方式与ios的一样，使用iframe来调用native代码。

#### 2.通过在webview页面里直接注入原生js代码方式，使用addJavascriptInterface方法来实现。
在android里实现如下：


```
class JSInterface {
    @JavascriptInterface //注意这个代码一定要加上
    public String getUserData() {
        return "UserData";
    }
}
webView.addJavascriptInterface(new JSInterface(), "AndroidJS");
```

上面的代码就是在页面的window对象里注入了AndroidJS对象。在js里可以直接调用


```
alert(AndroidJS.getUserData()) //UserDate
```

#### 3.使用prompt,console.log,alert方式，这三个方法对js里是属性原生的，在android webview这一层是可以重写这三个方法的。一般我们使用prompt，因为这个在js里使用的不多，用来和native通讯副作用比较少。


```

class YouzanWebChromeClient extends WebChromeClient {
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        // 这里就可以对js的prompt进行处理，通过result返回结果
    }
    @Override
    public boolean onConsoleMessage(ConsoleMessage consoleMessage) {

    }
    @Override
    public boolean onJsAlert(WebView view, String url, String message, JsResult result) {

    }

}
```
---

## 总结

#### OC/SWIFT调用js

直接用一些方法执行我们js中的一些语句，也就是说，我们最好定义一个JSBridge对象，上面放着一些方法准备被native调用，当然也就可以在这些方法里传点参数啥的给咱们了

#### js调用ios

咱们可以整一个请求发出去，这个请求呢会被native给拦截到，他就指的啥意思了

比如，我们可以通过 location.href=A://b=1&c=2&d=3 当然这里的A、b、c、d都要商量好，bcd就是传参数

但是location.href只能发一次，所以我们可以用iframe去发，发完了给iframe干掉就可以了

或者，native会给window对象上放上一些方法，我们直接调用就可以了（ios7以上）

#### android 调用 js 和oc、swift一样，这里就不说了

#### js调用Android

1.也跟调用ios一样，搞个请求，用个iframe

2.Android能想办法给咱的window对象上挂个东西，比如JSBridge啥的然后咱直接调这个玩意的方法就行了

3.他们能把咱的prompt、console.log、alert给重写咯，也就是说咱用alert已经不能弹出了，反而能给Android传参数了，但是一般不会重写alert，重写的都是不怎么用的prompt