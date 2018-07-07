【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# UIApplication
iOS中应用的集中控制和协调点。

## 概述
每一个iOS应用都有一个`UIApplication`实例（或者极少数情况下是`UIApplication`的子类）。当应用启动的时候，系统会调用`UIApplicationMain`方法；在它的其他任务中，这个方法创建一个`UIApplication`对象的单例。其后，你可以通过调用`sharedApplication`类方法来获取这个对象。

应用的`application`对象的主要作用是处理传入的用户事件的初始路由。它将控制对象（`UIControl`类的实例）转发给他的消息分派给适当的目标对象。`application`对象维护一个打开的窗口列表（`UIWindow`对象）并且通过这些`window`对象可以检索任何应用的UIView对象。

`UIApplication`类定义了一个遵守`UIApplicationDelegate`协议的委托并实现了协议的一些方法。`application`对象会通知委托重大的运行时事件——例如，应用启动，低内存警告和应用销毁——给他一个适当的响应机会。

应用可以通过`openURL:`方法协作的处理一个资源，像邮件或者图片文件。例如，用邮件URL调用这个方法的应用会引起Mail应用启动并展示该消息。

这个类中的API允许你管理特定设备行为。使用`UIApplication`对象执行如下操作：

* 临时的挂起传入的触摸事件（`beginIgnoringInteractionEvents`）
* 注册远程通知（`registerForRemoteNotifications`）
* 确定是否有注册过已安装的应用处理URL scheme（`canOpenURL:`）
* 扩展应用执行用来在后台完成任务（`beginBackgroundTaskWithExpirationHandler:`,`beginBackgroundTaskWithName:expirationHandler:`）
* 调度并取消本地通知（`scheduleLocalNotification:`,`cancelLocalNotifications`）
* 协调远程控制事件的接收（`beginReceivingRemoteControlEvents`,`endReceivingRemoteControlEvents`）
* 执行应用级别的状态复位任务（在[管理状态复位行为]()任务组中的方法）

### 子类化说明
大多数应用都不需要子类化`UIApplication`。而是通过应用委托来管理系统和应用之间的交互。

如果应用一定要在系统处理之前处理传入事件——非常少见的情况——你可以实现一个自定义的事件或者行为调度机制。为了达到这个目的，需要子类化`UIApplication`并重写`sendEvent:`和/或`sendAction:to:from:forEvent`方法。对你拦截的每一个方法，在你处理完事件之后，通过调用`[super sendEvent:event]`将其分配回给系统。只有在需要的时候才拦截方法并尽可能的避免拦截。

## 主题
### 获取应用实例
---
#### sharedApplication 类方法
返回应用的单例

**声明**
```
@property(class, nonatomic, readonly) UIApplication *sharedApplication
```
**返回值**
在`UIApplicationMain`方法中创建的实例。
**讨论**
在启动时候`UIApplicationMain`方法创建的共享应用实例。

### 管理应用的行为
---
#### delegate
应用对象委托
**声明**
```
@property(nonatomic, assign) id<UIApplicationDelegate> delegate
```

**讨论**
每一个应用都必须有一个应用委托对象来响应应用相关的消息。例如，当应用完成启动、进入前台或后台执行状态改变的时候，应用会通知他的委托。相同的，来自系统的应用相关消息经常会路由到应用委托来处理。Xcode提供会为每一个应用提供初始的应用委托并且你不需要在后续对其进行改变。

这个委托一定要遵守`UIApplicationDelegate`正式协议。

#### UIApplicationDelegate
由`UIApplication`对象单例调用的方法集合，用以响应应用生命周期中的重要事件。

### 注册远程通知
---
`method` registerForRemoteNotifications 
注册用来接收通过Apple Push Notification服务发送的远程通知。
**声明**
```
- (void)registerForRemoteNotifications; 
```
**讨论**



