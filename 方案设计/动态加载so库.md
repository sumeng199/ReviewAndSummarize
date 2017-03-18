## 动态加载so库

### 基础点
- 一种CPU架构 = 一种ABI = 一种对应的SO库；
- 加载SO库时，需要加载对应CPU架构类型的SO库；
- 尽量提供全平台CPU类型的SO库支持 (服务器/本地)；

### 网络下载SO方案

- 全部下载，然后加载对应CPU架构类型的SO库
  >不现实，下载包大且包含其它无用CPU架构支持库

- 只下载当前设备对应类型的SO库，然后加载
  >本地获取设备支持的CPU架构类型列表，按照支持优先级排序，上传服务器，服务器根据优先级返回最优的CPU类型SO库下载链接

----
### 加载SO方案

### 方案一：
>下载对应版本的 so 库，复制到内部存储（外部存储中的文件没有可执行的权限, 可以复制到应用私有目录下, 即 data/data/packagename/... 目录下），再使用 System.load(String libPath) 方法加载 so 文件。无需root，可直接操作。需root方案没实际意义就不写了。

### 方案二：只懂java的略过不需要看
>NDK 加载方案：下载对应版本 so 库，无需复制到内存当中，直接在SD卡上，通过 dlopen 打开SD卡中的so加载到内存，然后通过 dlsym 查找 so 中的函数地址
[链接](http://114.215.110.123/article/mobile/299417.html)


### 方案三：结合热修复思想，扩展性强，且不仅限于对SO的动态加载。但是会涉及到插件化开发，需要谨慎考虑项目中是否有必要引入
>[结合插件化apk原理](https://github.com/limpoxe/Android-Plugin-Framework), 将so库按不同CPU架构类型打包成相对应的插件apk，放在服务器端，客户端根据当前设备的CPU类型下载相应的插件apk后，插入宿主apk即可(即主项目)。
存储在服务器的SO库依然按照APK包的压缩方式打包，也就是，SO库存放在APK包的 libs/xxxabi 路径下面。

[插件化开发推荐](https://github.com/quintushorace/android_plug)
