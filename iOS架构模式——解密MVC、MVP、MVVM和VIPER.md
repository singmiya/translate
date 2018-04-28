
【原文】https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52
#iOS架构模式——解密MVC、MVP、MVVM和VIPER

使用`MVC`进行`iOS`开发感觉到很怪异？在切换到`MVVM`的时候心存疑虑？听说过`VIPER`，但是不知道是否值得采用？
读下去，这篇文章将为你一一解惑。
如果你正打算组织一下在`iOS`环境下你掌握的架构模式知识体系。我们接下来回简单地回顾几个流行的架构并做几个小的练习。关于某个例子如果你想了解的更详细一些，可以查看下方的链接。

*掌握设计模式会让人沉迷其中，所有一定要当心：相比阅读本文章之前，你可能会问更多像这样的问题：*
*由谁进行网络请求：`Model`？还是`ViewController`?*

*如何向新视图(`View`)的`ViewModel`中传递`Model`*

*由谁创建一个新的`VIPER`模块：路由（`Router`）？还是展示器（`Presenter`）？*
<div align='center'>
*![thinking](/Users/csip/Documents/docs/notes/images/thinking.png)*
</div>

---

##为什么在乎架构的选择？
因为如果你不这样做，终有一天，你在调试一个拥有着数十个不同方法和变量（things）的庞大的类文件时，你会发现你无法找到并修复此文件中的任何问题。自然地，也很难把这个类文件当做一个整体而熟稔于心，这样你可能总是会错过一些重要的细节。如果你的应用已经处于这样的境况，很有可能是这样：

* 这个类是`UIViewController`的子类。
* 你的数据直接存储在`UIViewController`中。
* 你的`UIViews`什么都不做。
* `Model`是哑数据结构（`dumb data structure` ）。

> dumb data structure: 只用来存储数据的结构，没有任何方法。详见：https://stackoverflow.com/questions/32944751/what-is-dumb-data-and-dumb-data-object-mean

* 单元测试没有0覆盖。
即使你是按照苹果的指导方针并实现苹果的MVC模式，也会出现上述问题，所有不要难过。苹果的MVC模式存在着一些些问题，这点我们稍后再说。
让我们定义一下一个好的架构应该有的**特点**：

1. 能把代码职责均衡的解耦到不同的功能类里。（Balanced distribution of responsibilities among entities with strict roles.）
2. 可测试性（Testability usually comes from the first feature.）。
3. 易用、维护成本低（Ease of use and a low maintenance cost.）。

