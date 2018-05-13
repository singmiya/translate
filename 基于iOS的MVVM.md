【原文】https://www.teehanlax.com/blog/model-view-viewmodel-for-ios/

# 基于iOS的MVVM

如果你开发iOS应用已经有一段时间了，你可能会听说过模型-视图-控制器（`Model-View-Controller`）或者`MVC`。这是你构建`iOS`应用的标准方案。然而，近来，我越来越受不了MVC中的一些缺点了。在这篇文章中，我打算旧事重提，看看什么是`MVC`？它的弱点具体是什么？此外还会向你介绍一种构建应用的新方式：模型-视图-视图模型（`Model-View-ViewModel`）。请拿出你的术语bingo卡片，因为我们即将有一个转换范例。

---

## 模型-视图-控制器（`Model-View-Controller`）
模型-视图-控制器（`Model-View-Controller`）是构建代码的最权威范例。在`MVC`中，所有的对象都被分类为模型、视图或者控制器。模型（`Model`）保存数据，视图（`View`）展示与用户交互的界面，视图控制器对模型（`Model`）和视图（`View`）之间的交互进行“居中调停”。
![MVC](/Users/csip/Documents/docs/notes/translate/datas/MVC.png)

![MVC](https://github.com/singmiya/translate/blob/master/datas/MVC.png)
在图表中，视图中的任何用户交互都会通知到控制器。然后，视图控制器会更新模型（`Model`）以反映状态的改变。然后，模型（通常是通过`Key-Value-Observation`）通知控制器去更新需要在其视图中展示的数据。这种协调构成了iOS应用的大量应用代码。

通常，模型（`Model`）对象非常的简单。很多时候，它们可能是`Core Data`中的[管理对象（`managed objects`）](https://developer.apple.com/documentation/coredata/nsmanagedobject)，或者，如果你不使用`Core Data`，你可以用其他流行的[模型层](https://github.com/Mantle/Mantle)。根据苹果官方文档描述，模型包含数据和操作数据的逻辑。事实上，模型通常都比较简单，不管好坏，模型处理逻辑都推给了控制器。

视图（`View`）可以是`UIKit`组件或者程序员自定义的`UIKit`组件集合。这些东西正是在可视化可交互的应用组件.xib或者Storyboard中使用的零件。正如你知道的按钮（`Buttons`）、标签（`Labels`）等。视图永远不会直接引用模型（`Model`），只能通过`IBAction`事件引用控制器。与视图无关的业务逻辑不应该出现在这里。

剩下就是控制器了。控制器是执行“胶水（`glue code`）”代码的地方，这些代码会协调模型（`Model`）和视图（`View`）之间的所有交互。控制器负责管理属于自己的视图的视图层次结构。他们负责响应视图加载，显示，消失等等。控制器也更倾向于处理置身于模型（`Model`）之外的模型逻辑和置身于视图（`View`）之外的业务逻辑。这把我们引向了MVC的第一个问题...

---

## 臃肿的视图控制器（`Massive View Controller`）
由于视图控制器中存在大量的代码，这往往会使它们变的极其臃肿。在iOS项目中，视图控制器中包含成千上万行代码的情况并不罕见。膨胀的视图控制器让你的应用不堪重负：难以维护（由于它们庞大的规模），其中数十个属性使它们的状态更难以管理，控制器逻辑同其遵守的众多协议的实现代码混合。

由于众多可能的状态，所以很难对臃肿的视图控制器进行测试，不管是手动还是单元测试。通常，把代码分成很小的片段是一个很好的方法。这样我想到了最近放生的[一件事](http://mikehadlow.blogspot.co.uk/2013/12/are-your-programmers-working-hard-or.html)。

---

## 缺少网络逻辑
`MVC`的定义——苹果公司也在使用——中声明：所有的对象都可以按照模型（`Model`）、视图（`View`）、控制器（`Controller`）进行分类。那么你们把网络逻辑代码放在哪里？与API进行通讯的代码在哪里？

你可以把它放在模型对象中，但这可能会变的棘手，因为网络调用应该是异步完成的，因此，如果网络比拥有的模型存活时间长，那么，它就变得复杂了。当然，你也不应该把网络逻辑代码放在视图中，那么就剩下控制器了。但是，这也不是一个好主意，因为，这加速了臃肿视图控制器问题的产生。

那么，应该放在哪里呢？MVC的这三个部分没有合适的地方放置网络逻辑代码。

---

## 可测试性差
MVC的另一个大问题是不利于程序员编程单元测试。因为，视图控制器中混合了视图处理逻辑和业务逻辑，为了单元测试而对这些组件进行分离就变的困难无比。很多人都忽略了这项工作...任何东西都不测试。

---

## “管理”的模糊定义
我之前提到过视图控制器可以管理视图层级结构；视图控制器存在一个“视图”属性，并且视图可以通过`IBOutlets`访问任何子视图。过多的`outlets`不利于对视图控制器进行扩展，在某种情况下，你最好使用子视图控制器帮助你管理所有的子视图。

什么情况下呢？什么时候对视图控制器中的视图进行分离对它有利呢？验证用户输入的业务逻辑是属于控制器还是模型呢？

这些界线很模糊，无法对其达成统一。不管你的界线如何划分，视图和响应控制器都是紧耦合的，总之，你可能都会会把它们当做一个组件。

现在有一个新的概念...

## 模型-视图-视图模型（`Model-View-ViewModel`）
![MVVM](/Users/csip/Documents/docs/notes/translate/datas/MVVM1.png
)
理论上，`MVC`能够很好的工作。然而，在现实中并非如此。现在，我们已经介绍过打破MVC经典使用的情况（MVC在实际使用中出现的问题），下面让我们看一种可选的架构模式：模型-视图-视图模型（`Model-View-ViewModel`）。

`MVVM`由[微软提出](http://msdn.microsoft.com/en-us/library/hh848246.aspx)，但不要排斥它。`MVVM`和`MVC`很相似。它形式化了视图和控制器紧耦合的特性并且引入了一个新的组件。

在`MVVM`中，视图和视图控制器正式的连接在了一起；我们把它们当做一个整体。视图仍然不会引用模型（`Model`），此外控制器也是如此。它们反而会引用视图模型（`View Model`）。

对于用户输入验证逻辑，视图展示逻辑，请求网络和其他杂项代码，视图模型是一个极佳的放置地方。唯一不属于视图模型的是视图自己的引用。`iOS`中视图模型的逻辑同样适用于`OSX`。（换句话说，只要不在视图模型中#import UIKit.h就不会出现问题）。

因为像映射模型中的值到格式化的字符串中这样的展示逻辑属于视图模型，这使得控制器变的非常简洁。在你开始使用MVVM时，最大的好处是，你只需要在视图模型中写很少的逻辑，并且随着你对范例的了解，你可以向其中添加更多的逻辑进去。

使用`MVVM`构建iOS应用有着很好的测试性；因为视图模型包含所有的展示逻辑并且不引用视图，它可以使用编程的方式对其进行全面的测试。测试核心数据模型会涉及到[许多](http://programming.oreilly.com/2013/05/upward-mobility-unit-testing-core-data.html)[黑客](http://www.cimgf.com/2012/05/15/unit-testing-with-core-data/)[行为](http://stackoverflow.com/questions/1876568/ocmock-with-core-data-dynamic-properties-problem)，但是使用`MVVM`构建的iOS应用可以对其进行全面的单元测试。

以我的经验，使用`MVVM`可能会稍微的增加你的代码量，但是会降低代码的复杂度。这个取舍很值得。

如果你在看一下`MVVM`结构图，你会注意到我在使用“notify”和“update”的时候很模糊，没有指出如何去做。你可以像在`MVC`中那样使用`KVO`，但是很快就会变得难以管理。在实际中，使用[ReactiveCocoa](http://www.teehanlax.com/blog/getting-started-with-reactivecocoa/)可以很好地把这些活动的组件连接在一起。

结合`ReactiveCocoa`如何使用`MVVM`，如果你想知道关于这些方面的更多信息，你可以阅读*Colin Wheeler‘s*的文章——[excellent write-up](http://cocoasamurai.blogspot.ca/2013/03/basic-mvvm-with-reactivecocoa.html)或者check out我写的[开源应用](https://github.com/AshFurrow/C-41)。你也可以阅读[我关于ReactiveCocoa和MVVM的书](https://leanpub.com/iosfrp)。






