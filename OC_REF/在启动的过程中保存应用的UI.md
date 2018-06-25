【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# 在启动的过程中保存应用的UI
当应用被系统杀死后返回它的上一个状态。

## 概述
保存应用的用户界面有助于形成一种应用总是在运行的错觉。在iOS设备上中断会频繁发生，并且长时间的中断会导致系统为了清理资源而将应用杀死。然而，用户并不知道应用应经被杀死同时也不期望应用的状态发生改变。相反，他们希望应用的状态和他们离开时是一样的。状态保存和回复确保了当它在次启动的时候会返回它的上一个状态。

在适当的时机，UIKit会把应用视图和视图控制器的状态保存到硬盘上的编码文件中。当应用被杀死并重启后，UIKit会从保存的数据中重构你的视图和视图控制器。响应数据的保存和恢复会自动进行，但是你必须做一些具体的工作使其支持这些处理：

* 允许支持状态保存和恢复。
* 为你想保存的视图控制器分配恢复标识。
* 必要的话，在恢复的时候重新创建视图控制器。
* 编码和解码你需要恢复至上一个状态的视图控制器的自定义数据。

如果你的界面是完全由`storyboard`定义的话，UIKit自己知道如何重新创建视图控制器，并且这些工作都是自动进行的。如果你没有使用`storyboard`或者如果你想对视图控制器的创建和初始化有更强的掌控力的话，你可以自己创建。

### 为你的应用开启保存和恢复状态
你通过实现应用委托的`application:shouldSaveApplicationState:`和`application:shouldRestoreApplicationState:`方法来加入保存和恢复状态。俩方法的返回值都是布尔型的，它们指明了与其相关的处理是否发生并且大多数情况下你只是返回`YES`。然而，有时当你要恢复的应用界面不合适的时候，你可以返回`NO`。

当UIKit调用`application:shouldSaveApplicationState:`方法的时候，你可以保存数据同时返回`YES`。你可以保存你打算在恢复期间使用的数据。例如，表1展示了一个保存应用当前版本号的例子。在恢复的时候，`application:shouldRestoreApplicationState:`方法会检查村当中的版本号并且如果版本号与预期的不符会阻止恢复的进行。

表1 声明支持状态保存和恢复
```
func application(_ application: UIApplication, shouldSaveApplicationState coder: NSCoder) -> Bool {
	// Save the current app version to the archive
	coder.encode(11.0, forKey: "MyAppVersion")
	// Always save state information
	return true
}

func application(_ application: UIApplication, shouldRestoreApplicationState coder: NSCoder) -> Bool {
	// Restore the state only if the app version matches
	let version = coder.decodeFloat(forKey: "MyAppVersion")
	if version == 11.0 {
		return true
	}
	// Do not restore from old data
	return false
} 
```
如果你阻止了恢复的发生，你仍然可以在应用委托的`application:didFinishLaunchingWithOptions:`方法中手动的配置应用界面。

### 为视图控制器分配恢复标识
你可以通过为要保存的视图控制器分配恢复标识来明确的告诉UIKit你想要保存他们。一个恢复标识是一个你通过编程方式或者在IB中为视图控制器分配的唯一字符串。视图控制器的类名通常是一个合适的恢复标识，但是你可以使用任何字符串。在`storyboard`中为视图控制器添加字符串或者在运行时将其赋值给视图控制器的`restorationIdentifier`属性。

![图1](https://github.com/singmiya/translate/blob/master/datas/uikit_10.png)




