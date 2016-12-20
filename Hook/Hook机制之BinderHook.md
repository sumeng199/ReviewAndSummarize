## [Hook机制之Binder Hook](http://weishu.me/2016/02/16/understand-plugin-framework-binder-hook/)

Android系统通过 **Binder** 机制给应用程序提供了一系列的系统服务，诸如 **ActivityManagerService**, **ClipboardManager**, **AudioManager** 等；这些广泛存在于系统服务中，给应用程序提供了诸如任务管理，音频，视频的异常强大的功能。

插件框架作为各个插件的管理者，为了使得插件能够无缝地使用这些系统服务，自然会对这些系统服务做出一定的改造(Hook)，使得插件的开发和使用更加方便，从而大大降低插件的开发和维护成本。比如，Hook住ActivityManagerService可以让插件无缝地使用startActivity方法而不是使用特定的方式(比如that语法)来启动插件或者主程序的任意界面。


| 过程 |  | 备注 |
| :----: | :-------: | :-------: |
| 系统服务获取过程 | -- | 系统的各个远程 **service** 对象都是以 **Binder** 的形式存在的，有一个管理者， **ServiceManager**, 所以要 **Hook** 的话需要从这个管理者入手 |
| 寻找 **Hook** 点 | | 由于系统服务的使用者都是对第二步获取到的IXXInterface进行操作，因此如果我们要hook掉某个系统服务，只需要把第二步的asInterface方法返回的对象修改为为我们Hook过的对象就可以了。 |
| asInterface过程 | | |

### 总结
要达到修改系统服务的目的，需要步骤
- 首先需要**伪造一个系统服务对象**， 接下来想办法让 asInterface 能够返回我们的这个伪造对象而不是原始的系统服务对象
- 只要让 **getService** 返回的 **IBinder** 对象的 **queryLocalInterface** 方法直接返回我们伪造过的系统服务对象就能达到目的。所以，我们需要伪造一个 **IBinder** 对象，主要是修改它的 queryLocalInterface 方法， 让它返回我们伪造的系统服务对象，然后把这个伪造对象放置在 **ServiceManager** 的缓存 **map** 里面即可
