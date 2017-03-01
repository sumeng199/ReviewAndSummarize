## 动态加载so库

### 学习资料
- [Android动态加载补充 加载SD卡中的SO库](https://segmentfault.com/a/1190000004062899)
- [Android动态加载 使用SO库时要注意的一些问题](https://segmentfault.com/a/1190000005646078)

### 注意点
- 一种CPU架构 = 一种ABI = 一种对应的SO库；
- 加载SO库时，需要加载对应类型的SO库；
- 尽量提供全平台CPU类型的SO库支持；

### 网络下载SO加载方案

- 全部下载，然后加载对应CPU架构类型的SO库
  >不现实

- 只下载当前设备对应类型的SO库，然后加载
  >本地获取设备支持的CPU架构类型列表，按照支持优先级排序，上传服务器，服务器根据优先级返回最优的CPU类型SO库

----
### 方案一：
下载对应版本的 so 库，复制到内部存储（外部存储中的文件没有可执行的权限, 可以复制到应用私有目录下, 即 data/data/packagename/... 目录下），再使用 System.load(String libPath) 方法加载 so 文件。


----
### 方案二：感觉最优，有很大的扩展性，而不仅限于对SO的动态加载；但是会涉及到插件化开发，需要谨慎考虑是否有必要
1： [结合插件化apk原理](https://github.com/limpoxe/Android-Plugin-Framework), 将so库按不同CPU架构类型打包成相对应的插件apk，放在服务器端，客户端根据当前设备的CPU类型下载相应的插件apk后，插入宿主apk即可(即主项目)。
存储在服务器的SO库依然按照APK包的压缩方式打包，也就是，SO库存放在APK包的 libs/xxxabi 路径下面。

### [插件化](https://github.com/limpoxe/Android-Plugin-Framework)
若插件中包含so，则需要在宿主的相应目录下添加至少一个so文件，以确保插件和宿主支持的so种类完全相同

>例如：插件包含armv7a、arm64两种so文件，则宿主必须至少包含一个armv7a的so以及一个arm64的so。
   若宿主本身不需要so文件，可随意创建一个假so文件放在宿主的相应目录下。例如pluginMain工程中的libstub.so其实只是一个txt文件。
   需要占位so的原因是，宿主在安装时，系统会扫描宿主中的so的（根据文件夹判断）类型，决定宿主在哪种cpu模式下运行、并保持到系统设置里面。
   (系统源码可查看com.android.server.pm.PackageManagerService.setBundledAppAbisAndRoots()方法)
   例如32、64、还是x86等等。如果宿主中不包含so，系统默认会选择一个最适合当前设备的模式。
   那么问题来了，如果系统默认选择的模式，和将来下载安装的插件中支持的so模式不匹配，则会出现so异常。
   因此需要提前通知系统，宿主需要在哪些cpu模式下运行。提前通知的方式即内置占位so。
