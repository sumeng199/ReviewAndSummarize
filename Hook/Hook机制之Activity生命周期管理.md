## [Activity生命周期管理](http://weishu.me/2016/03/21/understand-plugin-framework-activity-management/)
在Java平台要做到动态运行模块、热插拔可以使用ClassLoader技术进行动态类加载，比如广泛使用的OSGi技术。在Android上当然也可以使用动态加载技术，但是仅仅把类加载进来就足够了吗？Activity，Service等组件是有生命周期的，它们统一由系统服务AMS管理；使用ClassLoader可以从插件中创建Activity对象，但是，一个没有生命周期的Activity对象有什么用？所以在Android系统上，**仅仅完成动态类加载是不够的；我们需要想办法把我们加载进来的Activity等组件交给系统管理，让AMS赋予组件生命周期；这样才算是一个有血有肉的完善的插件化方案。**

### 绕过AndroidManifest.xml的限制
