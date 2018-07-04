【译】为避免撕逼，提前声明：本文纯属翻译，仅仅是为了学习，加上水平有限，见谅！

# UI恢复过程
学习如何自定义UIKit状态的恢复过程。

## 概述
图1描述了从应用启动到恢复完成期间发生的调用顺序。恢复发生在应用初始化的过程中，并且只有当恢复状态归档可用且应用委托的`application:shouldRestoreApplicationState:`方法返回`YES`的时候它才继续进行。

图1 界面恢复顺序
![](https://github.com/singmiya/translate/blob/master/datas/uikit_12.png)

恢复过程的第一步是为你的界面创建视图控制器对象（显示的或者隐式的）。第二步是解码并恢复这些对象的状态。这两步都需要重新创建视图控制器的层级。例如，当导航控制器和它的子视图控制器创建后，这些对象并不会立即关联在一起。实际上是导航控制器的`decodeRestorableStateWithCoder:`方法重新创建了它与其子视图控制器的关系。

当恢复状态完成后，UIKit会调用应用委托的`application:didFinishLaunchingWithOptions:`方法。使用该方法对你的界面做任何一次最后的修改或者添加。例如，可能会向视图控制器层级中添加登录界面。

### 重新创建视图控制器
再回复期间，UIKit试图创建或者从保存的界面中定位视图控制器对象。UIKit首先会要求你提供视图控制器对象。如果你没有提供视图控制器，UIKit会隐式的寻找它。下面是UIKit重新创建视图控制器的步骤顺序：

1. **询问视图控制器的恢复类。**一个恢复类知道如何创建一个特定的视图控制器。你通过向视图控制器的`restorationClass`属性赋值这个类指定恢复类。在恢复期间，UIKit会调用恢复类的`viewControllerWithRestorationIdentifierPath:coder:`方法去请求视图控制器的一个新实例，然后方法会返回该实例。如果你返回`nil`，UIKit会停止创建视图控制器并且脱离恢复过程。
2. **询问应用委托。**如果视图控制器没有恢复类，UIKit会调用应用委托的`application:viewControllerWithRestorationIdentifierPath:coder:`方法。如果这个方法返回`nil`，UIKit会继续查找。
3. **检查存在的对象。**UIKit会按照完全一样的恢复路径查找已创建过的视图控制器。
4. **从storyboard中初始化视图控制器。**如果它仍然没有视图控制器，UIKit会自动从应用的storyboard中初始化它。

甚至在状态恢复之前，UIKit会从storybo中加载应用的默认视图控制器。由于UIKit可以零耗费的加载这些视图控制器，最好不要使用恢复类和应用委托创建它们。对于所有的其他视图控制器，仅在storyboard没有定义视图控制器时才分配恢复类。你可能也会为视图控制器分配恢复类以防特殊情况。例如，如果关联的恢复归档引用过时或缺失数据，你可能会避免展示视图控制器。

当用代码重新创建视图控制器的时候，除了其他任何初始化之外，始终将值重新赋值给视图控制器的`restorationIdentifier`属性。在适当的时候将值赋值给`restorationClass`属性。在创建时间分配这些值确保视图控制器可以在下一个循环中将其保存。
```
func viewController(withRestorationIdentifierPath 
                    identifierComponents: [Any], 
                    coder: NSCoder) -> UIViewController? {
                    
   let vc = MyViewController()
   vc.restorationIdentifier = identifierComponents.last as? String
   vc.restorationClass = MyViewController.self
   return vc
}
```
> **Note**
> 你的恢复类应该始终由UIKit返回期望的类。恢复归档包含每一个保存的视图控制器的类。如果恢复类返回了不同的类实例，UIKit不会调用视图控制器的`decodeRestorableStateWithCoder:`方法。


