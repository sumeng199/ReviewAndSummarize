## Protocol Buffers

### 简介
Protocol Buffers(也称protobuf)是Google公司出口的一种独立于开发语言，独立于平台的可扩展的结构化数据序列机制。通俗点来讲它跟xml和json是一类。是一种数据交互格式协议。

网上有很多它的介绍，主要优点是它是基于二进制的，所以比起结构化的xml协议来说，它的体积很少，数据在传输过程中会更快。另外它也支持c++、Java、Python、PHP、JavaScript等主流开发语言

### 使用场景
经常是设备与设备之间进行通信，而不是设备与服务器做通信。很多设备是 **Linux** 下 **C语言** 做核心服务，c来解析json比较麻烦。

### 学习博客
- [通信协议之Protocol buffer(Java篇)](http://blog.csdn.net/briblue/article/details/53187780#comments)
