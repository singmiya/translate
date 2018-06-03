【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# 关于使用UIKit进行应用开发
学习关于UIKit和Xcode为iOS和tvOS应用提供的基本支持。

## 概述
`UIKit`框架提供了你构建iOS或tvOS应用所需要了核心对象。使用这些对象可以在屏幕上展示内容，与这些内容进行交互并管理与系统的交互。应用在他们的基本行为上依赖于`UIKit`，并且`UIKit`还提供了许多方法让你可以自定义适合自己特殊需求的行为。

> **important**
> 开发iOS或tvOS应用总是从在Xcode——苹果集成的开发环境，中创建一个项目开始。如果你没有Xcode，你可以从App Store中下载。你也可以从[developer.apple.com](https://developer.apple.com/)下载最新的版本。

`Xcode`为每一个你创建的应用提供了作为起始点的工程模板。例如，图1展示了用Xcode中单视图模板创建的应用结构。工程模板提供了最简的用户界面，所以你可以立即构建运行工程并在真机或模拟器上查看运行结果。

![图一](/Users/csip/Documents/docs/notes/translate/datas/uikit_1.png)

当你构建应用的时候，Xcode会编译源文件并为你的工程创建应用程序包（app bundle）。应用程序包（app bundle）是一个包含代码和与应用有关资源的结构化目录。资源包括图片资源，storyboards文件，字符串文件和为代码提供的应用元数据。应用程序包（app bundle）的结构很重要，但Xcode知道要把资源放在那里，所以目前不需要担心。

### 必要资源
每一个`UIKit`应用都必须有以下资源：

* 应用图标（App icons）
* 启动画面storyboard（Launch screen storyboard）

系统会把你的应用图标展示在主屏上面，设置页面和任何需要区别于其他应用的地方。由于他需要在多个地方，多个设备上使用，所以你需要在Xcode工程的AppIcon图片资源中提供不同的应用图标版本，正如图2所展示的那样。你的应用图标应该设计的别具一格，这样可以帮助你在主屏上迅速识别出你的应用。然而，你可能需要修改图标的细节以让它适合你必须提供的不同图片尺寸。

![图2](/Users/csip/Documents/docs/notes/translate/datas/uikit_2.png)

LaunchScreen.storyboard文件包含应用的初始用户界面，它可以是一个启动画面（splash screen）或一个实际界面的简化版。当用户点击应用的图标时，系统会立即显示启动界面，让用户知道你的应用正在启动。当应用初始化的时候，启动界面也可以给它提供封面。当应用完成初始化时，系统会隐藏启动界面，并显示应用的真实界面。

### 必须的应用元数据
系统从应用程序包中的属性列表（Info.plist）文件中导出关于应用的配置和权限信息。Xcode为每一个新工程模板提供此文件的预配置版本，但有时你可能会修改这个文件。例如，你的应用依赖特定的硬件或者需要使用特定的系统框架，你可能需要在这个文件中添加与这些特性相关的信息。

你对`Info.plist`文件进行的最常见的一项修改是声明应用的硬件和软件需要。这些需求是你如何向系统说明你的应用要运行什么。例如，一个导航应用可能需要使用到GPS硬件来提供分路段导航，如图3所示。应用商店会阻止应用安装到不满足应用要求的设备上。
![图3](/Users/csip/Documents/docs/notes/translate/datas/uikit_3.png)
更多关于`Info.plist`文件中key的信息，请看[Information Property List Key Reference](https://developer.apple.com/library/etc/redirect/xcode/content/1189/documentation/General/Reference/InfoPlistKeyReference/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009247)

### UIKit应用的代码结构
`UIKit`提供很多应用核心对象，包含那些与系统交互，运行应用主事件循环以及在屏幕上展示内容的对象。对于大部分对象你可能都是原封不动的使用或者仅仅做很小的调整。知道修改那个对象，什么时候修改对实现你的应用很重要。

`UIKit`应用的结构基于模型-视图-控制器（`MVC`）设计模式，其中对象是按照用途划分的。模型（`Model`）对象管理应用的数据和业务逻辑。视图（`View`）对象提供数据的视觉展示。控制器（`Controller`）对象在模型和视图对象之间充当桥梁的角色，适时的在两者之间传输数据。

图4展示了一个很经典的`UIKit`应用结构。你提供展示用用数据结构的模型对象。`UIKit`日工大部分的试图对象，尽管你可以根据需要为你的数据自定义视图。你的视图控制器和应用委托对象负责协调数据对象和UIKit视图之间的数据交换。
![图4](/Users/csip/Documents/docs/notes/translate/datas/uikit_4.png)

`UIKit`和`Foundation`框架提供很多你可以用来定义应用模型对象的基本类型。`UIKit`提供UIDocument对象来组织属于磁盘文件的数据结构。`Foundation`框架定义基本的对象来展示字符串，数字，数组和其他数据类型。`Swift Standar Library`提供很多在`Foundation`框架中可用的相同类型。

`UIKit`提供了应用中控制器和视图层中的很多对象。特别的，`UIKit`定义了`UIView`类，它通常用来在屏幕上展示你的内容，（你也可以使用`Metal`和其他系统框架直接在屏幕上渲染内容。）`UIApplication`对象来运行应用的主事件循环并管理应用的全部生命周期。