###解耦（ Why Distribution ?）
在我们试弄清楚事物是如何运作的时候，解耦可以保证大脑的负载均衡。如果你认为开发的（项目）越多你的大脑越能适应理解复杂的问题，那么你就是对的。但是这个能力不是线性扩展的并且很快就能达到上限。所以，解决复杂性的最简单的方式就是在多个实体间按照“[单一责任原则](https://en.wikipedia.org/wiki/Single_responsibility_principle)” 拆分职责。

###可测试（Why Testability ?）
对于那些已经习惯了单元测试的人来说这并不是一个问题，因为再添加了新的特性和重构了一个复杂的类后通常会运行失败。这意味着单元测试可以帮助开发者发现一些在运行时才会出现的问题，并且这些问题常见于安装在用户的手机上的应用上，此外要修复这些问题也需要[大概一周](http://appreviewtimes.com/)的时间。

###易用（Why Ease of use ?）
这个问题并不需要回答，但，值得一提的是：最好的代码总是那些没有被写出来的代码。因此，代码越少，错误也就越少。这也说明，总想着写最少代码的开发者不是因为他们懒，并且你也不应该因为一个更聪明的解决方案而忽视维护成本。

---

##MV(X)概要
现今，当我们提及架构设计模式的时候，我们有很多的选择。比如：

* `MVC`
* `MVP`
* `MVVM`
* `VIPER`

上述的前三个架构采取的是，把应用中的实体（entities）放入下面三个类别中其中一个内。

* **`Models`**——负责域数据（domain data）和操作数据的[数据访问层（Data access layer）](https://en.wikipedia.org/wiki/Data_access_layer)，可认为"**Person**"和"**PersonDataProvider**"类。
* **`Views`**——负责表现层（**GUI**），对于iOS环境来说就是所有以"**UI**"开头的类。
* **`Controller`**/**`Presenter`**/**`ViewModel`**——模型（`Model`）和视图（`View`）的粘合剂、中介，通常的负责通过响应用户在视图（`View`）上的操作通知模型（`Model`）更新、并通过模型的改变来更新视图（`View`）。

实体解耦能让我们：

* 更好的理解它们（这点我么已经知道）
* 复用（大多用于视图（`View`）和模型（`Model`））
* 独立测试

让我们想看一下`MV(X)`架构，之后再回过头来看`VIPER`

---

###MVC

####MVC前世

在讨论苹果的MVC架构时，先来看一下[传统的MVC](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller)是什么样的。
![Traditional MVC](/Users/csip/Documents/docs/notes/images/t_mvc.png)
在这种情况中，视图（`View`）是无状态的。一旦模型（`Model`）改变视图（`View`）仅仅只是被控制器（`Controller`）渲染而已。想象一下点击一个链接导航到其他地方后网页完全加载出来的情况。尽管，iOS应用可以使用传统的MVC架构，但这并没有多大意义，因为架构本身就存在问题——三个实体（`entities`）之间联系太过紧密，每一个实体都知道（引用）另外两个实体。这就导致了实体的复用性急剧下降——在你的应用中这并不是你所想要的。出于这个原因，我们就不写MVC范例了。

> Traditional MVC doesn't seems to be applicable to modern iOS development.
> 传统MVC架构看上去并不适合用于现在的iOS开发中。

###Apple's MVC

####预期
![Expectation MVC](/Users/csip/Documents/docs/notes/images/e_mvc.png)
控制器（`Controller`）是视图（`View`）与模型（`Model`）两者之间的中介，这使得视图（`View`）与模型（`Model`）都不知道对方的存在。控制器（`Controller`）是可复用的最少的，对我们来说这通常很好，因为我们必须有一个地方去放置一些不适合放在模型（`Model`）中且比较棘手的逻辑。
理论上，看起来非常简单，但是你感觉到有些地方不对，是不是这样？你甚至听说过人们把MVC称作Massive View Controller。此外，[视图控制器"瘦身"（View Controller offloading）](https://www.objc.io/issues/1-view-controllers/lighter-view-controllers/)成了iOS开发者中的一个重要话题。如果苹果只是采用传统`MVC`架构或者只是稍加改进，为什么会出现这种情况？

####MVC今生（现实情况）
![Realistic Cocoa MVC](/Users/csip/Documents/docs/notes/images/j_mvc.png)
`Cocoa MVC`鼓励你使用大型的视图控制器（**Massive** View Controllers），由于他们都参与到了视图（`View`）的生命周期中了以至于很难说他们是分离的。尽管你仍有能力分流一些业务逻辑和数据转换功能到模型（`Model`）中，但是当涉及到把工作分流到视图（`View`）中去时你就诶有更多的选择了，因为在大多数时候视图（`View`）的所有职责是把动作传递到控制器（`Controller`）中。视图控制器（`View Controller`）最终最为所有控件的委托和数据源，通常负责调度和取消网络请求...应有尽有

你见过多少次这样的代码：

```swift
var userCell = tableView.dequeueReusableCellWithIdentifier("identifier") as UserCell
userCell.configureWithUser(user)
```
`cell`这个视图是由Model直接配置数据的，因此这违反了`MVC`指南，但是这种情况无时无刻不在发生着，而且通常人们并不认为这样有什么错的。如果你严格的遵守`MVC`架构，那么你应该在`Controller`中配置cell数据，不用把`Model`传递到`View`中去，这会增加控制器的大小（复杂度）。

> `Cocoa` `MVC` is reasonably unabbreviated the Massive View Controller.
> `Cocoa` `MVC` 被称作大型视图控制器是合理的。

在未提及[单元测试（Unit Testing）](http://nshipster.com/unit-testing/)前`MVC`的问题并不是很明显（希望，你的项目中有单元测试）。由于你的视图控制器（`View Controller`）与视图（`View`）是紧耦合的，因此很难对其进行测试，因为你不得不非常有创造性的模拟视图和他们的生命周期，使用这种方式编写视图控制器（`View Controller`）代码，你要尽可能的把业务逻辑和视图布局代码分离开来。

让我们来看一个简单地`playground`例子：

```
import UIKit
struct Person { // Model
	let firstName: String
	let lastName: String
}
class GreetingViewController: UIViewController {// View + Controller
	var person: Person!
	let showGreetingButton = UIButton()
	let greetingLabel = UILabel()
	
	override func viewDidLoad() {
		super.viewDidLoad()
		self.showGreetingButton.addTarget(self, action: "didTapButton:", forControlEvent: .TouchUpInside)
	}
	func didTapButton(button: UIButton) {
		let greeting = "Hello" + " " + self.person.firstName + " " + self.person.lastName
		self.greetingLabel.text = greeting
	}
	
	// 布局代码在这儿
	......	
}
// 组合MVC
let model = Person(firstName: "David", lastName: "Blaine")
let view = GreetingViewController()
view.person = model
```
> MVC assembling can be performed in the presenting view controller
> 组合`MVC`可以在展示视图控制器（`presenting view controller`）中来完成

不是很容易测试，是不是这样？我们可以把生成`greeting`的代码放入到`GreetingModel`类里并单独的进行测试，但是，在没有直接调用与`UIView`有关的方法（如：`viewDidLoad`, `didTapButton`，这些方法可能会加载所有的视图，不利于单元测试。）的情况下，我们无法测试`GreetingViewController`中的任何展示逻辑（尽管上面的代码中没有太多这样的逻辑）。

事实上，在模拟器（如：`iPhone4s`）上加载并测试视图并不能保证在其他设备（如：`iPad`）上也能正常工作，所以，我建议从Unit Test目标（`Unit Test target`）配置中移除主应用程序（`Host Application`）并在模拟器上没有应用运行的情况下运行测试。

> The interactions between the **View** and the **Controller** aren't really testable with [Unit Tests](https://ashfurrow.com/blog/whats-worth-unit-testing-in-objective-c/).
> 视图和控制器之间的交互很难进行[单元测试](https://ashfurrow.com/blog/whats-worth-unit-testing-in-objective-c/)。

综上所述，`Cocoa MVC` 可能是一个相当糟糕的选择。让我们按照文章开头定义的特点来评估一下这种架构模式：

* **解耦（Distribution）**——视图（`View`）和模型（`Model`）确实解耦了，然而，视图（`View`）和控制器（`Controller`）却是紧密耦合的。
* **可测试（Testability）**——由于紧耦合的关系，你只能测试视图（`Model`）。
* **易用（Ease of use）**——同其他模式相比代码最少。此外，大家都熟悉它，因此，很用以掌握甚至是新手。

如果你没有打算在架构时耗费太多时间并且觉得高成本的维护费用对你的小项目来说是一种过度的浪费的话，那么`Cocoa MVC`就是你的最好选择。

> Cocoa MVC is the best architectural pattern in term of the speed of the development.
> 在开发速度上面`Cocoa MVC`是最好的架构模式。

---

###MVP
####Cocoa MVC’s promises delivered（==这句话不知道怎么翻译==）
![Passive View variant of MVP](/Users/csip/Documents/docs/notes/images/MVP.png)
是不是很像苹果的MVC架构？没错，确实如此，它就是`MVP`（Passive View variant）。等下...是不是`Apple’s MVC`事实上就是`MVP`？并不是，回想一下在`MVC`中`View`和`Controller`是紧密耦合的，然而，MVP的中介——`Presenter`与View Controller的生命周期没有任何关系，并且很容易模拟View，所以`Presenter`中没有任何布局代码，但是它却负责用数据和状态更新`View`。
![](/Users/csip/Documents/docs/notes/images/man.jpeg)

> ##### **What if i told you，the UIViewController is the View.**
> 我告诉你，视图控制器就是视图。

就`MVP`来说，`UIViewController`的子类实际上就是视图（`Views`）不是展示器（`Presenters`）。这点区别带来了极好的可测试性，而花费代价是开发速度，因为你不得不手工进行数据和事件绑定，就如你在下面的例子中看到的那样：
```
import UIKit
struct Person { // Model
	let firstName: String
	let lastName: String
}

protocol GreetingView: class {
	func setGreeting(greeting: String)
}

protocol GreetingViewPresenter {
	init(view: GreetingView, person: Person)
	func showGreeting()
}

class GreetingPresenter: GreetingViewPresenter {
	unowned let view: GreetingView
	let person: Person
	required init(view: GreetingView, person: Person) {
		self.view = view
		self.person = person
	}
	
	func showGreeting() {
		let greeting = "Hello" + " " + self.person.firstName + " " + self.person.lastName
		self.view.setGreeting(greeting)
	}
}
class GreetingViewController: UIViewController, GreetingView {
	var presenter: GreetingViePresenter!
	let showGreetingButton = UIButton()
	let greetingLabel = UILabel()
	
	override func viewDidLoad() {
		super.viewDidLoad()
		self.showGreetingButton.addTarget(self, action: "didTapButton:", forControlEvents: .TouchUpInside)
	}
	
	func didTapButton(button: UIButton) {
		self.presenter.showGreeting()
	}
	
	func setGreeting(greeting: String) {
		self.greetingLabel.text = greeting
	}
	
	// layout code go here
}

let model = Person(firstName: "David", lastName: "Blaine")
let view = GreetingViewController()
let presenter = GreetingPresenter(view: view, person: model)
view.presenter = presenter
```
####关于组装的重要提示
MVP是第一个揭示出组装问题（assembly problem）的架构模式，而出现这个问题的原因是它有三个实际上独立的层。由于我们不想让视图（`View`）了解模型（`Model`），所以在展示视图控制器（也就是视图）执行组装是不正确的，因此我们不得不在其他地方执行它。例如，我们可以创建一个app范围（app-wide）的路由（`Router`）服务，让它来完成执行组装和视图到视图（`View-to-View`）的展示功能。这个问题不止在`MVP`中存在，在下面介绍的其他模式中也存在。

让我们看一下`MVP`的特点：

* **解耦（Distribution）**——我们在最大程度上分离了展示器（`Presenter`）和模型（`Model`），还有相当简单、轻薄的视图（dumb view）（在上述例子中的模型也很简单）。
* **可测试性（Testability）**——很棒，由于简单的视图，我们可以测试大多数的业务逻辑。
* **易用性（Easy of use）**——在我们简单不完整的例子中，相比于MVC这些代码成倍的增加了，但是与此同时，MVP模式的思路却更加的清晰。

> MVP in iOS means superb testability and a lot of code
> `iOS`中的`MVP`架构意味着极好的可测试性和大量的代码。

####绑定和Hooters 
还有一种类型的`MVP`架构模式——the Supervising Controller MVP。这个变种包括了视图和模型的直接绑定，展示器（The Supervising Controller）在处理动作的同时还可以改变视图。
![Supervising Presenter variant of the MVP](/Users/csip/Documents/docs/notes/images/MVP1.png)
但是，就如我们已经知道的，模糊的职责拆分是不正确的，视图和模型的紧耦合也同样不可取。这和`Cocoa`桌面应用开发很相似。和传统的`MVC`一样，给有瑕疵的架构写例子没有任何意义。

---

###MVVM
####`MV(X)`类中近期最优秀的架构（The latest and the greatest of the MV(X) kind）
[**MVVM**](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93viewmodel)是MV(X)这类中最新的架构形式，所以，我们希望它能够解决`MV(X)`之前所面临的问题。

理论上，`Model-View_ViewModel`这种架构很棒。不仅`View`和`Model`，而且`Mediator`——相当于`View Model`，我们都已经熟悉。
![MVVM](/Users/csip/Documents/docs/notes/images/MVVM1.png)
它和`MVP`很相似：

* MVVM把视图控制器当做视图。
* 视图（`View`）和模型（`Model`）之间不存在紧耦合。

另外，它还可以像`MVP`那样绑定；但是绑定不是发生在视图（`View`）和模型（`Model`）之间，而是视图（`View`）和视图模型（`View Model`）之间。

那么，iOS现实中的视图模型（`View Model`）的庐山面目是什么？它是你的视图及其状态的基本的`UIKit`的独立展示。视图模型触发模型的改变，并利用改变后的`Model`更新自己，由于我们在视图和视图模型之间进行了绑定，视图也会根据视图模型的改变而改变。

####绑定（Bindings）
绑定我在讲解`MVP`架构部分简单的提到过，这里我们在对其进行一些讨论。绑定是从`OSX`开发而来的，而且iOS中并没有这个概念。当然，我们有`KVO`和通知（`notifications`），但是它的使用并没有绑定方便。

所以，倘若不想自己编写绑定代码，我们还有两个选择：

* 一个是，基于KVO的绑定库，像[`RZDataBinding`](https://github.com/Raizlabs/RZDataBinding)或者[`SwiftBond`](https://github.com/SwiftBond/Bond)。
* 完全的[函数式响应式编程](https://gist.github.com/JaviLorbada/4a7bd6129275ebefd5a6)野兽，像[`ReactiveCocoa`](https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0CB4QFjAAahUKEwj2l6rZv5jJAhUFUhQKHWahCKs&url=https%3A%2F%2Fgithub.com%2FReactiveCocoa%2FReactiveCocoa&usg=AFQjCNHM-pOkluiSuPsaVwVujCDTknVFUA&sig2=54zu-ATo8vDMvtXbxZYTvQ)、[`RxSwift`](https://github.com/ReactiveX/RxSwift/)或者[`PromiseKit`](https://github.com/mxcl/PromiseKit)`。

事实上，现今，只要你听到“`MVVM`”你就会想到`ReactiveCocoa`，反之亦然。尽管使用简单地绑定也可以创建`MVVM`架构的项目，但是，`ReactiveCocoa`（或者同类的库）却可以让你把使用`MVVM`架构的优势最大化。

关于`Reactive`库有一个残酷的现实需要面对：功能强大却伴随着巨大的职责。当使用`Reactive`库的时候极容易把很多事情搞混，如果出现错误，你可能需要花费很多的时间去在APP中定位问题所在，所以看一下下图的调用堆栈。
<div align="center">
	![Reactive Debugging](/Users/csip/Documents/docs/notes/images/stack1.png)
</div>


在简单的例子中，使用FRF（functional reactive function：函数式响应式函数）库甚至KVO都显得大材小用，相反我们显式使用*`showGreeting`*方法让视图模型（`View Model`）更新，并使用*`greetingDidChange`*回调函数这样一个简单地属性。

```
import UIKit
struct Person { //Model
	let firstName: String
	let lastName: String
}
protocol GreetingViewModelProtocol: class {
	var greeting: String? {get}
	// function to call when greeting did change
	var greetingDidChange: ((GreetingViewModelProtocol) -> ()) ? (get set)
	init(person: Person)
	func showGreeting()
}
class GreetingViewModel: GreetingViewModelProtocol {
	let person: Person
	var greeting: String ? {
		didSet {
			self.greetingDidChange?(self)
		}
	}
	var greetingDidChange: ((GreetingViewModelProtocol) -> ())?
	required init(person: Person) {
		self.person = person
	}
	func showGreeting() {
		self.greeting = "Hello" + " " + self.person.firstName + " " + self.person.lastName
	}
}

class GreetingViewController: UIViewController {
	var viewModel: GreetingViewModelProtocol! {
		didSet {
			self.viewModel.greetingDidChange = { 
			[unowned self] viewModel in self.greetingLabel.text = viewModel.greeting 
			}
		}
	}
	let showGreetingButton = UIButton()
	let greetingLabel = UILabel()
	
	override func viewDidLoad() {
		super.viewDidLoad（）
		self.showGreetingButton.addTarget(self.viewModel, action: "showGreeting", forControlEvents: .TouchUpInside)
	}
	// layout code goes here
}

let model = Person(firstName： "David", lastName: "Blaine")
let viewModel = GreetingViewModel(person: model)
let view = GreetingViewController()
view.viewModel = viewModel
```
再回过来看一下我们的特点评估：

* **解耦（Distribution）**——在我们上面的的简例中可能不太明显，事实上，MVVM的视图比`MVP`的视图拥有更多的职责。因为，前者通过绑定从视图模型（**`ViewModel`**）更新自己，而后者则是把所有的事件前置到`Presenter`中，也不对自己的状态进行更新。
* **可测试性（Testability）**——**`View Model`**对**`View`**一无所知，这可以让我们轻易地对其进行测试。也可以对视图（**`View`**）测试，但由于`UIKit`依赖，你可能想跳过她。
* **易用性（Easy of use）**——在我们的例子中，`MVVM`同`MVP`有同样的代码量，但是在实际的应用中，对于`MVP`你需要把所有事件从视图（`View`）前置到展示器（`Presenter`）并手动的更新视图，而对于`MVVM`，如果你使用了绑定则会变的很容易。

> MVVM极其吸人眼球，它融合了上述所有架构的的优势，此外，由于它在视图（`View`）端进行了绑定，你可以不需要任何额外的代码对视图（`View`）进行更新。虽然如此，可测试性依然保持在一个很好的层次。

---

###VIPER
####把搭建乐高积木的经验拿到iOS应用设计中使用
[`VIPER`](https://www.objc.io/issues/13-architecture/viper/)使我们最后的选择，这种架构尤为有趣，因为他不是属于MV(X)类的架构。

到目前为止，关于职责粒度的划分非常合理这点你肯定赞同。`VIPER`在职责划分上面又做了一次迭代，这次我们一共有五层。
![VIPER](/Users/csip/Documents/docs/notes/images/VIPER1.png)

* **交互器（Interactor）**——包含与数据（Entities）或者网络相关的业务逻辑，向创建一个实体的对象或者从网络获取对象。为了这个目的，你需要用到一些`Services`和`Managers`，这些不能算是VIPER的一部分，更确切的说只是些外部依赖。
* **展示器（Presenter）**——包含与UI相关（但是独立于UIKit）的业务逻辑，调用交互器（`Interactor`）中的方法。
* **实体（Entities）**——简单地数据对象，并不是数据访问层，因为数据访问是交互器（`Interactor`）的职责。
* **路由（Router）**——用来连接`VIPER`中的模块。

大致上说，`VIPER`模块可以是一个界面，也可以是整个应用的用户界面（user story）——想象一下验证功能，它可以是一个界面也可以是几个相关联的界面。”乐高积木“块应该多大呢？——这取决你自己。

如果我们同`MV(X)`这一类进行比较，我们可以看到几个不同的职责解耦之处：

* **模型（`Model`）**（数据交互（data interaction））逻辑转移到了**交互器**（**`Interactor`**）中，同时**实体（`Entities`）**作为单一的数据结构存在。
* 只有**控制器/展示器/视图模型**的UI展示责任转移到了**展示器（Presenter）**，而不是数据修改功能。
* **`VIPER`**是第一个明确的负责导航功能的架构模式，这点是通过**路由（`Router`）**来解决的。

在iOS应用中，寻一个适当的方式进行页面路由是一个具有挑战性的工作，而`MV(X)`这类模式只是简单的避而不谈。

这个例子没有涉及到路由和模块间的交互，因为，这些话题在`MV(X)`这类模式中也没有提及。
```
import UIKIt
struct Person { // 实体（通常要比这个复杂，例如：NSManagedObject）
	let firstName: String
	let lastName: String
}
struct GreetingData { // 传递数据结构（不是实体）
	let greeting: String
	let subject: String
}

protocol GreetingOutput: class {
	func receiveGreetingData(greetingData: GreetingData)
}

class GreetingInteractor: GreetingProvider {
	weak var output: GreetingOutput!
	func provideGreetingData() {
		let person = Person(firstName: "David", lastName: "Blaine")// 通常来自于数据访问层
		let subject = person.firstName + " " + person.lastName
		let greeting = GreetingData(greeting: "Hello", subject: subject)
		self.output.receiveGreetingData(greeting)
	}
}

protocol GreetingViewEventHandler {
	func didTapShowGreetingButton()
}

protocol GreetingView: class {
	func setGreeting(greeting: String)
}

class GreetingPresenter: GreetingOutput, GreetingViewEventHandler {
	weak var view: GreetingView!
	var greetingProvider: GreetingProvider!
	
	func didTapShowGreetingButton() {
		self.greetingProvider.provideGreetingData()
	}
	
	func receiveGreetingData(greetingData: GreetingData) {
		let greeting = greetingData.greeting + " " + greetingData.subject
		self.view.setGreeting(greeting)
	}
}

class GreetingViewController: UIViewController, GreetingView {
	var eventHandler: GreetingViewEventHandler!
	let showGreetingButton: UIButton()
	let greetingLabel = UILabel()
	
	override func viewDidLoad() {
		super.viewDidLoad()
		self.showGreetingButton.addTarget(self, action: "didTapButton", forControlEvents: .TouchUpInside)
	}
	
	func didTapButton(button: UIButton) {
		self.eventHandler.didTapShowGreetingButton()
	}
	
	func setGreeting(greeting: String) {
		self.greetingLabel.text = greeting
	}
	
	// layout code goes here
}

let view = GreetingViewController()
let presenter = GreetingPresenter()
let interactor = GreetingInteractor()
view.eventHandler = presenter
presenter.greetingProvider = interactor
interactor.output = presenter
```

再来看一下特点评估：

* **解耦（Distribution）**——毋庸置疑，VIPER架构在职责间解耦的表现最好。
* **可测试性（Testability）**——不足为奇，更好的解耦，更好的可测试性。
* **易用性（Easy of use）**——最后，上述两个的表现所花费的代价你已经猜出来了。你不得不写大量的没有多少职责的接口（interface）类。

####乐高积木提现在哪里呢？
当使用`VIPER`时，感觉就像用乐高积木搭建一座帝国大厦一样，这是一个有问题的信号。也许，对于你的应用来说现在使用`VIPER`架构还为时过早，你可以考虑一个简单的架构。有些人则选择忽略这个问题，还继续大炮打麻雀——大材小用。我猜测他们觉得未来他们的应用会因此而受益，尽管现在维护成本高的不合情理。如果你也这样想的话，我建议你试一下[Generamba](https://github.com/rambler-digital-solutions/Generamba)——一个可以生成`VIPER`架构的工具。尽管如此，对我个人来说，这样就像在使用有自动瞄准系统的大炮一样而不是简单地投石机。

---

##结论
我们已经看过了几种架构模式，我希望大家都能为各自的困惑找到答案，毫无疑问你会意识到这篇文章并没有提供什么高招，所以，选择架构模式的关键是根据具体的情况进行权衡、取舍。

因此，在同一个应用中出现架构混合是很正常的一件事。例如：你一开始用的是`MVC`架构，突然你意识到有一个特定的界面很难再用`MVC`架构进行有效的维护了，然后你就把它转换成了`MVVM`架构而且仅仅只是对这一个界面进行了转换。对于其他的界面如果`MVC`架构工作正常的话没有必要进行重构，因为这两个架构很容易兼容。

>####Make everything as simple as possible, but not simpler——Albert Einstein
>尽可能的简化一切，但并不简单——阿尔伯特·爱因斯坦

