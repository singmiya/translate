#objc:Issue13——Architecture
## 编辑推荐
距objc.io第一期的出现已经有一年了，我们正在庆祝我们的一周年！感谢在此期间所有支持我们的朋友，特别是那些让我们从社区获得的卓越贡献的人。

你肯能和我们一样正为苹果上周在WWDC发布的一系列以开发者为中心的声明感到不知所措。让我们开心的是今年苹果的保密协议也有所松动，这意味着我们不必等到秋天再写这些。

在我们深入讨论新东西之前，这个月我们为你准备了一个更永恒的话题。我们想会过来整理一下我们写过的第一期文章：[更轻的视图控制器（lighter view controllers）](https://www.objc.io/issues/1-view-controllers/)。但是这次我们选择一个范围更广的话题，这期的文章会涉及各种不同的问题，而这些问题可能会是你在思考应用架构的时候遇到的。

上个月，我们有机会和一个在柏林UIKonf的有趣的开发团队坐在一起对这个话题进行头脑风暴：
![uikonf meeting](/Users/csip/Documents/docs/notes/images/issue13/uikonf-meeting.jpg)
头脑风暴的结果是五篇分别对应不同架构问题的文章：由*Ash Furrow* 编写的《`MVVM`概念》，由*Stephen Poletto*编写的《避免单例滥用》，由*Krzystof Zablłocki*编写的《用IB模块化行为（modular behaviors with Interface Builder）》，最后一个是，由*Conrad Stoll*和*Jeff Gilbert*编写的有别于传统`MVC`的架构——`VIPER`。

All the best from a very summery Berlin,

Chris, Daniel, and Florian.


## `MVVM`简介——by *Ash Furrow*
2011年我从500px得到了我的第一份iOS开发工作。在大学我已经做了几年iOS外包开发了，但是这是我第一份真正的iOS开发工作。我作为唯一的iOS开发者被雇佣去开发设计精美的iPad应用。仅仅7周我们就发布了`1.0`版本然后继续迭代，添加更多的功能，本质上讲，让代码库更加复杂。

有事，我都不知道我在做什么东西。像其他好的程序员一样，我知道自己的设计模式，但是我对产品架构决策的效率评估太接近客观了（ but I was way too close to the product I was making to objectively measure the efficacy of my architectural decisions. ）。随着另一个人加入到团队，然我意识到我们陷入到麻烦中了。

听说过`MVC`？也有人称之为Massive View Controller。那是当时的感觉。令人尴尬的细节就不再说了，但是如果说能再重来一次的话，我会做出不同的决定。

自此，我做的一个关键架构的改变而且在应用开发中用就到了它，那就是使用一个称之为`Model-View—ViewModel（MVVM）`的`MVC`。

`MVVM`究竟为何物呢？而非关注`MVVM`出现的历史背景，让我们典型的`iOS`应用是什么样的，并从中推出`MVVM`：
![MVVM](/Users/csip/Documents/docs/notes/images/issue13/mvvm1-16d81619.png)
从上图我们看到了`MVC`的架构图。模型（`Model`）展示数据，视图展示用户界面，控制器协调两者之间的交互。Cool！

思考一下，尽管视图和控制器在技术上是不同的组件，但是他们总是成双成对，形影不离的。视图最后一次匹配不同视图控制器（`View Controller`）是什么时候？反之亦然。因此为什么不形式化他们之间的连接？
![MVC](/Users/csip/Documents/docs/notes/images/issue13/intermediate-5287a0c6.png)
这可以更准确地描述你已经编写的`MVC`代码。但是它并不能解决应用程序中笨重的试图控制器（Massive View Controller）继续笨重下去的趋势。在标准的`MVC`应用程序中，很多逻辑被放在了视图控制器（`View Controller`）中处理。当然有些是属于视图控制器（`View Controller`）的，但是很多并不属于，这些在`MVVM`术语中被称为展示逻辑，如把一些值转换成可以在视图中战士的对象，如把一个NSDate对象转换成格式化的NSString对象。

从上图可以看到我们漏掉了一些东西。在这里我们可以里面放置展示逻辑。我打算把它叫做视图模型（`View Model`），它位于`view/controller`和`model`之间：
![](/Users/csip/Documents/docs/notes/images/issue13/mvvm-b27768df.png)

看起来好了很多！这幅图准确地描述了什么是`MVVM`：增强版的`MVC`，通过`MVVM`我们正式的连接了视图（`view`）和控制器（`controller`），把展示逻辑从从控制器移出到了视图模型（`view model`）中。`MVVM`听起来很复杂，但本质上讲，它是你已经熟悉的`MVC`架构的精心改良版。

现在我们已经知道`MVVM`是什么了，但是为什么有人会用它呢？在iOS中，对于我来说，`MVVM`的驱动力是他可以减少视图控制器（`view controller`）的复杂度，并且使得展示逻辑更容易测试。我们通过例子来看一下它是如何达成目标的。

我希望你能从这篇文章学到的有三个重要的方面：

* `MVVM`兼容你已存在的`MVC`架构。
* `MVVM`让你的应用更容易测试。
* `MVVM`配合绑定机制使用最佳。

正如我们之前看到的，本质上讲MVVM仅仅是MVC的精心改良版，因此，很容易看到它是如何被整合到一个具有标准`MVC`架构的现有应用程序中去。创建一个简单地`Person`模型（`Model`）和对应的视图控制器（`View Controller`）：

```
@interface Person: NSObject
- (instancetype)initwithSalutation:(NSString *)salutation firstName:(NSString *)firstName lastName:(NSString *)lastName birthdate:(NSDate *)birthdate;

@property (nonatomic, readonly) NSString *salutation;
@property (nonatomic, readonly) NSString *firstName;
@property (nonatomic, readonly) NSString *lastName;
@property (nonatomic, readonly) NSDate *birthdate;

@end
```

Cool.假设我们有一个PersonViewController，在viewDidLoad方法中基于model的属性仅仅设置一些labels：
```
- (void)viewDidLoad {
	[super viewDidLoad];
	if (self.model.salutation.length > 0) {
		self.nameLabel.text = [NSString stirngWithFormat:@"%@ %@ %@", self.model.salutation, self.model.firstName, self.model.lastName];
	} else {
		self.nameLabel.text = [NSString stringWithFormat:@"%@ %@", self.model.firstName, self.model.lastName];
	}
	NSDateFormat *dateFormatter = [[NSDateFormatter alloc] init];
	[dateFormatter setDateFormat:@"EEEE MMMM d, yyyy"];
	self.birthdateLabel.text = [dateFormatter stringFromDate:model.birthdate];
}
```
这是很简单`MVC`架构。现在让我们看一下如何用一个视图模型（`View Model`）扩展它：

```
@interface PersonViewModel: NSObject

- (instancetype)initWithPerson:(Person *)person;

@property (nonatomic, readonly) Person *person;
@property (nonatomic, readonly) NSString *nameText;
@property (nonatomic, readonly) NSString *birthdateText;

@end
```
下面就是这个模型（`Model`）的实现方式：
```
- (instancetype)initWithPerson:(Person *)person {
	self = [super init];
	if (!self) return nil;
	_person = person;
	if (person.salutation.length > 0) {
		_nameText = [NSString stringWithFormat:@"%@ %@ %@", self.person.salutation, self.person.firstName, self.person.lastName];
	} else {
		_nameText = [NSString stringWithFormat:@"%@ %@", self.person.firstName, self.person.lastName];
	}
	NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"EEEE MMMM d, yyyy"];
    _birthdateText = [dateFormatter stringFromDate:person.birthdate];
    
    return self;
}

@end
```
Cool.我们把viewDidLoad中的展示逻辑移到了视图模型（`View Model`）中。现在viewDidLoad方法就显得非常轻量级。
```
- (void)viewDidLoad {
	[super viewDidLoad];
	self.nameLabel.text = self.viewModel.nameText;
	self.birthdateLabel.text = self.viewModel.birthdateText;
}
```

正如你看到的，与`MVC`架构相比改变不大。同样的代码，只是把它移来移去而已。`MVVM`兼容`MVC`，形成了`lighter view controllers`，并且更容易测试。

可测试性？这是什么？众所周知，由于视图控制器（`View Controller`）中处理的东西太多导致很难对它进行测试。在`MVVM`架构中，我们试图把尽可能多的代码移到了视图模型（`View Model`）中。由于视图控制器（`View Controller`）处理的东西减少，从而使得它的测试更容易，同时视图模型（`View Model`）也变得极易测试。让我们看一下：
```
SpecBegin(Person)
	NSString *salutation = @"Dr.";
	NSString *firstName = @"first";
	NSString *lastName = @"last";
	NSDate *birthdate = [NSDate dateWithTimeIntervalSince1970:0];
	
	it(@"should user the salutation available. ", ^{
		Person *person = [[Person alloc] initWithSalutation:salutation firstName:firstName lastName:lastName birthdate:birthdate];
		PersonViewModel *viewModel = [[PersonViewModel alloc] initWithPerson:person];
		expect(viewModel.nameText).to.equal(@"Dr. first last");
	});
	
	it(@"should not use an unavailable salutation. ", ^{
		Person *person = [[Person alloc] initWithSalutation:nil firstName:firstName lastName:lastName birthdate:birthdate];
		PersonViewModel *viewModel = [[PersonViewModel alloc] initWithPerson:person];
		expect(viewModel.nameText).to.equal(@"first last");
	});
	it(@"should use correct date format. ", ^{
		Person *person = [[Person alloc] initWithSalutation:nil firstName:firstName lastName:lastName birthdate:birthdate];
		PersonViewModel *viewModel = [[PersonViewModel alloc] initWithPerson:person];
		expect(viewModel.birthdateText).to.equal(@"Thursday January 1, 1970");
	});
SpecEnd
```
如果没有把这部分逻辑移到视图模型（`View Model`）中，如果要对其进行测试，就不得不实例化完整的视图控制器（`View Controller`）及视图（`View`），同时比较视图（`View`）上标签中的值。这样不仅测试起来不方便，而且测试结果也没有说服力。现在我们可以随意的改变视图层级而不必担心破坏单元测试。使用`MVVM`所带来的测试上的好处是显而易见的，尽管是这样一个简单地例子，并且这种效果会随着展示逻辑的复杂变的越来越明显。

注意在上述的简例中，模型（`Model`）是不可变的，所以我们可以在初始化的时候设置模型（`Model`）的属性值。对于可变的model，我们需要使用一种绑定机制，以保证当支持这些属性的模型（`Model`）改变时，视图模型（`View Model`）也会跟着更新。此外，一旦视图模型（`View Model`）中的模型（`Model`）发生改变，视图（`View`）中的属性也需要更新。模型（`Model`）改变需要通过视图模型（`View Model`）向下传递至视图（`View`）。

在OSX系统中，可以使用`Cocoa` 绑定，但是在iOS系统中没有这种奢侈品。因此，键值监听（`KVO，Key-value observation`）就进入我们的视线，而且效果很棒。然而，即使是一个简单`KVO`绑定也需要很多样板代码，更别说当有很多属性需要绑定的时候了。所以，我喜欢使用`ReactiveCocoa`，但是并没有强制要求在MVVM中使用`ReactiveCocoa`。`MVVM`是一个很好的范例，它可以独立运行，并且只有好的绑定框架与其配合才能表现的更加完美。

我们已经说了很多：从简单地`MVC`得到`MVVM`，知道它们如何兼容范例，从可测试性看`MVVM`，了解到当`MVVM`和绑定机制配合时效果最好。如果你想知道`MVVM`的更多信息，可以查看这个[博客](https://www.teehanlax.com/blog/model-view-viewmodel-for-ios/)，它更详细的阐述了MVVM的好处，或者[这篇文章](https://www.teehanlax.com/blog/krush-ios-architecture/)，它是关于我们如何把`MVVM`应用在最近的工程中并取得巨大成功的。我也有一个基于`MVVM`的开源应用——[C-41](https://github.com/AshFurrow/C-41)，我对它进行了完全测试。你可以从git上把它pull下来，如果有什么问题可以[告诉我](http://twitter.com/ashfurrow)。

---

## 避免单例滥用——by *Stephen Poletto*
单例是整个`Cocoa`使用的核心设计模式之一。事实上，苹果的开发库把单例当做“`Cocoa`核心竞争力”之一。作为iOS开发者，从`UIApplication`到`NSFileManager`，我们对与单例的交互已经很熟悉了。在开源项目、苹果代码示例和*StackOverflow*中，我们见到过的单例已多如牛毛。甚至，Xcode还有默认的代码片段，如：”Dispatch Once“，这使得你往代码中添加单例变的非常的简单：
```
+ (instancetype)sharedInstance {
	static dispatch_once_t once;
	static id sharedInstance;
	dispatch_once(&once, ^{
		sharedInstance = [[self alloc] init];
	});
	return sharedInstance;
}
```
因为这些原因，单例在iOS编程中就很常见。但问题是，它很容易被滥用。

其他人把单例称作‘反面模式’，‘邪恶’和‘病态骗子’，然而我并没有完全抹去单例的价值。相反，我想论证单例的几个问题，从而，让你在下次打算自动完成`dispatch_once`代码片段的时候再三思考这样做可能带来的后果。

### 全局状态
大多数开发者都认为可变的全局状态是不可取的。有状态性使程序难以理解和调试。在最小化有状态代码方面，面向对象程序员有很多东西需要从函数编程上面学习。
```
@implementation SPMath {
	NSUInteger _a;
	NSUInteger _b;
}
- (NSUInteger)computeSum {
	return _a + _b;
}
```
在上述简单数学库的实现中，在调用`computeSum`方法之前程序员希望为实例变量`_a`和`_b`设置合适的值。这存在几个问题：

1. `computeSum`方法没有通过把`_a`和`_b`的值作为参数而显式的指出方法依赖于上述的两个值。其他阅读代码的人必须通过检查实现去理解依赖关系，而不是通过检查接口并理解哪些变量控制函数输出。隐藏依赖关系这样是不好的。
2. 当为了准备调用`computeSum`而修改`_a`和`_b`的时候，程序员需要确定这些修改不会影响其它依赖这些变量的代码的正确性。这在多线程环境尤为困难。

把这下面这个例子与上述的例子比较一下：
```
+ (NSUInteger)computeSumOf:(NSUInteger)a plus:(NSUInteger)b {
	return a + b;
}
```
这里方法对`a`和`b`的依赖就很明显。为了调用这个方法我们不需要改变实例的状态。我们也不必担心由于调用此方法而导致的持久的副作用，我们甚至可以把这个方法当做类方法，以表明我们调用此方法不需要修改实例状态。

但是，这个例子和单例有什么关系呢？用*Miško Hevery*的话说，“单例是披着羊皮的全局状态。”单例可以使用在任何地方，而不用明确的声明依赖关系。就像`computeSum`方法中的`_a`和`_b`没有明确的依赖关系一样，程序的任何模块都可以调用`[SPMySingleton sharedInstance]`并使用单例。这意味着与单例交互的任何副作用都会影响到程序的任何地方的任何代码。
```
@interface SPSingleton: NSObject

+ (instancetype)sharedInstance;
- (NSUInteger)badMutableState;
- (void)setBadMutableState:(NSUInteger)badMutableState;

@end

@implementation SPConsumerA
- (void)someMethod {
	if([[SPSingleton sharedInstance] badMutableState]) {
		//...
	}
}
@end

@implementation SPConsumerB
- (void)someOtherMethod {
	[[SPSingleton sharedInstance] setBadMutableState:0];
}

@end
```
在上述的例子中，`SPConsumerA`和`SPConsumerB`是程序中两个完全独立的模块。然而`SPConsumerB`可以通过单例提过的共享状态影响`SPConsumerA`的行为。在不使用单例的情况下，只有在消费者B中引入消费者A，明确两者之间的关系才能达到上述这样的效果。在单例中，由于它的全局有状态的性质，导致了看似两个不相关的模块之间的隐藏和隐式的耦合。

让我们看一个更具体的例子，并提出另外一个由全局可变状态而引起的问题。假设我们想在我们的应用中创建一个web查看器。为了支持这个web查看器，我们创建了一个简单地URL缓存:
```
@interface SPURLCache

+ (SPURLCache *)sharedURLCache;
- (void)storeCacheResponse:(NSCachedURLResponse *)cachedResponse forRequest:(NSURLRequest *)request;

@end
```
编写web查看器的开发者开始写几个单元测试，以保证代码在期望的几个不同的情况下能够正常工作。首先，写一个测试程序保证web查看器在没有设备连接的时候会显示一个错误。然后，写一个测试程序保证web查看器可以适当的处理服务器错误。最后，为简单地成功情况写一个测试程序，保证返回的web内容能被适当的展示出来。开发者运行所有的测试程序，并且它们会像预期的那样工作。Nice！

几个月后，这些测试程序开始失败，尽管web查看器的代码自从第一次写过后在没有进行任何更改！发生了什么？

结果是有人改变了测试程序的执行顺序。成功情况的测试首先执行，其次是另外的两个。现在失败的情况以外的成功了，因为整个测试是通过单例`URL`缓存对结果进行缓存的。

持久状态是单元测试的死敌，因为单元测试是由每个测试的相对立而产生的。如果状态从一个测试保留到下一个测试，然后，测试的执行循序突然就变的重要了。Buggy测试，特别是当测试应该失败的时候而它反而成功了，这不是一个好现象。

### 对象生命周期
单例的另外一个主要的问题是他们的生命周期。当向你的代码中添加添加单例时，很容易想到“只存在这样的一个。”但是，我在自己项目之外看到的大部分iOS代码中，这个假设都有可能失效。

例如，假设我们要创建一个能看见用户好友列表的应用。他们的每一个好友都有一个头像，并且我们想让应用把这个照片下载下来并把它缓存到设备上。使用`dispatch_once`代码片段很方便，但我们可能会发现自己正在编写一个`SPThumbnailCache`单例：
```
@interface SPThumbnailCache: NSObject
+ (instancetype)sharedThumbnailCache;
- (void)cacheProfileImage:(NSData *)imageData forUserId:(NSString *)userId;
- (NSData *)cachedProfileImageForUserId:(NSString *)userId;
@end
```
我们继续开发这个应用，并且看起来一切正常，直到某一天，当我们决定是时候实现“log out”函数了，这样就可以在应用中切换用户了。突然，我们出现了一个难以处理的问题：特定用户的状态保存到了全局的单例中了。当用户退出登录，我希望能够把磁盘上的持久状态清除掉。否则，我们会在用户设备上遗留下孤立数据，从而浪费宝贵的磁盘空间。万一，用户退出后转用另一个账户登录，我们同样希望能够为新用户创建一个新的`SPThumbnailCache`单例。

这里的问题是，根据定义，单例被假定为“创建一次，永远存活”的实例。对于上述的问题你可能会想到好几个解决方案。也许当用户退出登陆的时候我们可以把单例实例销毁掉：
```
static SPThumbnailCache *sharedThumbnailCache;
+ (instancetype)sharedThumbnailCache {
	if(!sharedThumbnailCache) {
		sharedThumbnailCache = [[self alloc] init];
	}
	return sharedThumbnailCache;
}

+ (void)tearDown {
	sharedThumbnailCache = nil;
}
```
这是明目张胆的对单例模式的滥用，但是很管用对不对？

我们当然可以让这个解决方案起作用，但是代价太大了。举例来说，我们已经失去了`dispatch_once`方案的简单性，并且这解决方案可以保证线程安全，所有的代码都调用`[SPThumbnailCache sharedThumbnailCache]`这个方法只是获取同一个实例。对于使用缩略图缓存的代码的执行顺序，我们需要格外的小心。假设在用户退出登陆的过程中，有一些保存图片到缓存的后台任务正在执行：
```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	[[SPThumbnailCache sharedThumbnailCache] cacheProfileImage:newImage forUserId:userId];
});
```
我们需要确定在后台任务执行完之前不能执行`tearDown`方法。这保证`newImage`数据能够正确的清除掉。或者，我们需要保证当缩略图缓存被清除的时候能把后台任务取消。否者，新的缩略图缓存将被懒创建并且旧用户状态（也就是`newImage`）将被存储到它里面。

因为，单例实例没有明显的所有者（例如：单例自己管理声明周期），所以，‘关闭’单例就变得非常困难。

就因为这点，我希望你说，“缩略图缓存就不应该使用单例的！”问题是在项目刚开始并不能完全理解对象的生命周期。对于一个具体的例子，`Dropbox`iOS应用仅仅支持单用户的登陆。直到有一天，当我们允许多用户（个人用户和企业账户）同时登陆时，应用在单用户登陆这种情况下已经存在好几年了。突然，假定“同一时刻只允许一个用户登录”开始闪退了。通过假设一个对象的生命周期匹配你的应用的生命周期，你将会限制你的代码的扩展性，并且当产品需要改变的时候你需要为此付出代价。

这里的教训是，单例应该保存为全局的状态，而不是在某一个范围内。如果把状态限制在任何一个比“应用完整生命周期”短的会话范围内，这个状态则不应该被单例管理。管理特定用户状态的单例是“代码异味”，你应该审慎的重新评估你的对象图的设计。

### 避免（使用）单例
所以，如果单例对于范围化的状态如此的不利，那如何避免使用它们呢？

重新看一下上面例子。由于我们有一个缓存特定个体用户状态的缩略图缓存，让我们定义一个用户对象：
```
@interface SPUser:NSObject
@property (nonatomic, readonly) SPThumbnailCache *thumbnailCache;
@end

@implementation SPUser
- (instancetype)init {
	if((self = [super init])) {
		_thumbnailCache = [[SPThumbnailCache alloc] init];
	}
	return self;
}
@end
```
现在我们有一个对象可以模拟授权的用户会话了，我们可以把所有的特定用户状态存储在这个对象内。现在，假设我们有一个渲染了好友列表的视图控制器。
```
@interface SPFriendListViewController: UIViewController
- (instancetype)initWithUser:(SPUser *)user;
@end
```
我们可以明确地把授权的用户对象传递到视图控制器中。这种传递依赖到独立的对象中的技术的一个更为正式的名字叫[依赖注入（dependency injection）](https://en.wikipedia.org/wiki/Dependency_injection)，并且他有一大堆的好处：

1. 它能够让阅读此接口的人清楚的明白：当用户登陆的时候`SPFriendListViewController`才会显示出来。
2. 只要`SPFriendListViewController`在使用它就可以保持用户对象的强引用。例如，更新先前的例子，我们可以使用下面的后台任务把图片保存到缩略图缓存。

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	[_user.thumbnailCache cacheProfileImage:newImage forUserId:userId];
});
```
即使这个后台任务仍然没有完成，应用中其他地方的代码也可以创建并使用全新的`SPUser`对象，而不需要阻塞进一步的交互因为第一个实力已经被销毁了。

为了进一步证明第二点，让我们想象一下使用依赖注入前后的对象图。

假设，我们的`SPFriendListViewController`是当前窗口的根视图控制器。在单例对象模型中，我们有如下如这样的一个对象图：
![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 5.21.20 AM-5a14d6af.png)

视图控制器和自定义图片视图列表与`sharedThumbnailCache`交互。当用户退出，我们希望清空更试图控制器并把用户带入登录界面。
![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 5.21.20 AM-5a14d6af.png)

问题是，好友列表试图控制器可能仍然在执行代码（由于后台操作），因此，仍会有未结束的调用挂起`sharedThumbnailCache`方法。

把这解决方案同使用依赖注入的解决方案对比：
![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 5.21.20 AM-5a14d6af.png)

假设，为简单起见，`SPApplicationDelegate`管理`SPUser`实例（事实上，你可能想会想着把用户状态的管理拆分到里一个对象里面以保持你的应用代理更轻）。当列表视图控制器被安装到了窗口上后，用户对象的引用也被传了进去。这个应用也会顺着对象图到个人图片视图。现在，当用户退出时，我们的对象图想起来是这样的：

![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 5.54.07 AM-8275c2ed.png)

这个对象图看起来和我们使用单例的情况没有什么区别。所以有什么严重的问题？

问题是作用域。在单例情况下，`sharedThumbnailCache`在程序中的任何模块都是可用的。假设，用户快速的登录一个新的账户。新用户想看他的好友，这意味着又一次和缩略图缓存交互：
![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 5.59.25 AM-87634b36.png)
当用户使用新账户登陆时，我们应该可以重新构建并与全新的`SPThumbnailCache`进行交互，而不必关心旧缩略图缓存的销毁。根据对象管理的标准规则，旧的视图控制器和缩略图缓存应该在后台自动清理。简言之，我们应该把用户A的状态和用户B的状态隔离开来：
![](/Users/csip/Documents/docs/notes/images/issue13/Screen Shot 2014-06-02 at 6.43.56 AM-524a953d.png)

### 结论
这篇文章没有什么新颖的东西。人们对单例的抱怨已经存在多年，而且也知道全局的状态非常不好。但是在iOS开发的领域，单例已司空见惯，以至于有时会忘记多年来从其他地方的面向对象编程习得的教训。

所有这一切的关键是，在面向对象编程中，我们希望最小化可变状态的作用域。单例站在了这种情况的对立面，因为它能让可变状态从程序中的任何地方获取到。下一次在你想要使用单例的时候，我希望你考虑一下依赖注入作为替代。

---

## 在iOS应用中的行为——by *Krzysztof Zabłocki*
作为开发者，我们力求编写整洁且组织良好的代码。达到这个目的我们有很多模式可以使用，其中最好的一个当属组合（composition）模式了。组合更容易让我们遵循**单一职责原则（`Single Responsibility Principe`）**并简化类文件。

不像重视图控制器（Massive View Controller）同时服务于不同的角色（如数据源和委托）那样，你可以把这些角色分离到不同的类文件中。视图控制器只负责配置它们并协调工作。毕竟，代码写的越少，需要调试和维护的代码也就越少。

### 那么行为究竟是什么？
行为是一个负责实现特定角色的对象，例如，你可以有一个实现视差动画的行为。

这篇文章中的行为将利用`Interface Builder`限制代码量，同时与非编码人员进行有效的合作。然而，如果你不使用`Interface Builder`你也可以使用行为，仍然可以获取大多数的好处。

很多行为除了设置之外不需要任何额外的代码，你完全可以通过`Interface Builder`和代码（同样的设置方法）完成这些设置。很多情况下，你甚至不需要用属性引用它们。

### 为什么使用行为？
许多iOS项目最后都成了重视图控制器（Massive View Controller）类，因为人们把应用中80%的逻辑放到了这里。这是一个严重的问题，因为试图控制器是我们代码中复用率最少的部分，很难对他们进行测试和维护。

行为可以帮我们避免这种情形，那么它给我们带来了那些好处呢？

### 轻视图控制器（Lighter View Controller)
使用行为意味着可以吧很多代码从控制器中移入到分离的类文件中。如果使用行为，通常都会形成轻量的视图控制器。例如，我的通常少于100行代码。

### 代码复用
因为行为仅仅只负责一个角色，它很容易避免特定行为和特定应用之间逻辑的依赖。这允许你在不同的应用之间共享香味。

### 测试性
行为是像黑盒一样工作的小类。这意味着他们很容易被单元测试覆盖。你甚至不用创建真实的视图只用提供模拟对象就可以测试它们。

### 让非编码人员修改应用逻辑的能力
如果我们决定使用`Interface Builder`利用行为，我们可以教设计师如何修改应用逻辑。设计师可以添加、移除行为和修改参数，并不需要知道Objective-C的任何相关知识。

这对工作流来说是很大的一个好处，特别是小团队。

### 如何创建灵活的行为？
行为是一个不需要任何专门代码的简单对象，但是这有几个概念可以真正的帮助它们更易使用，更加强大。

### 运行时属性
很多开发者都忽视了`Interface Builder`，甚至都没有学习过它，同样的，经常会错过它真正的强大之处。

运行时属性是使用`Interface Builder`的关键特性之一。它们给你提供一种设置自定义类的方式，设置可以为iOS的内建类设置属性。例如，你是否设置过图层的圆角半径？你可以直接在`Interface Builder`中为其简单地设置运行时属性：
![](/Users/csip/Documents/docs/notes/images/issue13/cornerRadius-d9b86675.png)
当在`Interface Builder`中创建行为的时候，你打算严重依赖运行时属性去设置行为选项。结果，这儿通常会有更多的运行时属性：
![](/Users/csip/Documents/docs/notes/images/issue13/runtimeAttributes-0db8153b.png)
### 行为的生存时间
如果一个对象从`Interface Builder`创建，它会立即创建立即移除，除非其他的对象拥有它的强引用。然而，这对行为来说并不是完美的，因为它需要同它作用于的视图控制器存活的时间一样长。

你可以在视图控制器中创建一个属性并强引用行为，但是这并不完美，原因有如下几个：

* 当创建并配置好行为后，你不需要同很多行为进行交互。
* 创建属性仅仅只为了保持对象生存这很麻烦。
* 如果需要移除特定的行为，你需要去清理那些未使用的属性。

### 使用Objective-C运行时去反转生存时间绑定
而不是手动的在视图控制器中设置行为的强引用，如果需要的话，我们可以把行为当做视图的一个关联对象，作为配置过程的一部分。

这意味着，在某种情况下如果我们需要移除特定的行为，我们只需要移除配置这个行为的代码或者`Interface Builder`对象，并不需要额外的改变。

这可以像下面那样实现：
```
@interface KZBehavior: UIControl
@property(nonatomic, weak) IBOutlet id owner;
@end

@implementation KZBehavior
- (void)setOwner:(id)owner {
	if(_owner != owner) {
		[self releaseLifetimeFromObject:_owner];
		_owner = owner;
		[self bindLifetimeToObject:_owner];
	}
}

- (void)bindLifetimeToObject:(id)object {
	objc_setAssociatedObject(object, (__bridge void *)self, self, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (void)releaseLifetimeFromObject:(id)object {
	objc_setAssociatedObject(object, (__bridge void *), nil, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
```
这里我们利用关联对象来创建特定所有者对象的强引用。

### 行为事件
让行为能够发布事件非常有用，例如，当一个动画结束时。可以通过从`UIControl`继承行为来在`Interface Builder`中启用该功能。然后，特定的行为可以调用：
```
[self sendActionsForControlEvents:UIControlEventValueChanged];
```
这允许你把行为连接到视图控制器的代码上。

### 一个简单行为的例子
因此，什么东西最容易作为行为来实现？

下面是将视差动画添加到UIViewController类（不是自定义类）的简单方法：
/Users/csip/Documents/docs/notes/images/issue13/parallaxAnimationBehaviour.mov
是否曾经从你的照片库和相机选择一张图片？
/Users/csip/Documents/docs/notes/images/issue13/imagePickerBehaviour.mp4


### 更为高级的特性
上面的行为很简单，但是，当我们需要更高级的特性的时候，以是否香菇应该怎么做？行为可以做到你想让他有多强大它就有多强大，我们看几个更为复杂的例子。


如果行为需要几个像UIScrollViewDelegate这类委托的话，你很快就会看到在一个特定的界面上你最多只能有一个行为的情况。但是，我们可以通过实现一个简单地多路代理的对象来处理这种问题：
```
@interface MutiplexerProxyBehavior: KZBehavior
@property(nonatomic, strong) IBOutletCollection(id) NSArray *targets;
@end

@implementation MutiplexerProxyBehavior
- (NSMethodSignature *)methodSignatureForSelector:(SEL)sel {
	NSMethodSignature *sig = [super methodSignatureForSelector:sel];
	if(!sig) {
		for(id obj in self.targets) {
			if((sig = [obj methodSignatureForSelector:sel)) {
				break;
			}
		}
	}
	return sig;
}
- (BOOL)respondsToSelector:(SEL)aSelector {
	BOOL base = [super respondsToSelector:aSelector];
	if(base) {
		return base;
	}
	return [self.targets.firstObject respondsToSelector:aSelector];
}
- (void)forwardInvocation:(NSInvocation *)anInvocation {
	for(id obj in self.targets) {
		if([obj respondsToSelector:anInvocation.selector]) {
			[anInvocation invokeWithTarget:obj];
		}
	}
}
@end
```
通过创建一个多路行为的实例，你可以把它指定为滚动视图的一个委托（或者任何一个有委托的对象），这样委托调用将会转发给所有人。

### 结论
行为是一个有趣的概念，它可以简化你的代码库，允许你在不同的应用间复用很多代码。它们还将允许你通过微调或者修改应用中的行为，从而同团队中的非编码人员有效的协作。

---

## 子类化——by *Chris Eidhof*
这篇文章和我通常写的文章有所不同。它不是一个指南更像是一系列想法和模式。我将要讲述的差不多所有模式都是通过犯错误这样艰难的方式找出来的。绝不是说我就是子类化方面的权威，但是，我只是想把我学到的一些事情分享出来。不要把它作为一个权威指南，而是一个例子集合。

当被问及面向对象编程时（`OOP`），*Alan Kay*（创造者）写到，他不是关于类的，而是消息传递^[?](http://c2.com/cgi/wiki?AlanKayOnMessaging)。 但，仍然有很多人关注于创建类层次。在这篇文章中，我们将看几个有用的案例，但是我们主要关注创建复杂的类层次结构。在我们的经验中，这会让代码更加简单，更容易维护。关于这个话题有很多文章，你可以在[Clean Code](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)和[Code Complete](http://www.amazon.com/Code-Complete-Practical-Handbook-Construction/dp/0735619670)这样的书中找到，推荐大家阅读这两本书。

### 什么时候子类化
首先，让我们讨论一下在哪些情况下创建子类是有意义的。如果你正在构建一个有自定义布局的`UITableViewCell`，那么你需要创建一个子类。几乎所有的观点都是如此；一旦你开始布局，把它移到子类里就变得有意义了，这样你不仅优雅的整理了你的代码，还让代码在整个项目都可复用。

假设，你的代码是针对不同的平台和版本的，你需要为每个平台和版本编写自定义的部分。这时创建一个`OBJDevice`类就有意义了，它可以子类化出`OBJIphoneDevice`和`OBJIPadDevice`，甚至可以更深层的子类化出`OBJIPhone5Device`，它们可以重写特定的方法。例如，你的`OBJDevice`类可以包含`applyRoundedCornersToView:withRadius:`方法。它有一个默认的实现，但是这个方法可以被特定的子类重写。

另外一种非常有用的子类化情形是在模型对象（`model object`）中。绝大多数时候，模型对象从类中继承的实现方法有`isEqual:`、`hash`、`copyWithZone:`和`description`。这些方法通过对属性迭代实现一次，很难出错。（如果你想找这样的基类，可以考虑用Mantle，它正事这样做的，而且不仅仅如此。）

### 什么时候不子类化
我参与过很多的项目，我也见过深层次的子类化。很惭愧我也这样做过。除非层次很浅，否则很快就会达到极限。

幸运的是，如果你发现自己正处于这样深的层次，这有很多可选方案。在下面的小节中，我们会深入分析每一部分。如果你的子类只是共享相同的接口和协议，这是一个很好的选择。如果你知道一个对象需要修改很多地方，你可能需要使用委托动态的改变和配置它。当你想要对已存在的对象扩展一些简单地功能时，类别（`category`）可能是一个选择。当你有一系列子类，每一个都重写（`override`）同样的方法，你可能会使用配置对象。最后，当你想重用一些功能时，最好组合多个对象而不是扩展他们。

### 可选方案
#### 方案：协议
通常的，一个使用子类的原因是你想要保证一个对象响应一个确定的消息。设想一下，在一个应用中你有一个可以播放视频的播放器（`player`）对象。现在，如果你想增加`YouTobe`的支持，你需要同样的接口，但是不同的实现。一种是用子类实现的方式如下：
```
@interface Player: NSObject
- (void)play;
- (void)pause;
@end

@interface YouTobePlayer: Player
@end
```
也许，这两个类并没有共享太多代码，仅仅是是相同的接口。在这种情况下，使用协议可能是一个更好的解决方案。使用协议，你可能会写下如下代码：
```
@protocol VideoPlayer <NSObject>
- (void)play;
- (void)pause;
@end

@interface Player: NSObjet <VideoPlayer>
@end

@interface YouTobePlayer: NSObject <VideoPlayer>
@end
```
这种情况下，`YouTobePlayer`不需要知道`Player`的内部情况。

#### 方案：委托
再一次假设，你有一个像上面例子中那样的`Player`类。现在，在一个地方，你想要在`paly`方法中执行一个自定义动作。做到它相当的容易：你可以创建一个自定义子类，重写`paly`方法，调用`[super paly]`，然后执行自定义的功能。这是一种处理方式。另外一种是改变`Player`对象并给他一个委托。例如:
```
@class Player;
@protocol PlayerDelegate
- (void)playerDidStartPlaying:(Player *)player;
@end

@interface Player: NSObject
@property (nonatomic, weak) id<PlayerDelegate> delegate;
- (void)play;
- (void)pause;
@end
```
现在，在播放器的`play`方法中，委托获得了`playerDidStartPlaying:`消息。这个类的任何使用者只需实现委托协议而不需要创建子类，并且`Player`对象依然保持着通用性。这是一个非常强大的技术，苹果在他们自己的框架中大量的使用了该技术。有时，你想把不同的方法组织到不同的协议中，如UITableView做的那样，它不仅有委托还有数据源。

#### 方案：类别（Category）
有时，你可能想对一个对象进行一点功能扩展。假设，你想向NSArray中扩展一个`arrayByRemovingFirstObject`方法。你可以把它放入类别中，而不是创建一个子类。像这样：
```
@interface NSArray (OBJExtras)
- (void)obj_arrayByRemovingFirstObject;
@end
```
当使用类别扩展一个不是你自己创建的类时，在方法前加前缀是一个很好的做法。如果不这样做，其他人可能会使用同样的技术实现同样的一个方法。然后，如果行为不匹配，可能会有意想不到的事情发生。

使用类别的危险之一是，你的项目可能最终会使用大量的类别，这样你可能会丢失你的概述（you can lose your overview.）。这种情况下，创建一个自定义类可能更简单。

#### 方案：配置对象
我一直在犯的一个错误（现在可以很快的意识到）是：会创建一个带有很多抽象方法的类，然后很多子类都会重写一个特定的方法。例如，在一个演示应用程序，你可能会一个`Theme`类，他有几个属性，如：`backgroundColor`和`font`，还有一些在幻灯片上放置东西的逻辑。

然后，对于每一个主题，你创建一个`Theme`的子类，重写一个方法（如：`setup`），并配置属性。直接使用子类没有意义。这种情况下，你可以使用配置对象让你的代码更加简单。你可以把共享逻辑放在`Theme`类中（如：幻灯片布局），但是把配置放在一个仅仅有属性的简单对象中。

例如，一个`ThemeConfiguration`类有`backgroundColor`和`font`属性，并且`Theme`类在初始化的时候获取一个这个类的实例。

#### 方案：组合
最强大的子类化的替代方案是组合。如果你想重用已存在的代码但是没有共享同样的接口，组合是你可以选择的“武器”。例如，假设你正在设计一个缓存类：
```
@interface OBJCache: NSObject
- (void)cacheValue:(id)value forKey:(NSString *)key;
- (void)removeCacheValueForKey:(NSString *)key;
@end
```
达到此目的的一个简单方式是子类化`NSDictionary`并且通过调用字典方法实现这两个方法。
```
@interface OBJCache: NSDictionary
```
然而，这存在几个缺点。用字典实现的事实应该是一个实现细节。现在，在任何需要一个`NSDictionary`类型参数的地方，你都可以提供一个`OBJCache`值。如果你想换一种完全不同的东西（例如，你自己的类库），你需要重构很多代码。

一个好的方式是把这个字典存放到一个私有属性中（或者实例变量中），并且仅仅暴露出两个`cache`方法。现在，你可以维持灵活性，当你获取更多的见解时可以随意修改实现，并且类的使用者不需要进行重构。

---

## 用VIPER构建iOS应用——by *Jeff Gilbert* and *Conrad Stoll*
众所周知，在建筑领域，我们塑造我们的建筑，随后我们的建筑也塑造我们。正如程序员最终知道那样，这也适用于构建软件。

设计我们的代码很重要，这样每一个片段都很容易识别，有特定和明确的目的，以合理的方式同其他片段相配合。这就是我们所谓的软件架构。好的架构不是让产品成功，而是让产品可维护并且帮助维护人员保持一个清晰地思路。

在这篇文章中，我们将介绍一种称之为[VIPER](https://mutualmobile.com/posts/meet-viper-fast-agile-non-lethal-ios-architecture-framework)的iOS应用架构方案。`VIPER`已被用来创建了很多大型的项目，但是为了这篇文章的我们通过创建的一个`to-do`列表应用来向你展示`VIPER`架构。你可以在[GitHub](https://github.com/objcio/issue-13-viper)上关注这个示例项目：
/Users/csip/Documents/docs/notes/images/issue13/2014-06-07-viper-preview .mp4

### VIPER为何物？
测试不总是构建iOS应用程序的主要部分。当我们开始寻求改善[Mutual Mobile](https://github.com/mutualmobile/)的测试实践时，我们发现为iOS应用写测试用例很困难。我们决定，如果我们打算改善测试软件的方式，我们首先要想出一个好的方式来构建应用程序。我们把这种方式称为`VIPER`。

对iOS程序来说，`VIPER`是应用[整洁架构（Clean Architecture）](http://blog.8thlight.com/uncle-bob/2012/08/13/the-clean-architecture.html)的架构模式。单词`VIPER`是由视图（`View`）、交互器（`Interactor`）、展示器（`Presenter`）、实体（`Entity`）和路由（`Routing`）的首字母组合成的。整洁架构把应用逻辑结构划分为不同的职责层。这让依赖分离更加简单（如：你的数据库）并且层边界间的交互也很容易测试。
![viper-intro](/Users/csip/Documents/docs/notes/images/issue13/2014-06-07-viper-intro-0a53d9f8.jpg)
大多是iOS应用都是使用`MVC`（model-view-controller）架构的。使用`MVC`作为应用的架构让你认为每一个类既是模型（`model`）也是视图（`view`）和控制器（`controller`）。由于很多应用逻辑都不属于模型（`model`）和视图（`view`），最后它们都被放在了控制器中。这就导致了一个被称之为[大型视图控制器（Massive View Controller）](https://twitter.com/Colin_Campbell/status/293167951132098560)的问题，在这里视图控制器做了太多的工作。为大型视图控制器瘦身不单单是寻求改善代码质量的iOS程序员所面临的挑战，它也是一个很好的开始（改善项目的架构的开始）。

`VIPER`的不同层通过为应用逻辑和导航相关的代码提过清晰地位置来应对这一挑战。随着VIPER架构的应用，你会意识到在我们的to-do列表例子中的视图控制器很精简、很平衡，视图控制机（view controlling machines）。你也会发现在视图控制器和其他类中的代码很容易理解和测试，因此也更利于维护。

### 基于用例的应用设计
应用通常作为一组用例来实现。用例也成为验收标准或者行为，用来描述应用是用来干嘛的？也许列表需要按时间、类型或者名称进行排序。这就是个用例。用例是负责业务逻辑的应用层。用例应该独立于它们的用户界面实现。它们也应该小且易于定义。决定如何把复杂的应用分解成小巧的用例很有挑战性而且需要练习，但对于限制你解决的每一个问题和你写的每一个类的范围非常有用。

使用VIPER构建应用需要实现一系列组件来完成每一个用例。应用逻辑是每一个用例实现的主要部分，但不是唯一的部分。用例同样影响着用户界面。此外，考虑如何让用例与其他核心组件配合很重要，例如网络和数据展示。组件就像用例的插件一样，VIPER描述的是每一个组件等角色是什么和他们是如何同其他组件交互的。

对于我们的代办列表应用，其中一个用例或者需求是用用户选择的不同的方式组织这些代办事项。通过把组织数据的逻辑分离成用例，我们可以保持用户界面代码整洁且易于将用例包装在测试中，以保证它可以如预期的那样继续工作。

### VIPER的主要部分
VIPER的主要部分是：

* 视图（`View`）：显示展示器让它显示的东西并将用户的输入传回给展示器。
* 交互器（`Interactor`）：包含用例指定的业务逻辑
* 展示器（`Presenter`）：包含准备展示内容（当从交互器接收到）的逻辑，并对用户的输入进行反馈（通过从交互器请求新数据）。
* 实体（`Entity`）：包含交互器使用的基本的模型对象。
* 路由（`Routing`）：包含描述哪些界面按照什么样的顺序战士的导航逻辑。

这些拆分遵循[单一责任原则](http://www.objectmentor.com/resources/articles/srp.pdf)。交互器（`Interactor`）负责业务分析，展示器负责交互设计，视图负责视觉设计。

下面是不同组件的关系图以及它们是如何连接的：
![viper-wireframe](/Users/csip/Documents/docs/notes/images/issue13/2014-06-07-viper-wireframe-76305b6d.png)
VIPER的不同组件可以以任何顺序在应用中实现，我们选择按照推荐实现的顺序去介绍这些组件。你会发现这个顺序和构建整个应用的过程大概一致，首先是讨论产品需要做什么，然后用户如何与它交互。

#### 交互器（`Interactor`）
交互器表示单个应用用例。它包含操作模型对象（`Entities`）的业务逻辑去执行特定的任务。交互器中所做的工作应该独立于UI。同样的交互器可以用在iOS应用中或者OSX应用中。

因为交互器是主要包含逻辑的简单对象（`PONSO:Plain Old NSObject`），所以使用TDD很容易开发。

这个简单应用的主要用例是展示用户即将到来的代买事项（例如：下星期到期的任何东西）。这个用例的业务逻辑是查询出今天和下周末之间到期的任何待办事项，然后为其指定一个相关的到期时间：今天，明天，本周晚些时候，下周。

下面是来自`VTDListInteractor`的相应方法：
```
- (vodd)findUpcomingItems {
	__weak typeof(self) welf = self;
	NSDate *today = [self.clock today];
	NSDate *endOfNextWeek = [[NSCalendar currentCalendar] dateForEndOfFollowingWeekWithDate: today];
	[self.dataManager todoItemsBetweenStartDate:today endDate:endOfNextWeek completionBlock:^(NSArray *todoItems) {
		[welf.output foundUpcomingItems:[welf upcomingItemsFromToDoItems:todoItems]];
	}];
}
```

#### 实体（`Entity`）
实体是由交互器（`Interactor`）操作的模型对象。实体（`Entity`）只能由交互器（`Interactor`）来操作。交互器（`Interactor`）绝不会把实体（`Entity`）传递给展示层（如：展示器（`Presenter`））。

实体（`Entity`）往往也是普通对象。如果你是用`Core Data`，你将会希望你的管理对象保持在数据层之后。交互器不应该同`NSManagedObjects`一起使用。

下面是我们的待办项实体：
```
@interface VTDTodoItem: NSObject
@property (nonatomic, strong) NSDate *dueDate;
@property (nonatomic, copy) NSString *name;
+ (instancetype)todoItemWithDueDate:(NSDate *)dueDate name:(NSString *)name;
@end
```
如果你的实体仅仅只是数据结构请不要大惊小怪。任何应用相关的逻辑大多数都在交互器中。

#### 展示器（`Presenter`）
展示器是主要包含驱动UI逻辑的普通对象。它知道何时展示用户界面。它从用户交互中获取输入，所以它可以更新UI并向交互器发送请求。

当用户点击“+”按钮添加新代办事项时，`addNewEntry`就被调用了。对于这个方法，展示器要求线框展示用于添加新项的UI：
```
- (void)addNewEntry {
	[self.listWireframe presentAddInterface];
}
```
展示器也接收来自交互器的结果，并把结果转换为可以在视图中高校展示的表单。

下面是从交互器接收即将到来项目的方法。它会处理数据并决定向用户展示哪些东西：
```
- (void)foundUpcomingItems:(NSArray *)upcomingItems {
	if([upcomingItems count] == 0) {
		[self.userInterface showNoContentMessage];
	} else {
		[self updateUserInterfaceWithUpcomingItems:upcomingItems];
	}
}
```
绝不会把实体从交互器传递到展示器。而是把简单没有行为的数据结构从交互器传到了展示器。这可以防止在展示器中完成任何“实际工作”。展示器只为视图准备展示的数据。

#### 视图（`View`）
视图是被动的。它等待展示器给它展示的内容；从不主动向展示器请求数据。为视图定义的方法（如：登陆界面的`LoginView`）应该允许展示器在一个较高的抽象层次上与其通信，用其内容展示，而不是如何展示内容。展示器不知道`UILabel`、`UIButton`等的存在。只知道它持有的内容以及该何时展示。如何展示内容这取决于视图。

视图是一个定义为Objective—C协议的抽象接口。一个视图控制器（`UIViewController`）或者其子类将会实现这个视图协议。例如，我们的示例中的添加界面有如下接口：
```
@protocol VTDAddViewInterface <NSObject>
- (void)setEntryName:(NSString *)name;
- (void)setEntryDueDate:(NSDate *)date;
```
视图和视图控制器都处理用户交互和输入。这也就不难理解为什么视图控制器总是会变得那么臃肿，因为这里是最容易处理该输入去执行一些动作的地方。为了让视图控制器保证精简，当用户执行确定的动作时我们需要提供一种方式去通知对其感兴趣的部分。视图控制器不能基于这些动作做出决定，但是可以把这些事件传递到可以做决定的地方。

在我们的例子中，“添加”视图控制器具有符合下面接口的事件处理器属性：
```
@protocol VTDAddModuleInterface <NSObject>
- (void)cancelAddAction;
- (void)saveAddActionWithName:(NSString *)name dueDate:(NSDate *)dueDate;
@end
```
当用户点击取消按钮，视图控制器告诉用户指定的事件处理器它去次奥了添加动作。那样，事件处理器可以做出如下处理：关闭“添加”视图控制器和通知列表视图更新。

视图和展示器之间的边界是使用[ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)的绝佳地方。在这个例子中，视图控制器可以提供方法返回代表按钮动作的信号。这可以让展示器很容易的对这些信号进行响应，而不用破坏职责分离。

#### 路由（`Routing`）
由交互设计师设计的线框图定义了从一个界面到另一个界面的路由。在`VIPER`中，路由职责由展示器和线框图这两个对象负责。线框图对象拥有`UIWindow`、`UINavigationController`、`UIViewController`等。它负责穿件视图/视图控制器并把它加载到window上。

由于展示器包含响应用户输入的逻辑，所以展示器知道何时导航到其他的界面以及导航到哪个界面。当然，线框图也知道如何导航。因此，展示器将使用线框图执行导航。他们共同描述了一个从一个视图导航到下一个的路由。

线框图也是一个明显的处理导航转场动画的地方。看一下来自于"添加"线框图的例子：
```
@implementation VTDAddWireframe
- (void)presentAddInterfaceFromViewController:(UIViewController *)viewController {
	VTDAddViewController *addViewController = [self addViewController];
	addViewController.eventHandler = self.addPresenter;
	addViewController.modalPresentationStyle = UIModalPresentationCustom;
	addViewController.transitioningDelegate = self;
	[viewController presentViewController:addViewController animated:YES completion:nil];
	self.presentedViewController = viewController;
}
@end

#pragma mark - UIViewControllerTransitioningDelegate Methods
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForDismissedController:(UIViewController *)dismissed {
	return [[VTDAddDismissalTransition alloc] init];
}
- (id<UIViewControllerAnimatedTransitioning>)animationControllerForPresentedController:(UIViewController *)presented presentingController:(UIViewController *)presenting sourceController:(UIViewController *)source {
	return [[VTDAddPresentationTransition alloc] init];
}
```
应用使用的是自定义视图控制器转场去展示“添加”视图控制器。因为线框图负责执行转场动作，所以它成了“添加”视图控制器的转场委托，并返回合适的转场动画。

### 适用于VIPER的应用组件
iOS应用架构需要考虑到一个事实，`UIKit`和`Cocoa Touch`是构建应用的主要工具。架构需要同应用中所有的组件和谐共处，但是，这也需要提供参考指南，用来说明框架中的一些模块如何使用以及用在何处。

iOS应用的主力是`UIViewController`。我们很容易认为，取代`MVC`的竞争者可以避免视图控制器的过度使用。但，视图控制器是平台的核心：它们处理屏幕翻转，响应用户输入，与像导航控制器这样的系统组件组合，现在在iOS7中，也许自定义界面转场动作。非常有用。

使用VIPER，视图控制器执行它应该做的事情：控制视图。我们的代办列表应用有两个视图控制器，一个是列表界面，另一个是“添加”界面。“添加”视图控制制器的实现很基础，因为它所要做的就是控制视图：
```
@implementation VTDAddViewController
- (void)viewDidAppear:(BOOL)animated {
	[super viewDidAppear:animated];
	UITapGestureRecognizer *gestureRecognizer = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(dismiss)];
	[self.transitioningBackgroundView addGestureRecognizer：gestureRecognizer];
	self.transitioningBackgroundView.userInteractionEnabled = YES;
}
- (void)dismiss {
	[self.eventHandler cancelAddAction];
}
- (void)setEntryName:(NSString *)name {
	self.nameTextField.text = name;
}

- (void)setEntryDueDate:(NSDate *)date {
	[self.datePicker setDate:date];
}
- (IBAction)save:(id)sender {
	[self.eventHandler saveAddActionWithName:self.nameTextField.text dueDate:self.datePicker.date];
}
- (IBAction)cancel:(id)sender {
	[self.eventHandler cancelAddAction];
}
#pragma mark - UITextFieldDelegate Methods
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
	[textField resignFirstResponder];
	return YES;
}
@end
```
当应用连接网络后，通常会更具吸引力。但是联网应该发生在哪里？应该由谁启动它呢？通常的，由交互器决定去启动网络操作，但是它不会直接处理联网代码。它将会请求一个像网络管理器或者`API`客户端的依赖。交互器可能需要从多个数据源汇总数据，以提供完成用例所需的信息。然后由展示器接收由交互器返回的数据，并为展示进行格式化。

数据存储负责向交互器提供实体。由于交互器应用其交互逻辑，它需要从数据存储取回实体，处理实体并把更新过的实体放回到数据存储中。数据存储管理持久化的实体。实体不知道数据存储，因此也就不知道如何对自己进行持久化。

交互器也不应该知道如何持久化实体。有时，交互器可能需要使用一个被称为数据管理器的对象去帮助自己同数据存储进行交互。数据管理器处理特定存储类型的操作，像创建获取数据请求，创建查询等。这让交互器更多的关注应用逻辑而不用知道实体是如何获取和持久化实体的。在你使用`Core Data`的时候使用数据管理器才是有意义的，你可以在下面看到对他的描述。

这是示例应有的数据管理器接口：
```
@interface VTDListDataManager: NSObject
@property (nonatomic, strong) VTDCoreDataStore *dataStore;
- (void)todoItemsBetweenStartDate:(NSData *)startDate endDate:(NSDate *)endDate completionBlock:(void (^)(NSArray *todoItems))completionBlock;
@end
```
但是用`TDD`开发交互器时，可以使用测试`double/mock`来切换生产数据存储。不与远程服务器（用于web服务）和本地磁盘（用于数据库）进行通信可以让你的测试更快速且更可重复。

把数据存储放在边界明显的层的理由是，它允许你推迟选择特定的持久化技术。如果你的数据存储是单个类，你可以使用使用基本的持久策略启动你的应用，然后在适当的情况下升级到到`SQLite`或者`Core Date`，而无需更改应用代码库中的其他任何内容。

在iOS项目中使用`Core Date`经常会引发比架构自己还要多的争议。然而，在`VIPER`中使用`Core Date`可以成为你曾经有过的最好的`Core Date`使用体验。`Core Date`是非常好的数据持久化工具，它有着极快的获取速度和极低的内存占用。但是有一个惯例，就是在应用程序的实现文件中，即使不应该出现，也需要设置繁琐的`NSManagedObjectContext`。`VIPER`把`Core Data`放在了它应该在的地方：数据存储层。

在待办列表例子中，应用仅有的两个部分知道`Core Data`正在被使用的是数据存储本身，在这里设置`Core Data`堆栈和数据管理器。数据管理器执行获取请求，把数据存储层返回的NSManagedObjects对象转换成标准的简单对象模型，并把它返回给业务逻辑层。这样，应用程序的核心就不会依赖`Core Data`，作为回报，你不用担心由于过时或线程有问题的`NSManagedObjects`而导致应用无法工作。

在数据管理器中，当请求访问`Core Data`存储时，看起来是下面这样：
```
@implementatin VTDListDataManager
- (void)todoItemsBetweenStartDate:(NSDate *)startDate endDate:(NSDate *)endDate completionBlock:(void (^)(NSArray *todoItems))completionBlock {
	NSCalendar *calendar = [NSCalendar autoupdatingCurrentCalendar];
	NSPredicate *predicate = [NSPredicate predicateWithFormat:@"(date >= %@) AND (date <= %@)", [calendar dateForBeginningOfDay:startDate], [calendar dateForEndOfDay:endDate]];
	NSArray *sortDescriptors = @[];
	__weak type(self) welf = self;
	[self.dateStore fetchEntriesWithPredicate:predicate sortDescriptors:sortDescriptors completionBlock:^(NSArray *entries){
		if(completionBlock) {
			completionBlock([welf todoItemsFromDataStoreEntries：entries]);
		}
	}];
}

- (NSArray *)todoItemsFromDataStoreEntries:(NSArray *)entries {
	return [entries arrayFromObjectsCollectedWithBlock:^id(VTDManagedTodoItems *todo) {
		return [VTDTodoItem todoItemWithDueDate:todo.date name:todo.name];
	}];
}
@end
```
几乎同`Core Data`有同样争议的是`UI Storyboards`。`Storyboards`有很多使用的特性，完全的忽略它们是一个错误。然而，当使用`storyboard`提供的所有特性时，很难实现VIPER的所有目标。

常常，我们做的妥协是选择不使用连线（`segues`：`storyboard`中`controller`之间的连线）。可能存在一些使用连线是有意义的例子，使用连线（`segues`）的危险在于，很难保持界面之间、UI和应用逻辑之间的完整分离。一般来说，当明显需要实现`prepareForSegue`方法的时候，我们尽量不要使用连线（`segues`）。

此外，`storyboards`是一种很好的实现用户界面布局的方式，特别是在使用自动布局的时候（`Auto Layout`）。待办列表例子中的两个界面我们都是用`storyboard`来实现，然后用如下代码去执行我们自己的导航：
```
static NSString *ListViewControllerIdentifier = @"VTDListViewController";
@implementation VTDListWireframe
- (void)presentListInterfaceFromWindow:(UIWindow *)window {
	VTDListViewController *listViewController = [self listViewControllerFromStoryboard];
	listViewController.eventHandler = self.listPresenter;
	self.listPresenter.userInterface = listViewController;
	self.listViewController = listViewController;
	[self.rootWireframe showRootViewController:listViewController inWindow:window];
}
- (VTDListViewController *)listViewControllerFromStoryboard {
	UIStoryboard *storyboard = [self mainStoryboard];
	VTDListViewController *viewController = [storyboard instantiateViewControllerWithIdentifier:ListViewControllerIdentifier];
	return viewController;
}
- (UIStoryboard *)mainStoryboard {
	UIStoryboard *storyboard = [UIStoryboard storyboardWithName:@"Main" bundle:[NSBundle mainBundle]];
	return storyboard;
}
@end
```

### 使用VIPER构建模块
通常在使用VIPER的时候，你会发现一个界面或一组界面常常会作为一个模块组织在一起。一个模块可以有几种方式描述，通常的把它作为一个特性来描述是最好的选择。在一个播客应用中，模块可能是一个音频播放器或者订阅浏览器。在我们的待办列表应用中，列表和“添加”界面都构建成了独立的模块。

把你的应用设计成一系列模块有几个好处。其中一个是：模块有着清晰且定义良好的接口，同时独立于其他模块。这使得添加/移除特性或者改变你的接口向用户呈现各种模块的方式。

我们希望在待办列表例子中清晰的区分模块，所以我们为“添加”模块定义了两个协议。第一个是模块接口，这里定义了模块可以做什么。第二个是模块委托，这里描述模块做了什么。例如：
```
@protocol VTDAddModuleInterface <NSObject> 
- (void)cancelAddAction;
- (void)saveAddActionWithName:(NSString *)name dueDate:(NSDate *)dueDate;
@end

@protocol VTDAddModuleDelegate <NSObject>
- (void)addModuleDidCancelAddAction;
- (void)addModelDidSaveAddAction;
@end
```
由于模块必须得呈现给用户，所以模块通常会实现模块接口。当另外一个接口想展示这个模块时，他的展示器需要实现模块接口协议，这样它就可以知道在展示它时模块做了什么。

模块可能包含用于多个界面的实体、交互器和管理器的通用应用逻辑层。当然，这依赖于这些界面之间的交互和他们之间的相似度。一个模块可以很容易的代表一个界面，正如在待办列表示例中多展示的那样。这种情况下，应用逻辑层可以对应于特定模块中非常具体的行为。

模块也是一种很好的组织代码方式。把一个模块的代码隐藏在自己的文件夹内并且`Xcode`中组会让很容易的找到你需要修改的东西。当你在期望的地方找到一个类时，这是一种很棒的感觉。

使用VIPER构建模块的另一个好处是它们很容易扩展到多种形式。在交互层分离所有用例的应用逻辑让你在重用应用层的同时还专注于为平板电脑、手机、和mac电脑构建新的用户界面。

更进一步，iPad应用的用户界面可能会重用iPhone应用的一些视图、视图控制器和展示器。这种情况下，一个iPad界面可能会由父展示器和线框图所代表，它可能会使用已存在的iPhone展示器和线框图组成界面。构建和维护跨平台的应用会相当有挑战性，但是能在整个应用和应用层促进重用的良好架构可以让这变的更容易。

### 使用VIPER进行测试
VIPER鼓励分离关注点这使得它更容易适应TDD。交互器包含独立于UI的纯逻辑，这使得测试更容易驱动。展示器包含为展示准备数据的逻辑且它独立于任何`UIKit`控件。开发这个逻辑也让测试更易驱动。

我们首选的方法从交互器开始。UI中的所有内容都可以满足用例的需要。通过使用TDD为交互器的API去测试驱动，你会更好的理解UI和用例之间的关系。

例如，我们将看到负责即将到来的待办事项列表的交互器。寻找即将到来项的规则是查询出截止到下周结束的所有待办事项并按照截止到今天、明天、本周晚些时候或者下周对每个待办项进行分类。

我们写的第一个例子是保证交互器找出截止到下周结束的所有待办事项：
```
- (void)testFindingUpcomingItemsRequestsAllToDoItemsFromTodayThroughEndOfNextWeek {
	[[self.dataManager expect] todoItemsBetweenStartDate:self.today endDate:self.endOfNextWeek completionBlock:OCMOCK_ANY];
	[self.interactor findUpcomingItems];
}
```
一旦我们知道交互器请求适当的待办事项，我们将会写几个测试方法去确定它把待办事项分配给正确的相关日期组（例如：今天、明天等）。
```
- (void)testFindingUpcomingItemsWithOneItemDueTodayReturnsOneUpcomingItemsForToday {
	NSArray *todoItems = @[[VTDTodoItem todoItemWithDueDate:self.today name:@"Item 1"]];
	[self dataStoreWillReturnToDoItems:todoItems];
	
	NSArray *upcomingItems = @[[VTDUpcomingItem upcomingItemWithDateRelation:VTDNearTermDateRelationToday dueDate:self.today title:@"Item 1"]];
	[self expectUpcomingItems:upcomingItems];
	
	[self.interactor findUpcomingItems];
}
```
现在我们知道交互器的API长什么样了，我们可以开发展示器了。当展示器接收到来自交互器的即将到来的待办事项时，我们将要测试我们是否正确的格式化数据并把它显示在UI上：
```
- (void)testFoundZeroUpcomingItemsDisplaysNoContentMessage {
	[[self.ui expect] showNoContentMessage];
	[self.presenter foundUpcomingItems:@[]];
}
- (void)testFoundUpcomingItemForTodayDisplaysUpcomingDataWithNoDay {
	VTDUpcomingDisplayData *displayData = [self displayDataWithSectionName:@"Today" sectionImageName:@"check" itemTitle:@"Get a haircut" itemDueDay:@""];
	[[self.ui expect] showUpcomingDisplayData:displayData];
	NSCalendar *calendar = [NSCalendar gregorianCalendar];
	NSData *dueData = [calendar dateWithYear:2014 month:5 day:29];
	VTDUpcomingItem *haircut = [VTDUpcomingItem upcomingItemWithDateRelation:VTDNearTermDateRelationToday dueDate:dueDate title:@"Get a haircut"];
	[self.presenter foundUpcomingItems:@"haircut"];
}
- (void)testFoundUpcomingItemForTomorrowDisplaysUpcomingDataWithDay {
	VTDUpcomingDisplayData *displayData = [self displayDataWithSectionName:@"Tomorrow" sectionImageName:@"alarm" itemTitle:@"Buy groceries" itemDueDay:@"Thursday"];
	[[self.ui expect] showUpcomingDisplayData:displayData];
	NSCalendar *calendar = [NSCalendar gregorianCalendar];
	NSDate *dueDate = [calendar dateWithYear:2014 month:5 day:29]; 
	VTDUpcomingItem *groceries = [VTDUpcomingItem upcomingItemWithDateRelation:VTDNearTermDateRelationTomorrow dueDate:dueDate title:@"Buy groceries"];
	[self.presenter foundUpcomingItems:@[groceries]];
}
```
我们也想测试一下，当用户想添加新的待办事项时，应用将开始适当的操作：
```
- (void)testAddNewToDoItemActionPresentsAddToDoUI {
	[[self.wireframe expect] presentAddInterface];
	[self.presenter addNewEntry];
}
```
现在我们可以开发视图了。当没有即将到来的待办事项的时候，我们会展示一个特别的消息：
```
- (void)testShowingNoContentMessageShowsNoContentView {
	[self.view showNoContentMessage];
	XCTAssertEqualObjects(self.view.view, self.view.noContentView, @"the no content view should be the view");
}
```
当有即将到来的待办事项展示时，我想确定列表被展示了出来：
```
- (void)testShowingUpcomingItemsShowsTableView {
	[self.view showUpcomingDisplayData:nil];
	XCTAssertEqualObjects(self.view.view, self.view.tableView, @"the table view should be the view");
}
```
构建交互器首先是与TDD自然的契合。如果你首先开发交互器，然后是展示器，你会在这些层周围构建出一套测试方法，为实现这些用例打下基础。你可以快速的遍历这些类，因为你不需要为了测试他们而与UI进行交互。然后当你开始开发视图的时候，你会有一个可行且经过测试的逻辑还有一个与其连接的展示层。到那时，你完成视图开发，你可能会发现当你第一次运行应用的时候一切工作正常，因为所有你通过的测试都告诉你它会起作用。

### 结论
我希望你喜欢这篇对VIPER的介绍。现在，你们中的很多人可能想知道下一步怎么做。如果你想用VIPER构建你下一个应用，应该从哪里开始？

这篇文章以及使用VIPER实现的实例应用正和我们能够做到的那样具体且有着良好的定义。我们的待办事项列表应用相当简单，但也非常准确的阐述了怎样使用VIPER构建一个应用。在实际的项目中，你是否严格按照例子去实现依赖于你自己的一系列挑战和约束。根据我们的经验，我们的每一项目都略微的改变了VIPER的使用方式，但是他们都从指导他们的方法中受益匪浅。

出于各种原因，你可能会出现偏离VIPER制定的路线的情况。也许你会遇见一个“兔子”对象，或者你的应用会在Storyboard中使用连线（`segues`）受益。没关系，在这些情况下，当你做决定的时候，想一下VIPER所代表的思想。VIPER的核心是一个基于[单一责任原则](http://en.wikipedia.org/wiki/Single_responsibility_principle)的架构。当在决定如何继续下一步的时候，如果你有疑问可以想一下这个原则。

你可能想知道，如果在已存在的应用中使用VIPER是否可行。在这种情况下，可以考虑构建使用VIEPR构建一个新特性。很多我们已存在的项目都可以采取这种方式。这允许你使用VIPER构建一个模块，并且可以帮助你发现任何已存在的问题，这是这个问题让你很难适应基于单一责任原则的架构。

每一个应用都有所差异这是开发软件最重要的事情之一，并且构建app的方式也不尽相同。对于我们来说，这意味着每一个应用都是一个新的学习和尝试新鲜东西的机会。

### Swift补遗
在上周的苹果开发者大会上，苹果介绍了作为未来开发Cocoa和Cocoa Touch的编程语言——[Swift](https://developer.apple.com/swift/)。对Swift语言进行深入的点评还为时过早，但是我们知道这个语言对如何设计和构建软件产生了重大的影响。我们决定使用Swift重写我们的VIPER待办示例应用去帮助我们认识这对VIPER意味着什么。目前为止，我们喜欢我们看到的东西。这里有几个我认为可以提高使用VIPER构建应用体验的Swift特性。

#### Structs
在VIPER中我们使用小且轻量级的模型类在层之间传递数据，如：从展示器到视图。这些普通对象通常只是想简单地携带少量的数据，并不想被子类化。Swift结构能够同这些情况非常完美的契合。下面是一个在VIPER Swift示例中使用结构的例子。注意这个结构需要相等操作，所以我们重载了“==”操作符去比较同类型的两个实例：
```
struct UpcomingDisplayItem: Equatable, Printable {
	let title: String = ""
	let dueDate: String = ""
	var description: String {
		get {
			return "\(title) -- \(dueDate)"
		}
	}
	init(title: String, dueDate: String) {
		self.title = title
		self.dueDate = dueDate
	}
}
func ==(leftSide: UpcomingDisplayItem, rightSide: UpcomingDisplayItem) -> Bool {
	var hasEqualSections = false
	hasEqualSections = rightSide.title == leftSide.title
	if hasEqualSections == false {
		return false
	}
	hasEqualSections = rightSide.dueDate == rightSide.dueDate
	return haseEqualSections
}
```

#### 类型安全
也许Object-C和Swift两者最大的区别是对类型的处理。Object-C是动态类型而Swift对在编译时实现类型检查的方式非常严格。对于像VIPER这样的由多个不同层组成的架构来说，类型安全对程序员的效率和总体架构来说是一个巨大的胜利。编译器帮助你确保容器和对象在层边界间进行传递时类型的正确性。如上面所示，这是使用结构的好地方。如果结构想要在两层边界之间生存，多亏了类型安全，你可以保证它将永远不可能从这两层间逃离。


