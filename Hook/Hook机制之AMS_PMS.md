## [Hook机制之AMS&PMS](http://weishu.me/2016/03/07/understand-plugin-framework-ams-pms-hook/)

AMS：ActivityManagerService
PMS：PackageManagerService

AMS和PMS就是以Binder方式提供给应用程序使用的系统服务，理论上我们也可以采用这种方式Hook掉它们。但是由于这两者使用得如此频繁，Framework给他们了一些“特别优待”，这也给了我们相对于Binder Hook更加稳定可靠的hook方式。

## AMS
对于FrameWork层的重要性不言而喻， Android的四大组件无一不与它打交道
>- **startActivity**：最终调用了 AMS 的 startActivity 系列方法，实现了 Activity的启动；Activity的生命周期回调，也在AMS中完成
- **startService, bindService**：最终调用到 AMS 的 startService 和 bindService 方法
- 动态广播的注册和接收在AMS中完成（静态广播在 PMS 中完成）
- **getContentResolver**：最终从AMS 的 getContentProvider 获取到 ContentProvider


## PMS
完成了诸如权限校验（checkPermission, checkUidPermission）, Apk meta信息获取（getApplicationInfo等），四大组件信息获取（query系列方法）等重要功能

有两个地方需要Hook掉
>- ActivityThread 的静态字段 sPackageManager
- 通过Context类的 getPackageManager() 方法获取到的 ApplicationPackageManager 对象里面的 mPM 字段
