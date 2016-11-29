### [SimpleJavaJsBridge: 交互库，需要与服务器端共同定一个协议](http://mp.weixin.qq.com/s/8f-zHzKLprtTGreA7uRHFA)

### 已有Js与Java通信方案

#### java 给 Js 发送消息
官方指定的唯一方法： **webview.loadUrl()**
````python
//例子：调用js的test(param)方法:  "javascript:"+js方法的名字＋方法的参数值
webView.loadUrl("javascript:test(1)");
````

#### Js 给 Java 发送消息
实际上只有 2 种方案
##### 1，官方方法
