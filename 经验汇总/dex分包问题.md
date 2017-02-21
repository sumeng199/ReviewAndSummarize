### 调研背景
> 项目中目前使用官方提供的MultiDex解决 linearAlloc以及64k的限制[^three]，由于MultiDex自身的问题以及项目目前所遇到的问题寻找优化方案
>- 首次启动时间长，出现黑屏，在特别低端的机器上可能会出现ANR
>- 随着项目功能的丰富，体积的增大，上述问题会变得越来越严重（因为classes2.dex会越来越大）

---
### 官方MultiDex解决方案的坑
>- 在应用安装到手机上的时候dex文件的安装是复杂的(complex)有可能会因为第二个dex文件太大导致ANR。请用proguard优化你的代码
>- 使用了mulitDex的App有可能在4.0(api level 14)以前的机器上无法启动，因为Dalvik linearAlloc bug(Issue 22586) 。请多多测试自祈多福。用proguard优化你的代码将减少该bug几率。
>- 使用了mulitDex的App在runtime期间有可能因为Dalvik linearAlloc limit (Issue 78035) Crash。该内存分配限制在 4.0版本被增大，但是5.0以下的机器上的Apps依然会存在这个限制。
>- 主dex被dalvik虚拟机执行时候，哪些类必须在主dex文件里面这个问题比较复杂。build tools 可以搞定这个问题。但是如果你代码存在反射和native的调用也不保证100%正确。

---
### 原因说明
#### ANR
install dex + dexopt[^code] 时间太长
```python
启动流程
1：安装完app点击图标后，系统发现没有对应的process，于是从该apk中抽取 classes.dex(主dex)加载，触发一次 dexopt。

2：App的LauncherActivity准备启动，触发Application启动

3：Application的 attachBaseContext()调用，执行里面的 MultiDex.install()，classes2.dex 被 install，再次触发 dexopt。（主线程执行）

4：执行 Application.onCreate()

5：launcher Activity被调起

上述流程必须在 5s 内执行完成

```
#### Crash

---
### 解决思路
#### 微信/手Q
##### 1：整体思路：
- 将首次加载放于地球（启动页）中，并用线程去加载（5.0之前加载dex时还是会挂起主线程）
- 加载Dex的方式
>加载逻辑主要是判断是否已经 dexopt，如果已经 dexopt，即放在 attachBaseContext 加载，反之放在启动页加载
>- 判断方法：判断应用版本号是否改变，改变就将 dex 以及 dexopt目录清空


##### 2：可行性分析
- dex放于assets下，无法利用到Android5.0 的art启动优势
- 需要自己定制分包规则，


#### 美团
##### 1：整体思路：
精简主dex + 异步加载secondary.dex。对异步执行速度的不确定性，解决方案是重写 Instrumentation  execStartActivity方法，hook跳转Activity的总入口做判断，如果当前secondary.dex还没有加载完，就执行等待操作（弹loading等等）
##### 2：可行性分析
- 分析主dex需要的classes这个脚本比较难写。。
>关于写分析脚本的思路是：直接使用mini-main-list参数获取build目录下的main-list文件，这样manifest声明的类和他们的直接依赖类搞定了，那后者的直接依赖类怎么解？这些在dvk runtime也是必须的classes。一个思路是解析class文件获得该class的依赖类。还一个思路是自己使用Dexclassloader 加载dex，然后hook getClass()方法，调用一次就记录一个。(太麻烦，不靠谱)
- 在App的manifest中注册的组件的类中，承载的业务太多，依赖很多第三方jar，导致直接依赖类非常多，短时间内无法梳理精简，没办法mini化主dex
- Application的启动入口问题：若不是由launcher Activity的启动触发，而是由Service, Receiver, ContentProvider的启动。只靠拦截重写 Instrumentation的 execStartActivity解决不了问题。

#### FB
##### 1：整体思路：
让 LauncherActivity 在另外一个进程启动！用来 load dex， load完成就启动MainActivity


---
### 参考资料
- [MultiDex工作原理分析和优化方案](https://zhuanlan.zhihu.com/p/24305296)
- [MultiDex实现原理解析](http://allenfeng.com/2016/11/17/principle-analysis-on-multidex/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [Android拆分与加载Dex的多种方案对比](http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=207151651&idx=1&sn=9eab282711f4eb2b4daf2fbae5a5ca9a&3rd=MzA3MDU4NTYzMw==&scene=6#rd)
- [MultiDex造成的启动性能问题](http://jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/1223/3796.html)
- [linearAlloc限制以及分包加载性能解决: 推荐](http://blog.zongwu233.com/the-touble-of-multidex)
- [美团Android DEX自动拆包及动态加载简介](http://tech.meituan.com/mt-android-auto-split-dex.html)

---
### 基础知识
#### dex如何打包的
Google文档说过这个问题比较复杂， 而且buildTools 已经帮我们搞定了。 它是如何工作的？文档说dex的时候，先依据manifest里注册的组件生成一个 main-list，然后把这list里的classes所依赖的classes找出来，把他们打成classes.dex就是主dex。剩下的classes都放clsses2.dex(如果使用参数限制dex大小的话可能会有classe3.dex 等等） 。主dex至少含有main-list 的classes + 直接依赖classes ，使用mini-main-list参数可以仅仅包含刚才说的classes。

[^code]:#### dexopt是什么
[^three]: #### [64k方法数与 linearAlloc (线性内存)原理分析](http://ingramchen.io/blog/2014/09/prevention-of-android-dex-64k-method-size-limit.html)
