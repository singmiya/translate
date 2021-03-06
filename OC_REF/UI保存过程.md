【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# UI保存过程
学习如何自定义UIKit状态的保存过程。

## 概述
图1描述了界面保存进行时的调用顺序。询问应用委托后，如果你想保存应用状态，UIKit会对当前视图控制器层级中的对象进行编码。仅当带有有效`restorationIdentifier`的视图控制器才会被保存。

![](https://github.com/singmiya/translate/blob/master/datas/uikit_11.png)

保存过程会遍历视图控制器层级结构并对找到的对象进行递归编码。处理从应用窗口的根视图控制器开始，这会将它们的数据写入到提供的归档文件中。如果根视图控制器的数据包含对其他视图控制器的引用，UIKit会要求每个新的视图控制器将其自己的数据编码到归档的单独部分。这些子视图控制器可能也会编码他们自己的子视图控制器，如此循环下去。

UIKit视图控制器会视情况而定的去自动对它的子视图控制器进行编码。如果你定义了一个自定义视图控制器容器，视图控制器的`encodeRestorableStateWithCoder:`方法同样的会把任何的子视图控制器写入到提供的归档文件中。

### 从保存中排除视图控制器
有两种方式从视图控制器（和它的视图）的保存中排除：

* 这个它的`restorationIdentifier`属性为`nil`。
* 提供恢复类并从`viewControllerWithRestorationIdentifierPath:coder:`方法中返回`nil`。

排除视图控制器可以防止将视图控制器保存进归档中。它也将视图控制器的子视图控制器的保存排除在外。

### 编码应用中的任何对象
状态恢复不仅限于应用的视图和视图控制器。任何遵守`UIStateRestoring`协议的对象都可以包含进恢复归档文件中。例如，你可能有一个遵守该协议的保存应用全局配置数据的对象。可以如下方式将其加入到归档文件中：

1. 当应用正在运行的时候，通过调用`UIApplication`的`registerObjectForStateRestoration:restorationIdentifier:`方法来注册对象。例如，你可以在创建它之后立即注册配置对象。
2. 在其中一个`encodeRestorableStateWithCoder:`方法中，把对象编码进归档文件。你也可以在应用委托的`application:willEncodeRestorableStateWithCoder:`方法中对其进行编码。

你可以对你自定义对象所需的任何数据进行编码，只要它在下一个启动周期时能将对象返回至它的上一个状态就行了。编码数据对应用的行为来说并不是至关重要的，并且不要对用其他方式持久化过的数据进行编码。例如，不要编码应用的设置或者用户数据，这些必须在启动周期之间保持不变。

