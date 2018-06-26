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


在恢复的期间，UIKit会试图保存引用窗口的根视图控制器。对于每一个带有恢复标识的根视图控制器，UIKit会要求视图控制器把它的自定义数据编码进归档文件中。一个视图控制器容器可以编码对子视图控制器的引用并将其作为自定义数据的一部分。如果确实如此，并且这些视图控制器都有恢复标识的话，UIKit会试图保存子视图控制器和他们的内容。这个处理会按照从一个视图控制器另一个的连接继续递归下去，直到视图控制器被全部保存或者忽略掉。

你不需要为每一个视图控制器都分配恢复标识。事实上，有些时候你可能不想保存所有的视图控制器。例如，如果应用展示一个临时的登录界面，你可能不想对其进行保存。与此相反，你可能想要在恢复的时候确认是否需要展示它。这种情况下，你可以不必为登录界面的视图控制器分配恢复标识。

更多关于保存什么的详细信息，请看[UI保存过程](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UI%E4%BF%9D%E5%AD%98%E8%BF%87%E7%A8%8B.md)

### 为应用编码、解码自定义信息
在保存的过程中，UIKit会调用每一个要保存的视图和视图控制器的`encodeRestorableStateWithCoder:`方法。使用这个方法可以保存将视图或视图控制器返回到其当前状态所需要的信息。

* 切记要保存关于视图和控制器的可视化状态。
* 切记要保存指向你想要保存子视图的引用。
* 切记不要包含已经持久存储的数据。而是，包含你可以在稍后定位到这个数据的标识符。

状态保存不是向硬盘保存应用数据的替代物。UIKit可以灵活的丢弃状态保存数据，允许应用返回到默认的状态。使用保存过程来保存应用的用户界面状态信息，像当前table选中的行。切记不要使用它去存储table中包含的数据。

表2展示了一个带有获取姓名输入框的视图控制器。如果其中一个文本框包含了没有保存的值，方法会将其保存并标识那个一个文本框包含这个值。在这种情况下，未保存的值不是应用持久化数据的一部分；它是一个在必要的时候是可以丢弃的值。

表2 编码视图控制器的状态
```
override func encodeRestorableState(with coder: NSCoder) {
	super.encodeRestorableState(with: coder)
	// Save the user ID so that we can load that user later。
	coder.encode(userID, forKey: "UserID")
	// Write out any temporary data if editing is in progress.
	if firstNameField!.isFirstResponder {
		coder.encode(firstNameField?.text, forKey: "EditedText")
		coder.encode(Int32(1), forKey: "EditField")
	} else if lastNameField!.isFirstResponder {
		coder.encode(lastNameField?.text, forKey: "EditedText")		coder.encode(Int32(2), forKey: "EditField")
	} else {
		// No editing was in progress.
		coder.encode(Int32(0), forKey: "EditField")
	}
}
```
更多关于UIKit如何保存应用的视图，视图控制器和状态信息的资料，请看[UI保存过程](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UI%E4%BF%9D%E5%AD%98%E8%BF%87%E7%A8%8B.md)

### 当需要的时候创建视图控制器
当应用启动的时候，如果保存的状态信息可用，系统会尝试使用保存过的信息恢复应用的界面。

1. UIKit会调用应用委托的`application:shouldRestoreApplicationState:`方法来决定恢复是否应该进行。
2. UIKit使用应用的`storyboard`去重新创建你的视图控制器。
3. UIKit调用每一个视图控制器的`decodeRestorableStateWithCoder:`方法来恢复它的状态信息。

UIKit首先会从`storyboard`中加载视图控制器和它的视图。在这些对象加载并初始化完成后，UIKit开始恢复他们的状态信息。使用`decodeRestorableStateWithCoder:`方法将视图控制器返回值它上一个状态。

表3展示了解码方法。这个方法从保存的用户id中恢复视图控制器的数据。如果一个文本框正在编辑中，这个方法也会恢复处理中的数据并将相应的文本框设置为第一响应者，这时就可以为这个文本框展示键盘。

表3
```
override func decodeRestorableState(with coder: NSCoder) {
	super.decodeRestorableState(with: coder)
	// Restore the first name and last name from the user ID
	let identifier = coder.decodeObject(forKey: "UserID") as! String
	setUserID(identifier: identifier)
	
	// Restore an in-progress values that was not saved
	let activeField = coder.decodeInteger(forKey: "EditField")
	let editedText = coder.decodeObject(forKey: "EditedText") as! String?
	switch activeField {
	 case 1:
	 	firstNameField?.text = editedText
	 	firstNameField?.becomeFirstResponder()
	 	break
	 case 2:
	 	lastNameField?.text = editedText
	 	lastNameField?.becomeFirstResponder()
	 	break
	 default:
	 	break
		
}
```
在`storyboard`中定义视图控制器是管理恢复状态的最简方式，但是不是唯一的方式。更多关于用其他方式重新创建视图控制器的方法，请看[UI恢复过程](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UI%E6%81%A2%E5%A4%8D%E8%BF%87%E7%A8%8B.md)

## 话题
### 过程细节
---
#### [UI保存过程](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UI%E4%BF%9D%E5%AD%98%E8%BF%87%E7%A8%8B.md)
学习如何自定义UIKit状态的保存过程。

#### [UI恢复过程](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UI%E6%81%A2%E5%A4%8D%E8%BF%87%E7%A8%8B.md)
学习如何自定义UIkit状态的恢复过程。


