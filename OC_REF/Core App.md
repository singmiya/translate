【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# Core App
管理应用数据模型和其与系统的交互。

## 话题
### 应用
---
#### [管理应用生命周期](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/%E7%AE%A1%E7%90%86%E5%BA%94%E7%94%A8%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.md)
理解应用委托如何管理应用高层次的行为。

#### [UIApplication](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIApplication.md)
应用在iOS系统上运行的控制和协调的中心点。

#### [UIApplicationDelegate](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIApplicationDelegate.md)
由UIApplication单例对象调用的方法集，来响应应用生命周期中的重要事件。

#### [UIApplicationMain](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIApplicationMain.md)
创建应用对象和应用委托并设置事件循环。

**Declaration**
```
int UIApplicationMain(int argc, char * _Nonnull *argv, NSString *principalClassName, NSString *delegateClassName);
```
**参数**
argc
在`argv`中的参数个数；这通常是`main`函数的对应参数。

argv
参数的对象列表；这通常是`main`函数的对应参数。

principalClassName
`UIApplication`类或其子类的名称。如果你指定为`nil`，则默认为是`UIApplication`

delegateClassName
实例化应用委托的类名称。如果`principalClassName`指定为`UIApplication`的子类，你可能也要指定其子类作为委托；子类实例来接收应用委托消息。如果你要从应用主`nib`文件中加载委托对象，则要将其指定为`nil`。

**返回值**
尽管这个方法指定返回一个整型值，但实际上它从不返回。当用户点击`Home`按钮退出iOS应用的时候，应用会进去后台运行。

**讨论**
这个方法实例化来自主类的应用对象，实例化给定类的的委托（如果有的话）并为应用设置委托。它也会创建一个主事件循环，其中包含应用运行循环，并开始处理事件。如果在应用`Info.plist`文件中通过设置`NSMainNibFile`键并把有效的`nib`文件名称作为其对应的值这种方式来指定加载的主`nib`文件，那这个方法会加载这个`nib`文件。

尽管声明了返回类型，但这个方法根本不会有值返回。如果想知道这个方法更多行为信息，请看[iOS应用编程指南](https://developer.apple.com/library/etc/redirect/xcode/content/1189/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40007072)中的[Expected App Behaviors](https://developer.apple.com/library/etc/redirect/xcode/content/1189/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/ExpectedAppBehaviors/ExpectedAppBehaviors.html#//apple_ref/doc/uid/TP40007072-CH3)

**可用版本**
iOS 2.0+
tvOS 9.0+

### 设备环境
---
#### [响应Apple TV展示模式变化](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/%E5%93%8D%E5%BA%94Apple%20TV%E6%98%BE%E7%A4%BA%E6%A8%A1%E5%BC%8F%E7%9A%84%E5%8F%98%E5%8C%96.md)
当设备的屏幕尺寸改变时，相应的图片和资源也会动态的改变。

#### [UIDevice](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIDevice.md)
代表当前的设备

#### [UITraitCollection](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UITraitCollection.md)
应用的iOS界面环境，由像水平和竖直尺寸类，展示比例和用户界面习惯等这些特征定义。

#### [UIAdaptivePresentationControllerDelegate](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIAdaptivePresentationControllerDelegate.md)
一系列与展示控制器配合使用的方法，确定如何响应应用的特征变化。

### 文档
---
#### [UIDocument](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIDocument.md)
管理应用数据分散部分的抽象基类。

#### [UIManagedDocument](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIManagedDocument.md)
与Core Data交互的文件管理对象。

### 剪切板
---
#### [UIPasteboard](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIPasteboard.md)
帮助用户在同一个应用的不同地方或不同的应用间分享数据的对象。

#### [UIPasteConfiguration](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIPasteConfiguration.md)
对象需要实现的接口，来为粘贴和拖放行为定义接受指定数据类型的能力。

#### [UIPasteConfigurationSupporting](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIPasteConfigurationSupporting.md)
确定响应对象是否支持粘贴配置的接口。

### 数据管理
---
#### [保护用户隐私](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/%E4%BF%9D%E6%8A%A4%E7%94%A8%E6%88%B7%E9%9A%90%E7%A7%81.md)
通过加固用户数据并尊重用户期望自己的数据被如何使用的意愿来保护用户的隐私。

#### [使用自定义URLs与其他应用通信](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89URL%E4%B8%8E%E5%85%B6%E4%BB%96%E5%BA%94%E7%94%A8%E9%80%9A%E4%BF%A1.md)
通过特定的格式化URLs接收来自其他应用的数据。

#### [UIDataSourceModelAssociation](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIDataSourceModelAssociation.md)
定义接口的一组方法，用来为应用中数据对象提供持久引用。


### 活动
---
#### [UIActivity](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIActivity.md)
你用来实现应用特有服务的抽象类的子类。

#### [UIActivityViewController](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIActivityViewController.md)
你用来为你的应用提供标准服务的视图控制器。

#### [UIActivityItemSource](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIActivityItemSource.md)
activity 视图控制器用来检索操作数据项而使用的一组方法。

### 引导式访问
---
#### [UIGuidedAccessRestrictionDelegation](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIGuidedAccessRestrictionDelegate.md)
你用来为iOS引导式访问特性添加自定义约束的一组方法。

#### [UIGuidedAccessRestrictionStateForIdentifier](https://github.com/singmiya/translate/blob/uikit_translate/OC_REF/UIGuidedAccessRestrictionStateForIdentifier.md)
为指定的引导式访问约束返回约束状态

**声明**
```
UIGuidedAccessRestrictionState UIGuidedAccessRestrictionStateForIdentifier(NSString *restrictionIdentifier);
```

**参数**
restrictionIdentifier
唯一标识引导式访问约束的字符串

**返回值**
引导式访问约束的当前状态。所有约束的初始状态是`UIGuidedAccessRestrictionStateAllow`

**可用版本**
iOS 7.0+
tvOS 9.0+


