# Android 7.0新特性

## 多窗口支持

* Android 7.0的手机和平板上，用户可以并排运行两个应用
* Android TV设备上，应用可以将自身置于画中画模式

## 通知增强功能

* 模板更新

  ​	更新了通知模板

* 消息传递样式自定义

  ​	自定义使用MessageStyle类的通知相关的用户界面标签，可以配置消息、会话标题和内容视图

* 捆绑通知

  ​	系统可以将消息组合在一起（例如：按消息主题）并显示组，用户可以进行拒绝或归档的操作

* 直接回复

  ​	对于实时通信的应用，Android系统支持内联回复，以便用户在通知界面进行快速回复

* 自定义视图

  ​	通知中可以使用自定义视图时，可以利用系统装饰元素，如通知标题和操作等

## 配置文件指导的JIT/AOT编译

​	在Android 7.0，添加了即时的（JIT）编译器，对ART进行代码分析，应用在运行的时候持续提升Android应用的性能。

​	配置文件指导的编译有助于减少整个RAM的占用

​	降低设备对电池的影响，节约时间和省电

## 快速的应用安装路径

​	JIT编译器最实际的好处之一就是提高了应用安装和系统更新的速度，系统更新省去了优化步骤

## 随时随地低电耗模式

​	屏幕关闭了一段时间之后，并且没有插入电源，低电耗模式就会对应用使用书序的CPU和网络限制

## Project Svelte :后台优化

​	扩展了JobScheduler和 GCMNetworkManager   ,删除了三个常用的隐式广播：		CONNECTIVITY_ACTION、ACTION_NEW_PICTURE和ACTION_NEW_VIDEO 

## SurfaceView

​	在Android 7.0 开始 ，建议使用SurfaceView代替TextureView

## 流量节省程序

​	启用流量节省程序后，在使用数据流量进行上网的时候，系统屏蔽后台流量消耗，同事指示应用在前台尽可能使用较少的流量

## Vulkan API

​	Android 7.0 之后一项新的3D渲染API ,vulkan是3D图形和渲染的一项开放标准,仅适用于已启用Vulkan硬件设备的应用，如Nexus 5X....

## Quick Settings Tile API

​	快速设置，可以通过向左向右滑动显示更多的设置，用户可以拖动图块来进行添加或移动

## 号码屏蔽

## 来电过滤

​	实现新的CallScreeningService,允许手机基于来电的Call.Details 执行大量操作 

*  拒绝来电
*  不允许来电到达通话记录
*  不向用户显示来电通知

## 多语言区域支持，更多语言

## 新增表情符号

## Android 中的ICU4J API

## WebView

* Chrome 和WebView 配合使用
* 多进程
* JavaScript 在页面加载前运行
* 不安全七点上的地理定位
* 测试WebView 测试版

## OpenGL™ ES 3.2 API

## Android TV 录制

## Android for work 

* 工作资料安全性挑战
* 关闭工作
* Always on VPN
* 自定义配置

## 无障碍增强功能

## 直接启动

## 密钥认证

## 网络安全性配置

* 自定义信任锚
* 仅调试重写
* 明文流量选择退出
* 证书固定

## 默认受信任的证书颁发机构

## APK singature scheme v2

## 作用域目录访问

## 键盘快捷键辅助工具

## Custom Pointer API

## Sustained Performance API

## VR支持

## 打印服务增强

## FrameMetricsListener API

## 虚拟文件











​	